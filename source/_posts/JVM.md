---
title: JVM 源码阅读(OpenJDK8)
tags: JVM
categories: Java
---

## Java 启动执行的源代码跟踪(OpenJDK8)
main.c —> java.c(JLI_Launch函数) 文件:  
```bash
1. CreateExecutionEnvironment    ==>    GetJREPath   +   GetJVMPath  
            GetJREPath   :    获取 libjava.so 文件路径,  赋值给 jrepath .
            GetJVMPath  :   获取  libjvm.dylib  文件路径, 赋值给 jvmpath.
2. LoadJavaVM     ==>      libjvm = dlopen(jvmpath, RTLD_NOW + RTLD_GLOBAL);            加载链接文件 libjvm.dylib,  将以下3个函数指向链接文件中的函数
                                           CreateJavaVM = (CreateJavaVM_t)dlsym(libjvm, "JNI_CreateJavaVM”);           
                                              GetDefaultJavaVMInitArgs = (GetDefaultJavaVMInitArgs_t)dlsym(libjvm, "JNI_GetDefaultJavaVMInitArgs”);
                                              GetCreatedJavaVMs = (GetCreatedJavaVMs_t)dlsym(libjvm, "JNI_GetCreatedJavaVMs");
3. SetJavaCommandLineProp       ==>    JLI_StrCCmp(str, "-Xss”)  JLI_StrCCmp(str, "-Xmx”)  JLI_StrCCmp(str, "-Xms”)       jvm 堆栈大小
4. JVMInit   ==>   ContinueInNewThread(InvocationFunctions* ifn)
                                                ==>   ifn->GetDefaultJavaVMInitArgs(&args1_1);
                                                    ==>    ContinueInNewThread0(JavaMain, threadStackSize, (void*)&args);   create new thread, and execute JavaMain function
                                                     主要函数:  ==>   JavaMain  函数   执行下列过程
 JavaMain  函数     ==>   RegisterThread()  +  InitializeJVM(&vm, &env, &ifn) { => (ifn->CreateJavaVM(pvm, (void **)penv, &args);) }   +  LoadMainClass(env, mode, what)   +  GetApplicationClass(env)  +  PostJVMInit(env, appClass, vm) (for GUI purpose)
                        +    mainID = (*env)->GetStaticMethodID(env, mainClass, "main”, "([Ljava/lang/String;)V”);
          + mainArgs = CreateApplicationArgs(env, argv, argc);                  build platform specific argument array
          + (*env)->CallStaticVoidMethod(env, mainClass, mainID, mainArgs);     invoke main thread
//  InitializeJVM -> CreateJavaVM 函数
ifn->CreateJavaVM(pvm, (void **)penv, &args); ==> 会调用 jni.cpp 的 JNI_CreateJavaVM 函数.  
    JNI_CreateJavaVM   ==>   Threads::create_vm((JavaVMInitArgs*) args, &can_try_again);

// java.c -> LoadMainClass(env, mode, what) 函数 ,  env 为load 进的 jni 函数环境.              通过class loader 取 load class 进来
==>   jclass cls = GetLauncherHelperClass(env);   
                   GetLuncherHelperClass 函数会:  return  helperClass = FindBootStrapClass(env, "sun/launcher/LauncherHelper”) load jni: JVM_FindClassFromBootLoader
==>  NULL_CHECK0(mid = (*env)->GetStaticMethodID(env, cls,
                "checkAndLoadMain",
                "(ZILjava/lang/String;)Ljava/lang/Class;"));  
==> str = NewPlatformString(env, name);    str 为组装后的java 启动 main函数类名
==>  result = (*env)->CallStaticObjectMethod(env, cls, mid, USE_STDERR, mode, str); 调用 LauncherHelper.java的checkAndLoadMain函数,返回 mainclass.

// java.c  ->  GetApplicationClass(env) 函数
    /*
     * In some cases when launching an application that needs a helper, e.g., a
     * JavaFX application with no main method, the mainClass will not be the
     * applications own main class but rather a helper class. To keep things
     * consistent in the UI we need to track and report the application main class.
     */
==>  jclass cls = GetLauncherHelperClass(env);
==>   NULL_CHECK0(mid = (*env)->GetStaticMethodID(env, cls,
                "getApplicationClass",
                "()Ljava/lang/Class;”));
==>  return (*env)->CallStaticObjectMethod(env, cls, mid);   调用 LauncherHelper.java的getApplicationClass函数,返回 appclass.
```

