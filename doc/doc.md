# okdownload

## API

### 单个任务下载



```
DownloadTask task = new DownloadTask.Builder(url, parentFile)

                .setFilename(filename) //设置下载文件名，没提供的话先看 response header ，再看 url path(即启用下面那项配置)

                .setFilenameFromResponse(false) //是否使用 response header or url path 作为文件名，此时会忽略指定的文件名，默认false

                .setMinIntervalMillisCallbackProcess(150) // 下载进度回调的间隔时间为 150 ms

                .setConnectionCount(3) // 设置 3 个线程同时下载

                .setPassIfAlreadyCompleted(false) // 任务过去已完成是否要重新下载

                .setWifiRequired(false) //是否只允许wifi下载，默认为false

                .setAutoCallbackToUIThread(true) //是否在主线程通知调用者，默认为true

                .setHeaderMapFields(Map<String, List<String>> headerMapFields) //设置请求头

                .addHeader(String key, String value) //追加请求头

                .setPriority(0) //设置优先级，默认值是0，值越大下载优先级越高

                .setReadBufferSize(4096) //设置读取缓存区大小，默认4096

                .setFlushBufferSize(16384) //设置写入缓存区大小，默认16384

                .setSyncBufferSize(65536) //写入到文件的缓冲区大小，默认65536

                .build();
                
task.setTag();

String tag = (String) task.getTag();

// 异步执行任务
task.enqueue(listener);

// 取消任务
task.cancel();

// 同步执行任务
task.execute(listener);
```

### 多个任务下载
```
DownloadTask.enqueue(tasks, listener);

DownloadTask.cancel(tasks);
```

### 批量创建下载任务
```
DownloadContext.Builder builder = new DownloadContext.QueueSet()
        .setParentPathFile(parentFile)
        //.setParentPath(path)
        .setMinIntervalMillisCallbackProcess(150)
        //.setHeaderMapFields(Map<String, List<String>> headerMapFields)
        .setWifiRequired(false)
        .setReadBufferSize(4096)
        .setFlushBufferSize(16384)
        .setSyncBufferSize(65536)
        .setSyncBufferIntervalMillis(2000)
        .setAutoCallbackToUIThread(true)
        .setPassIfAlreadyCompleted(false)
        .commit();
        
DownloadTask task1 = builder.bind(url1); // 必须和 QueueSet 的 ParentPath 一致，生成的 DownloadTask 被加入到 QueueSet 中

builder.bind(url2).addTag(key, value);
builder.bind(url3).setTag(tag);

builder.setListener(contextListener);

DownloadTask task = new DownloadTask.Builder(url4, parentFile)
        .setPriority(10).build();
builder.bindSetTask(task);

DownloadContext context = builder.build();

//顺序开始任务
context.startOnSerial(listener);

//并行开始任务
context.startOnParallel(listener);

// stop
context.stop();

// 获取 Task 列表
DownloadTask[] context.getTasks()

```

### DownloadContextListener
```
interface DownloadContextListener {
    void taskEnd(DownloadContext context, DownloadTask task,
                 EndCause cause, Exception realCause, int remainCount);

    void queueEnd(DownloadContext context);
}
```

### 动态任务队列
```
DownloadSerialQueue serialQueue = new DownloadSerialQueue(downloadListener);

serialQueue.enqueue(task1);
serialQueue.enqueue(task2);

serialQueue.pause();

serialQueue.resume();

DownloadTask[] discardTasks = serialQueue.shutdown();

int workingTaskId = serialQueue.getWorkingTaskId();
int waitingTaskCount = serialQueue.getWaitingTaskCount();

```

### 合并 Listener

```
DownloadListener listener1 = new DownloadListener1();
DownloadListener listener2 = new DownloadListener2();

DownloadListener combinedListener = new DownloadListenerBunch.Builder()
                   .append(listener1)
                   .append(listener2)
                   .build();

DownloadTask task = new DownloadTask.build(url, file).build();
task.enqueue(combinedListener);
```

