


# Kubectl.go

1.logs.InitLogs()
k8s.io/kubernetes/pkg/util/logs 日志管理代码
每30秒刷新一次日志。 使用了github.com/golang/glog


2. NewDefaultClientConfigLoadingRules
currentMigrationRules



type Factory struct {
	clients *ClientCache
	flags   *pflag.FlagSet

	// Returns interfaces for dealing with arbitrary runtime.Objects.
	Object func() (meta.RESTMapper, runtime.ObjectTyper)
	// Returns interfaces for dealing with arbitrary
	// runtime.Unstructured. This performs API calls to discover types.
	UnstructuredObject func() (meta.RESTMapper, runtime.ObjectTyper, error)
	// Returns interfaces for decoding objects - if toInternal is set, decoded objects will be converted
	// into their internal form (if possible). Eventually the internal form will be removed as an option,
	// and only versioned objects will be returned.
	Decoder func(toInternal bool) runtime.Decoder
	// Returns an encoder capable of encoding a provided object into JSON in the default desired version.
	JSONEncoder func() runtime.Encoder
	// ClientSet gives you back an internal, generated clientset
	ClientSet func() (*internalclientset.Clientset, error)
	// Returns a RESTClient for accessing Kubernetes resources or an error.
	RESTClient func() (*restclient.RESTClient, error)
	// Returns a client.Config for accessing the Kubernetes server.
	ClientConfig func() (*restclient.Config, error)
	// Returns a RESTClient for working with the specified RESTMapping or an error. This is intended
	// for working with arbitrary resources and is not guaranteed to point to a Kubernetes APIServer.
	ClientForMapping func(mapping *meta.RESTMapping) (resource.RESTClient, error)
	// Returns a RESTClient for working with Unstructured objects.
	UnstructuredClientForMapping func(mapping *meta.RESTMapping) (resource.RESTClient, error)
	// Returns a Describer for displaying the specified RESTMapping type or an error.
	Describer func(mapping *meta.RESTMapping) (kubectl.Describer, error)
	// Returns a Printer for formatting objects of the given type or an error.
	Printer func(mapping *meta.RESTMapping, options kubectl.PrintOptions) (kubectl.ResourcePrinter, error)
	// Returns a Scaler for changing the size of the specified RESTMapping type or an error
	Scaler func(mapping *meta.RESTMapping) (kubectl.Scaler, error)
	// Returns a Reaper for gracefully shutting down resources.
	Reaper func(mapping *meta.RESTMapping) (kubectl.Reaper, error)
	// Returns a HistoryViewer for viewing change history
	HistoryViewer func(mapping *meta.RESTMapping) (kubectl.HistoryViewer, error)
	// Returns a Rollbacker for changing the rollback version of the specified RESTMapping type or an error
	Rollbacker func(mapping *meta.RESTMapping) (kubectl.Rollbacker, error)
	// Returns a StatusViewer for printing rollout status.
	StatusViewer func(mapping *meta.RESTMapping) (kubectl.StatusViewer, error)
	// MapBasedSelectorForObject returns the map-based selector associated with the provided object. If a
	// new set-based selector is provided, an error is returned if the selector cannot be converted to a
	// map-based selector
	MapBasedSelectorForObject func(object runtime.Object) (string, error)
	// PortsForObject returns the ports associated with the provided object
	PortsForObject func(object runtime.Object) ([]string, error)
	// ProtocolsForObject returns the <port, protocol> mapping associated with the provided object
	ProtocolsForObject func(object runtime.Object) (map[string]string, error)
	// LabelsForObject returns the labels associated with the provided object
	LabelsForObject func(object runtime.Object) (map[string]string, error)
	// LogsForObject returns a request for the logs associated with the provided object
	LogsForObject func(object, options runtime.Object) (*restclient.Request, error)
	// PauseObject marks the provided object as paused ie. it will not be reconciled by its controller.
	PauseObject func(object runtime.Object) (bool, error)
	// ResumeObject resumes a paused object ie. it will be reconciled by its controller.
	ResumeObject func(object runtime.Object) (bool, error)
	// Returns a schema that can validate objects stored on disk.
	Validator func(validate bool, cacheDir string) (validation.Schema, error)
	// SwaggerSchema returns the schema declaration for the provided group version kind.
	SwaggerSchema func(unversioned.GroupVersionKind) (*swagger.ApiDeclaration, error)
	// Returns the default namespace to use in cases where no
	// other namespace is specified and whether the namespace was
	// overridden.
	DefaultNamespace func() (string, bool, error)
	// Generators returns the generators for the provided command
	Generators func(cmdName string) map[string]kubectl.Generator
	// Check whether the kind of resources could be exposed
	CanBeExposed func(kind unversioned.GroupKind) error
	// Check whether the kind of resources could be autoscaled
	CanBeAutoscaled func(kind unversioned.GroupKind) error
	// AttachablePodForObject returns the pod to which to attach given an object.
	AttachablePodForObject func(object runtime.Object) (*api.Pod, error)
	// UpdatePodSpecForObject will call the provided function on the pod spec this object supports,
	// return false if no pod spec is supported, or return an error.
	UpdatePodSpecForObject func(obj runtime.Object, fn func(*api.PodSpec) error) (bool, error)
	// EditorEnvs returns a group of environment variables that the edit command
	// can range over in order to determine if the user has specified an editor
	// of their choice.
	EditorEnvs func() []string
	// PrintObjectSpecificMessage prints object-specific messages on the provided writer
	PrintObjectSpecificMessage func(obj runtime.Object, out io.Writer)
}


const (
	RunV1GeneratorName                          = "run/v1"
	RunPodV1GeneratorName                       = "run-pod/v1"
	ServiceV1GeneratorName                      = "service/v1"
	ServiceV2GeneratorName                      = "service/v2"
	ServiceNodePortGeneratorV1Name              = "service-nodeport/v1"
	ServiceClusterIPGeneratorV1Name             = "service-clusterip/v1"
	ServiceLoadBalancerGeneratorV1Name          = "service-loadbalancer/v1"
	ServiceAccountV1GeneratorName               = "serviceaccount/v1"
	HorizontalPodAutoscalerV1Beta1GeneratorName = "horizontalpodautoscaler/v1beta1"
	HorizontalPodAutoscalerV1GeneratorName      = "horizontalpodautoscaler/v1"
	DeploymentV1Beta1GeneratorName              = "deployment/v1beta1"
	DeploymentBasicV1Beta1GeneratorName         = "deployment-basic/v1beta1"
	JobV1Beta1GeneratorName                     = "job/v1beta1"
	JobV1GeneratorName                          = "job/v1"
	ScheduledJobV2Alpha1GeneratorName           = "scheduledjob/v2alpha1"
	NamespaceV1GeneratorName                    = "namespace/v1"
	ResourceQuotaV1GeneratorName                = "resourcequotas/v1"
	SecretV1GeneratorName                       = "secret/v1"
	SecretForDockerRegistryV1GeneratorName      = "secret-for-docker-registry/v1"
	SecretForTLSV1GeneratorName                 = "secret-for-tls/v1"
	ConfigMapV1GeneratorName                    = "configmap/v1"
)