```bash
typedef struct {
    CreateJavaVM_t CreateJavaVM;
    GetDefaultJavaVMInitArgs_t GetDefaultJavaVMInitArgs;
    GetCreatedJavaVMs_t GetCreatedJavaVMs;
} InvocationFunctions;
```

### JVM 依赖的两个 lib库
libjava.so
libjvm.so

### java class 与 jni mapping 表
See   openjdk8/hotspot/src/share/vm/classfile/vmSymbols.hpp   

### jni.cpp 文件 JNI_CreateJavaVM 函数:  
```bash
1. HS_DTRACE_PROBE3(hotspot_jni, CreateJavaVM__entry, vm, penv, args);
2.  result = Threads::create_vm((JavaVMInitArgs*) args, &can_try_again);
3.    JavaThread *thread = JavaThread::current();
    /* thread is thread_in_vm here */
    *vm = (JavaVM *)(&main_vm);
4.  *(JNIEnv**)penv = thread->jni_environment();
    // Tracks the time application was running before GC
5.  RuntimeService::record_application_start();          track runtime application for gc
    // Notify JVMTI
6.  if (JvmtiExport::should_post_thread_life()) {
       JvmtiExport::post_thread_start(thread);
    }
    EventThreadStart event;
    if (event.should_commit()) {
7.    event.set_javalangthread(java_lang_Thread::thread_id(thread->threadObj()));
      event.commit();
    }
    // Check if we should compile all classes on bootclasspath
8.  if (CompileTheWorld) ClassLoader::compile_the_world();        do compile work
9.  if (ReplayCompiles) ciReplay::replay(thread);                 replay compile work
    // Since this is not a JVM_ENTRY we have to set the thread state manually before leaving.
10. ThreadStateTransition::transition_and_fence(thread, _thread_in_vm, _thread_in_native);      


openjdk8/hotspot/src/share/vm/runtime/thread.cpp:
thread.cpp 文件 create_vm 函数:

jint Threads::create_vm(JavaVMInitArgs* args, bool* canTryAgain) {

  extern void JDK_Version_init();
  // Check version
  if (!is_supported_jni_version(args->version)) return JNI_EVERSION;

  // Initialize the output stream module
  ostream_init();

  // Process java launcher properties.
  Arguments::process_sun_java_launcher_properties(args);

  // Initialize the os module before using TLS
  os::init();

  // Initialize system properties.
  Arguments::init_system_properties();

  // So that JDK version can be used as a discrimintor when parsing arguments
  JDK_Version_init();

  // Update/Initialize System properties after JDK version number is known
  Arguments::init_version_specific_system_properties();

  // Parse arguments
  jint parse_result = Arguments::parse(args);        arguments.cpp 文件中, 支持的所以参数都列举在这.
  if (parse_result != JNI_OK) return parse_result;

  os::init_before_ergo();

  jint ergo_result = Arguments::apply_ergo();
  if (ergo_result != JNI_OK) return ergo_result;

  if (PauseAtStartup) {
    os::pause();
  }

#ifndef USDT2
  HS_DTRACE_PROBE(hotspot, vm__init__begin);
#else /* USDT2 */
  HOTSPOT_VM_INIT_BEGIN();
#endif /* USDT2 */

  // Record VM creation timing statistics
  TraceVmCreationTime create_vm_timer;
  create_vm_timer.start();

  // Timing (must come after argument parsing)
  TraceTime timer("Create VM", TraceStartupTime);

  // Initialize the os module after parsing the args
  jint os_init_2_result = os::init_2();
  if (os_init_2_result != JNI_OK) return os_init_2_result;

  jint adjust_after_os_result = Arguments::adjust_after_os();
  if (adjust_after_os_result != JNI_OK) return adjust_after_os_result;

  // intialize TLS
  ThreadLocalStorage::init();

  // Bootstrap native memory tracking, so it can start recording memory
  // activities before worker thread is started. This is the first phase
  // of bootstrapping, VM is currently running in single-thread mode.
  MemTracker::bootstrap_single_thread();

  // Initialize output stream logging
  ostream_init_log();

  // Convert -Xrun to -agentlib: if there is no JVM_OnLoad
  // Must be before create_vm_init_agents()
  if (Arguments::init_libraries_at_startup()) {
    convert_vm_init_libraries_to_agents();
  }

  // Launch -agentlib/-agentpath and converted -Xrun agents
  if (Arguments::init_agents_at_startup()) {
    create_vm_init_agents();
  }

  // Initialize Threads state
  _thread_list = NULL;
  _number_of_threads = 0;
  _number_of_non_daemon_threads = 0;

  // Initialize global data structures and create system classes in heap
  vm_init_globals();

  // Attach the main thread to this os thread
  JavaThread* main_thread = new JavaThread();
  main_thread->set_thread_state(_thread_in_vm);
  // must do this before set_active_handles and initialize_thread_local_storage
  // Note: on solaris initialize_thread_local_storage() will (indirectly)
  // change the stack size recorded here to one based on the java thread
  // stacksize. This adjusted size is what is used to figure the placement
  // of the guard pages.
  main_thread->record_stack_base_and_size();
  main_thread->initialize_thread_local_storage();

  main_thread->set_active_handles(JNIHandleBlock::allocate_block());

  if (!main_thread->set_as_starting_thread()) {
    vm_shutdown_during_initialization(
      "Failed necessary internal allocation. Out of swap space");
    delete main_thread;
    *canTryAgain = false; // don't let caller call JNI_CreateJavaVM again
    return JNI_ENOMEM;
  }

  // Enable guard page *after* os::create_main_thread(), otherwise it would
  // crash Linux VM, see notes in os_linux.cpp.
  main_thread->create_stack_guard_pages();

  // Initialize Java-Level synchronization subsystem
  ObjectMonitor::Initialize() ;

  // Second phase of bootstrapping, VM is about entering multi-thread mode
  MemTracker::bootstrap_multi_thread();

  // Initialize global modules
  jint status = init_globals();
  if (status != JNI_OK) {
    delete main_thread;
    *canTryAgain = false; // don't let caller call JNI_CreateJavaVM again
    return status;
  }

  // Should be done after the heap is fully created
  main_thread->cache_global_variables();

  HandleMark hm;

  { MutexLocker mu(Threads_lock);
    Threads::add(main_thread);
  }

  // Any JVMTI raw monitors entered in onload will transition into
  // real raw monitor. VM is setup enough here for raw monitor enter.
  JvmtiExport::transition_pending_onload_raw_monitors();

  // Fully start NMT
  MemTracker::start();

  // Create the VMThread
  { TraceTime timer("Start VMThread", TraceStartupTime);
    VMThread::create();
    Thread* vmthread = VMThread::vm_thread();

    if (!os::create_thread(vmthread, os::vm_thread))
      vm_exit_during_initialization("Cannot create VM thread. Out of system resources.");

    // Wait for the VM thread to become ready, and VMThread::run to initialize
    // Monitors can have spurious returns, must always check another state flag
    {
      MutexLocker ml(Notify_lock);
      os::start_thread(vmthread);
      while (vmthread->active_handles() == NULL) {
        Notify_lock->wait();
      }
    }
  }

  assert (Universe::is_fully_initialized(), "not initialized");
  if (VerifyDuringStartup) {
    // Make sure we're starting with a clean slate.
    VM_Verify verify_op;
    VMThread::execute(&verify_op);
  }

  EXCEPTION_MARK;

  // At this point, the Universe is initialized, but we have not executed
  // any byte code.  Now is a good time (the only time) to dump out the
  // internal state of the JVM for sharing.
  if (DumpSharedSpaces) {
    MetaspaceShared::preload_and_dump(CHECK_0);
    ShouldNotReachHere();
  }

  // Always call even when there are not JVMTI environments yet, since environments
  // may be attached late and JVMTI must track phases of VM execution
  JvmtiExport::enter_start_phase();

  // Notify JVMTI agents that VM has started (JNI is up) - nop if no agents.
  JvmtiExport::post_vm_start();

  {
    TraceTime timer("Initialize java.lang classes", TraceStartupTime);

    if (EagerXrunInit && Arguments::init_libraries_at_startup()) {
      create_vm_init_libraries();
    }

    initialize_class(vmSymbols::java_lang_String(), CHECK_0);

    // Initialize java_lang.System (needed before creating the thread)
    initialize_class(vmSymbols::java_lang_System(), CHECK_0);
    initialize_class(vmSymbols::java_lang_ThreadGroup(), CHECK_0);
    Handle thread_group = create_initial_thread_group(CHECK_0);
    Universe::set_main_thread_group(thread_group());
    initialize_class(vmSymbols::java_lang_Thread(), CHECK_0);
    oop thread_object = create_initial_thread(thread_group, main_thread, CHECK_0);
    main_thread->set_threadObj(thread_object);
    // Set thread status to running since main thread has
    // been started and running.
    java_lang_Thread::set_thread_status(thread_object,
                                        java_lang_Thread::RUNNABLE);

    // The VM creates & returns objects of this class. Make sure it's initialized.
    initialize_class(vmSymbols::java_lang_Class(), CHECK_0);

    // The VM preresolves methods to these classes. Make sure that they get initialized
    initialize_class(vmSymbols::java_lang_reflect_Method(), CHECK_0);
    initialize_class(vmSymbols::java_lang_ref_Finalizer(),  CHECK_0);
    call_initializeSystemClass(CHECK_0);

    // get the Java runtime name after java.lang.System is initialized
    JDK_Version::set_runtime_name(get_java_runtime_name(THREAD));
    JDK_Version::set_runtime_version(get_java_runtime_version(THREAD));

    // an instance of OutOfMemory exception has been allocated earlier
    initialize_class(vmSymbols::java_lang_OutOfMemoryError(), CHECK_0);
    initialize_class(vmSymbols::java_lang_NullPointerException(), CHECK_0);
    initialize_class(vmSymbols::java_lang_ClassCastException(), CHECK_0);
    initialize_class(vmSymbols::java_lang_ArrayStoreException(), CHECK_0);
    initialize_class(vmSymbols::java_lang_ArithmeticException(), CHECK_0);
    initialize_class(vmSymbols::java_lang_StackOverflowError(), CHECK_0);
    initialize_class(vmSymbols::java_lang_IllegalMonitorStateException(), CHECK_0);
    initialize_class(vmSymbols::java_lang_IllegalArgumentException(), CHECK_0);
  }

  // See        : bugid 4211085.
  // Background : the static initializer of java.lang.Compiler tries to read
  //              property"java.compiler" and read & write property "java.vm.info".
  //              When a security manager is installed through the command line
  //              option "-Djava.security.manager", the above properties are not
  //              readable and the static initializer for java.lang.Compiler fails
  //              resulting in a NoClassDefFoundError.  This can happen in any
  //              user code which calls methods in java.lang.Compiler.
  // Hack :       the hack is to pre-load and initialize this class, so that only
  //              system domains are on the stack when the properties are read.
  //              Currently even the AWT code has calls to methods in java.lang.Compiler.
  //              On the classic VM, java.lang.Compiler is loaded very early to load the JIT.
  // Future Fix : the best fix is to grant everyone permissions to read "java.compiler" and
  //              read and write"java.vm.info" in the default policy file. See bugid 4211383
  //              Once that is done, we should remove this hack.
  initialize_class(vmSymbols::java_lang_Compiler(), CHECK_0);

  // More hackery - the static initializer of java.lang.Compiler adds the string "nojit" to
  // the java.vm.info property if no jit gets loaded through java.lang.Compiler (the hotspot
  // compiler does not get loaded through java.lang.Compiler).  "java -version" with the
  // hotspot vm says "nojit" all the time which is confusing.  So, we reset it here.
  // This should also be taken out as soon as 4211383 gets fixed.
  reset_vm_info_property(CHECK_0);

  quicken_jni_functions();

  // Must be run after init_ft which initializes ft_enabled
  if (TRACE_INITIALIZE() != JNI_OK) {
    vm_exit_during_initialization("Failed to initialize tracing backend");
  }

  // Set flag that basic initialization has completed. Used by exceptions and various
  // debug stuff, that does not work until all basic classes have been initialized.
  set_init_completed();

#ifndef USDT2
  HS_DTRACE_PROBE(hotspot, vm__init__end);
#else /* USDT2 */
  HOTSPOT_VM_INIT_END();
#endif /* USDT2 */

  // record VM initialization completion time
#if INCLUDE_MANAGEMENT
  Management::record_vm_init_completed();
#endif // INCLUDE_MANAGEMENT

  // Compute system loader. Note that this has to occur after set_init_completed, since
  // valid exceptions may be thrown in the process.
  // Note that we do not use CHECK_0 here since we are inside an EXCEPTION_MARK and
  // set_init_completed has just been called, causing exceptions not to be shortcut
  // anymore. We call vm_exit_during_initialization directly instead.
  SystemDictionary::compute_java_system_loader(THREAD);
  if (HAS_PENDING_EXCEPTION) {
    vm_exit_during_initialization(Handle(THREAD, PENDING_EXCEPTION));
  }

#if INCLUDE_ALL_GCS
  // Support for ConcurrentMarkSweep. This should be cleaned up
  // and better encapsulated. The ugly nested if test would go away
  // once things are properly refactored. XXX YSR
  if (UseConcMarkSweepGC || UseG1GC) {
    if (UseConcMarkSweepGC) {
      ConcurrentMarkSweepThread::makeSurrogateLockerThread(THREAD);
    } else {
      ConcurrentMarkThread::makeSurrogateLockerThread(THREAD);
    }
    if (HAS_PENDING_EXCEPTION) {
      vm_exit_during_initialization(Handle(THREAD, PENDING_EXCEPTION));
    }
  }
#endif // INCLUDE_ALL_GCS

  // Always call even when there are not JVMTI environments yet, since environments
  // may be attached late and JVMTI must track phases of VM execution
  JvmtiExport::enter_live_phase();

  // Signal Dispatcher needs to be started before VMInit event is posted
  os::signal_init();

  // Start Attach Listener if +StartAttachListener or it can't be started lazily
  if (!DisableAttachMechanism) {
    AttachListener::vm_start();
    if (StartAttachListener || AttachListener::init_at_startup()) {
      AttachListener::init();
    }
  }

  // Launch -Xrun agents
  // Must be done in the JVMTI live phase so that for backward compatibility the JDWP
  // back-end can launch with -Xdebug -Xrunjdwp.
  if (!EagerXrunInit && Arguments::init_libraries_at_startup()) {
    create_vm_init_libraries();
  }

  // Notify JVMTI agents that VM initialization is complete - nop if no agents.
  JvmtiExport::post_vm_initialized();

  if (TRACE_START() != JNI_OK) {
    vm_exit_during_initialization("Failed to start tracing backend.");
  }

  if (CleanChunkPoolAsync) {
    Chunk::start_chunk_pool_cleaner_task();
  }

  // initialize compiler(s)
#if defined(COMPILER1) || defined(COMPILER2) || defined(SHARK)
  CompileBroker::compilation_init();
#endif

  if (EnableInvokeDynamic) {
    // Pre-initialize some JSR292 core classes to avoid deadlock during class loading.
    // It is done after compilers are initialized, because otherwise compilations of
    // signature polymorphic MH intrinsics can be missed
    // (see SystemDictionary::find_method_handle_intrinsic).
    initialize_class(vmSymbols::java_lang_invoke_MethodHandle(), CHECK_0);
    initialize_class(vmSymbols::java_lang_invoke_MemberName(), CHECK_0);
    initialize_class(vmSymbols::java_lang_invoke_MethodHandleNatives(), CHECK_0);
  }

#if INCLUDE_MANAGEMENT
  Management::initialize(THREAD);
#endif // INCLUDE_MANAGEMENT

  if (HAS_PENDING_EXCEPTION) {
    // management agent fails to start possibly due to
    // configuration problem and is responsible for printing
    // stack trace if appropriate. Simply exit VM.
    vm_exit(1);
  }

  if (Arguments::has_profile())       FlatProfiler::engage(main_thread, true);
  if (MemProfiling)                   MemProfiler::engage();
  StatSampler::engage();
  if (CheckJNICalls)                  JniPeriodicChecker::engage();

  BiasedLocking::init();

  if (JDK_Version::current().post_vm_init_hook_enabled()) {
    call_postVMInitHook(THREAD);
    // The Java side of PostVMInitHook.run must deal with all
    // exceptions and provide means of diagnosis.
    if (HAS_PENDING_EXCEPTION) {
      CLEAR_PENDING_EXCEPTION;
    }
  }

  {
      MutexLockerEx ml(PeriodicTask_lock, Mutex::_no_safepoint_check_flag);
      // Make sure the watcher thread can be started by WatcherThread::start()
      // or by dynamic enrollment.
      WatcherThread::make_startable();
      // Start up the WatcherThread if there are any periodic tasks
      // NOTE:  All PeriodicTasks should be registered by now. If they
      //   aren't, late joiners might appear to start slowly (we might
      //   take a while to process their first tick).
      if (PeriodicTask::num_tasks() > 0) {
          WatcherThread::start();
      }
  }

  // Give os specific code one last chance to start
  os::init_3();

  create_vm_timer.end();
#ifdef ASSERT
  _vm_complete = true;
#endif
  return JNI_OK;
}
```