### Listener 与 Download 关系管理

```
UnifiedListenerManager manager = new UnifiedListenerManager();

DownloadListener listener1 = new DownloadListener1();
DownloadListener listener2 = new DownloadListener2();
DownloadListener listener3 = new DownloadListener3();
DownloadListener listener4 = new DownloadListener4();

DownloadTask task = new DownloadTask.build(url, file).build();

manager.attachListener(task, listener1);
manager.attachListener(task, listener2);

manager.detachListener(task, listener2);

manager.enqueueTaskWithUnifiedListener(task, listener3);
manager.executeTaskWithUnifiedListener(task, listener3);

manager.attachListener(task, listener4);
```


### BreakpointInfo
```
class BreakpointInfo {
int getId();
boolean isSingleBlock();
BlockInfo getBlock(int blockIndex);
long getTotalLength();
long getTotalOffset();
String getUrl();
public String getFilename();
}
```

### BlockInfo
```
class BlockInfo {
long getContentLength();
long getCurrentOffset();
long getRangeLeft();
long getRangeRight();
}
```

#### DownloadListener 

```
interface DownloadListener {
void taskStart(DownloadTask task);
void connectTrialStart(DownloadTask task, Map<String, List<String>> requestHeaderFields);
void connectTrialEnd(DownloadTask task, int responseCode, Map<String, List<String>> responseHeaderFields);
void downloadFromBeginning(DownloadTask task, BreakpointInfo info,  ResumeFailedCause cause);
void downloadFromBreakpoint(DownloadTask task, BreakpointInfo info)
void connectStart(DownloadTask task, int blockIndex, Map<String, List<String>> requestHeaderFields);
void connectEnd(DownloadTask task, int blockIndex, int responseCode, Map<String, List<String>> responseHeaderFields);
void fetchStart(DownloadTask task, int blockIndex, long contentLength);
void fetchProgress(DownloadTask task, int blockIndex, long increaseBytes);
void fetchEnd(DownloadTask task, int blockIndex, long contentLength);
void taskEnd(DownloadTask task, EndCause cause, Exception realCause);
}

```

#### DownloadListener1

```
class DownloadListener1 {
void taskStart(DownloadTask task, Listener1Assist.Listener1Model model);
void retry(DownloadTask task, ResumeFailedCause cause);
void connected(DownloadTask task, int blockCount, long currentOffset, long totalLength);
void progress(DownloadTask task, long currentOffset, long totalLength);
void taskEnd(DownloadTask task, EndCause cause, Exception realCause, Listener1Assist.Listener1Model model);
}
```

#### DownloadListener2

```
class DownloadListener2 {
void taskStart(DownloadTask task);
void taskEnd(DownloadTask task, EndCause cause, Exception realCause);
}
```

#### DownloadListener3

```
class DownloadListener3 {
void started(DownloadTask task);
void completed(DownloadTask task);
void canceled(DownloadTask task);
void error(DownloadTask task, Exception e);
void warn(DownloadTask task);
void retry(DownloadTask task, ResumeFailedCause cause);
void connected(DownloadTask task, int blockCount, long currentOffset, long totalLength);
void progress(DownloadTask task, long currentOffset, long totalLength);
}       
```

#### DownloadListener4

```
class DownloadListener4 {
void taskStart(DownloadTask task);
void connectStart(DownloadTask task, int blockIndex, Map<String, List<String>> requestHeaderFields);
void connectEnd(DownloadTask task, int blockIndex, int responseCode, Map<String, List<String>> responseHeaderFields);
void infoReady(DownloadTask task, BreakpointInfo info, boolean fromBreakpoint, Listener4Assist.Listener4Model model);
void progressBlock(DownloadTask task, int blockIndex, long currentBlockOffset)
void progress(DownloadTask task, long currentOffset);
void blockEnd(DownloadTask task, int blockIndex, BlockInfo info);
void taskEnd(DownloadTask task, EndCause cause, Exception realCause, Listener4Assist.Listener4Model model);
}
```

