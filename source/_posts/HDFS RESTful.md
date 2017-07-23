---
title: HDFS RESTful API
tags: Hadoop
categories: bigdata
---

### HDFS RESTful API
Port: 50070  也是 hdfs web portal的端口号.
```
webhdfs://<HOST>:<HTTP_PORT>/<PATH>
hdfs://<HOST>:<RPC_PORT>/<PATH>
http://<HOST>:<HTTP_PORT>/webhdfs/v1/<PATH>?op=...
```

#### Authenticaiton
1. Authentication when security is off:
```bash
curl -i "http://<HOST>:<PORT>/webhdfs/v1/<PATH>?[user.name=<USER>&]op=…"
```
2. Authentication using Kerberos SPNEGO when security is on:
```bash
curl -i --negotiate -u : "http://<HOST>:<PORT>/webhdfs/v1/<PATH>?op=..."
```
3. Authentication using Hadoop delegation token when security is on:
```bash
curl -i "http://<HOST>:<PORT>/webhdfs/v1/<PATH>?delegation=<TOKEN>&op=..."
```
#### FileSystem operations:  
List  a  Directory :  
```bash
curl -i  "http://<HOST>:<PORT>/webhdfs/v1/<PATH>?op=LISTSTATUS"
```
Status of  a  File/Directory :  
```bash
curl -i  "http://<HOST>:<PORT>/webhdfs/v1/<PATH>?op=GETFILESTATUS"
```   
Make  a  Directory :  
```bash
curl -i -X PUT "http://<HOST>:<PORT>/webhdfs/v1/<PATH>?op=MKDIRS[&permission=<OCTAL>]"    
```
Delete  a  File/Directory :  
```bash
curl -i -X DELETE "http://<host>:<port>/webhdfs/v1/<path>?op=DELETE[&recursive=<true|false>]"
```
Rename  a  File/Directory :  
```bash
curl -i -X PUT "<HOST>:<PORT>/webhdfs/v1/<PATH>?op=RENAME&destination=<PATH>"    
```
Open and Read  a  File :  
```bash
curl -i -L "http://<HOST>:<PORT>/webhdfs/v1/<PATH>?op=OPEN[&offset=<LONG>][&length=<LONG>][&buffersize=<INT>]"
```
Create and Write to  a  File :  
```bash
curl -i -X PUT "http://<HOST>:<PORT>/webhdfs/v1/<PATH>?op=CREATE
                    [&overwrite=<true|false>][&blocksize=<LONG>][&replication=<SHORT>]
                    [&permission=<OCTAL>][&buffersize=<INT>]"
curl -i -X PUT -T <LOCAL_FILE> "http://<DATANODE>:<PORT>/webhdfs/v1/<PATH>?op=CREATE..."
```
Append to  a  File :  
```bash
curl -i -X POST "http://<HOST>:<PORT>/webhdfs/v1/<PATH>?op=APPEND[&buffersize=<INT>]"
curl -i -X POST -T <LOCAL_FILE> "http://<DATANODE>:<PORT>/webhdfs/v1/<PATH>?op=APPEND…"
```
Concatenate Files :  
```bash
curl -i -X POST "http://<HOST>:<PORT>/webhdfs/v1/<PATH>?op=CONCAT&sources=<PATHS>"
```
#### Other File System Operations :  
Get Content Summary of  a Directory :  
```bash
curl -i "http://<HOST>:<PORT>/webhdfs/v1/<PATH>?op=GETCONTENTSUMMARY"
```
Get File Checksum :  
```bash
curl -i "http://<HOST>:<PORT>/webhdfs/v1/<PATH>?op=GETFILECHECKSUM"
```
Get Home Directory :  
```bash
curl -i "http://<HOST>:<PORT>/webhdfs/v1/?op=GETHOMEDIRECTORY"
```
Set Permission :
```bash
curl -i -X PUT "http://<HOST>:<PORT>/webhdfs/v1/<PATH>?op=SETPERMISSION[&permission=<OCTAL>]"
```
Set Owner :
```bash
curl -i -X PUT "http://<HOST>:<PORT>/webhdfs/v1/<PATH>?op=SETOWNER
                              [&owner=<USER>][&group=<GROUP>]"
```
Set Replication Factor :
```bash
curl -i -X PUT "http://<HOST>:<PORT>/webhdfs/v1/<PATH>?op=SETREPLICATION
                              [&replication=<SHORT>]"
```
Set Access or Modification Time :
```bash
curl -i -X PUT "http://<HOST>:<PORT>/webhdfs/v1/<PATH>?op=SETTIMES
                              [&modificationtime=<TIME>][&accesstime=<TIME>]"
```
#### Delegation Token Operations :
Get Delegation Token :
```bash
curl -i "http://<HOST>:<PORT>/webhdfs/v1/?op=GETDELEGATIONTOKEN&renewer=<USER>"
```
Renew Delegation Token :
```bash
curl -i -X PUT "http://<HOST>:<PORT>/webhdfs/v1/?op=RENEWDELEGATIONTOKEN&token=<TOKEN>"
```
Cancel Delegation Token :
```bash
curl -i -X PUT "http://<HOST>:<PORT>/webhdfs/v1/?op=CANCELDELEGATIONTOKEN&token=<TOKEN>"
```
