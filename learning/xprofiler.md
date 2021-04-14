### 如何收集进程数据
xprofiler 中的 c++ 代码进行收集数据
然后写入到 
error.log
xprofiler.log
文件中

### 如何收集 libuv 数据
需要 引入 libuv.h uv.h 等数据，通过 v8 引擎的结构得到数据。
### 如何收集性能数据，传送性能数据
xtransit 中有解析 xprofiler.log 的文件
xtransit 中的 commands/upload_file.js 会进行上传文件的操作

xtransit 的工作
连接  xtransit-server
与之进行通信
处理对应的 message 在 lib/handler.js 中
xtransit-server 可能发送的命令有
```
shutdown
exec_command [commands目录下的文件名]
```
```
 const url = `${server}/xapi/upload_from_xtransit?${qs.stringify({ fileId, fileType, nonce, timestamp, signature })}`;
```
可以通过 upload 命令上传文件到 xprofiler-console 中

启动 monitor 定时执行 [orders]下面的命令
在执行之后，会将对应的内容     this.sendMessage('log', data);

### xtransit-server
暴露的三个接口
```
  router.post('/xapi/shutdown', apiController.shutdown);
  router.post('/xapi/check_client_alive', apiController.checkClientAlive);
  router.post('/xapi/exec_command', apiController.execCommand);
```
exec_command 是指向  ws 连接的 xtransit 的命令接口

本身处理的信息，会在 handler/message.js 中，获取到内容后，
通过 service/manager.js 中继续通过 proxy/request.js 不断地传递对应
type 的 message ，最后会，统一将 日志 log 发送到 xtransit-manager 中进行处理

### xtransit-manager
一个 egg 项目
会 暴露接口给 xtransit-server 进行 client  控制
/xtransit/app_secret
```js
 // xprofiler-console
  router.post('/xprofiler/clients', checkSign, checkParams(['appId']), 'xprofiler.getClients');
  router.post('/xprofiler/files', checkSign, checkParams(['appId', 'agentId', 'type']), 'xprofiler.getFiles');
  router.post('/xprofiler/errors', checkSign, checkParams(['appId', 'agentId', 'errorFile', 'currentPage', 'pageSize']), 'xprofiler.getErrors');
  router.post('/xprofiler/modules', checkSign, checkParams(['appId', 'agentId', 'moduleFile']), 'xprofiler.getModules');
  // commands
  router.post('/xprofiler/agent_osinfo', checkSign, checkParams(['appId', 'agentId']), 'xprofiler.getAgentOsInfo');
  router.post('/xprofiler/agent_node_processes', checkSign, checkParams(['appId', 'agentId']), 'xprofiler.getAgentNodeProcesses');
  router.post('/xprofiler/check_process_status', checkSign, checkParams(['appId', 'agentId', 'pid']), 'xprofiler.checkProcessStatus');
  router.post('/xprofiler/check_processes_alive', checkSign, checkParams(['appId', 'agentId', 'pids']), 'xprofiler.checkProcessesAlive');
  router.post('/xprofiler/check_file_status', checkSign, checkParams(['appId', 'agentId', 'filePath']), 'xprofiler.checkFileStatus');
  router.post('/xprofiler/take_action', checkSign, checkParams(['appId', 'agentId', 'pid', 'command', 'options']), 'xprofiler.takeAction');
  router.post('/xprofiler/transfer_file', checkSign, checkParams(['appId', 'agentId', 'fileId', 'fileType', 'filePath', 'server', 'token']), 'xprofiler.transferFile');

  // xtransit-server
  router.post('/xtransit/app_secret', checkSign, checkParams(['appId']), 'xtransit.getAppSecret');
  router.post('/xtransit/update_client', checkSign, checkParams(['appId', 'agentId', 'clientId', 'server', 'timestamp']), 'xtransit.updateClient');
  router.post('/xtransit/remove_client', checkSign, checkParams(['appId', 'agentId', 'clientId']), 'xtransit.removeClient');
  router.post('/xtransit/log', checkSign, checkParams(['appId', 'agentId', 'log']), 'xtransit.handleLog');
  router.post('/xtransit/update_action_status', checkSign, checkParams(['appId', 'agentId', 'filePath']), 'xtransit.updateActionStatus');
```

有一点比较特别的是：
使用  redis 存储日志文件，过时会进行清除。
比如说， error,package

### 命令

  xprofctl start_cpu_profiling   启动 cpu 采样
  xprofctl stop_cpu_profiling    生成 cpuprofile
  xprofctl start_heap_profiling  启动 heap 采样
  xprofctl stop_heap_profiling   生成 heapprofile
  xprofctl start_gc_profiling    启动 gc 采样
  xprofctl stop_gc_profiling     生成 gcprofile
  xprofctl heapdump              生成 heapsnapshot
  xprofctl diag_report           生成诊断报告
  xprofctl check_version         获取 xprofiler 版本号
  xprofctl get_config            获取 xprofiler 配置
  xprofctl set_config            设置 xprofiler 配置

### xprofiler-console
展示页面，有 redis, mysql 数据库