#### DownloadListener4WithSpeed

```
class DownloadListener4WithSpeed {
void taskStart(DownloadTask task);
void connectStart(DownloadTask task, int blockIndex, Map<String, List<String>> requestHeaderFields);
void connectEnd(DownloadTask task, int blockIndex, int responseCode, Map<String, List<String>> responseHeaderFields);
void infoReady(DownloadTask task, BreakpointInfo info, boolean fromBreakpoint, Listener4SpeedAssistExtend.Listener4SpeedModel model);
void progressBlock(DownloadTask task, int blockIndex, long currentBlockOffset, SpeedCalculator blockSpeed);
void progress(DownloadTask task, long currentOffset, SpeedCalculator taskSpeed);
void blockEnd(DownloadTask task, int blockIndex, BlockInfo info, SpeedCalculator blockSpeed);
void taskEnd(DownloadTask task, EndCause cause, Exception realCause, SpeedCalculator taskSpeed);
}
```

![](https://docs.qnsdk.com/listener.png)


### 下载步骤

1. 任务的调度和检查 ：DownloadDispatcher
	- 检查本地是否已经存在 passIfAlreadyCompleted ? COMPLETED : 重新下载
	- 检查任务是否冲突，是否有相同的任务已经在任务队列里面了 ：SAME_TASK_BUSY
	- 检查文件是否冲突，是否有两个任务的 uri 相同 ：FILE_BUSY
	- 通过一个 DownloadTask 创建一个 DownloadCall
	- 检查当前正在并行的任务数量是否超过最大并行数量，如果超过则加入到准备队列中，没有超过就加入到运行队列中，并调用 DownloadCall 的 execute 方法

2. 一个任务真正开始下载执行 ：DownloadCall  
	- 任务开始，进行回调 taskStart
	- 检查 task 的参数是否合理（url length > 0 ?）
	- 从数据库里拿断点信息 BreakpointInfo，如果找不到则创建一个新的
	- 创建一个 DownloadConnection 与服务器进行建立连接（Range ：bytes=0-0）
	- 对远端服务器进行检查，检查是否支持断点续传，并获取文件大小， etag，是否 chunked 等信息
	- 从远端服务器获取信息来检查是否出现异常
	- 检查本地文件和断点信息是否匹配，比如本地文件是否已经被删除
	- 如果不允许恢复
		- 把本地文件删除
		- 重新把一个任务平均分块（默认策略 ：文件大小在0-1MB、1-5MB、5-50MB、50-100MB、100MB以上时分别开启1、2、3、4、5个线程进行下载。）然后把信息更新到 BreakpointInfo 中
		- 回调用户 ：downloadFromBeginning
		- 通过 task 中 BreakpointInfo 开始下载
	- 如果允许恢复，则从断点处开始下载
		- 回调用户 ：downloadFromBreakpoint
		- 通过 task 中 BreakpointInfo 开始下载
	- 通过 BreakpointInfo 中的信息把一个 DownloadCall 分解为多个 block 子任务进行下载

3. 一个 block 开始下载执行 ：DownloadChain
	- 创建一个 DownloadConnection ，如果有重定向，则直接用重定向之后的地址进行创建
	- 增加 header，包括 User-Agent ，Range ，If-Match等
	- 开始建立连接，回调 connectStart
	- 链接结束，得到 ResponseCode，ResponseHeaderFields 等信息，并回调 connectEnd
	- 再次进行异常检查，比如只有一个 block 的时候，response 获取的 length 与 info 的 length 不一致，则更新 info ，并重新下载
	- 如果没有异常，开始写文件
	- outputStream 要先进行 seek 到 Range left 处再开始写入
	-  所有的 block 下载完，任务结束
