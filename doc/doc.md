# okdownload

## API

### 单个任务下载



```
task = new DownloadTask.Builder(url, parentFile)

                .setFilename(filename) //设置下载文件名，没提供的话先看 response header ，再看 url path(即启用下面那项配置)

                .setFilenameFromResponse(false) //是否使用 response header or url path 作为文件名，此时会忽略指定的文件名，默认false

                .setMinIntervalMillisCallbackProcess(150) // 下载进度回调的间隔时间为 150 ms

                .setConnectionCount(3) // 设置 3 个线程同时下载

                .setPassIfAlreadyCompleted(false) // 任务过去已完成是否要重新下载

                .setWifiRequired(false) //是否只允许wifi下载，默认为false

                .setAutoCallbackToUIThread(true) //是否在主线程通知调用者，默认为true

                //.setHeaderMapFields(Map<String, List<String>> headerMapFields) //设置请求头

                //.addHeader(String key, String value) //追加请求头

                .setPriority(0) //设置优先级，默认值是0，值越大下载优先级越高

                .setReadBufferSize(4096) //设置读取缓存区大小，默认4096

                .setFlushBufferSize(16384) //设置写入缓存区大小，默认16384

                .setSyncBufferSize(65536) //写入到文件的缓冲区大小，默认65536

                .setSyncBufferIntervalMillis(2000) //写入文件的最小时间间隔，默认2000

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


### 动态任务队列
```
DownloadSerialQueue serialQueue = new DownloadSerialQueue(commonListener);

serialQueue.enqueue(task1);
serialQueue.enqueue(task2);

serialQueue.pause();

serialQueue.resume();

int workingTaskId = serialQueue.getWorkingTaskId();
int waitingTaskCount = serialQueue.getWaitingTaskCount();

DownloadTask[] discardTasks = serialQueue.shutdown();
```

#### DownloadListener 

```
interface DownloadListener {
void taskStart(@NonNull DownloadTask task);
void connectTrialStart(@NonNull DownloadTask task, @NonNull Map<String, List<String>> requestHeaderFields);
void connectTrialEnd(@NonNull DownloadTask task, int responseCode, @NonNull Map<String, List<String>> responseHeaderFields);
void downloadFromBeginning(@NonNull DownloadTask task, @NonNull BreakpointInfo info, @NonNull ResumeFailedCause cause);
void downloadFromBreakpoint(@NonNull DownloadTask task, @NonNull BreakpointInfo info)
void connectStart(@NonNull DownloadTask task, int blockIndex, @NonNull Map<String, List<String>> requestHeaderFields);
void connectEnd(@NonNull DownloadTask task, int blockIndex, int responseCode, @NonNull Map<String, List<String>> responseHeaderFields);
void fetchStart(@NonNull DownloadTask task, int blockIndex, long contentLength);
void fetchProgress(@NonNull DownloadTask task, int blockIndex, long increaseBytes);
void fetchEnd(@NonNull DownloadTask task, int blockIndex, long contentLength);
void taskEnd(@NonNull DownloadTask task, @NonNull EndCause cause, @Nullable Exception realCause);
}

```

#### DownloadListener1

```
class DownloadListener1 {
void taskStart(@NonNull DownloadTask task, @NonNull Listener1Assist.Listener1Model model);
void retry(@NonNull DownloadTask task, @NonNull ResumeFailedCause cause);
void connected(@NonNull DownloadTask task, int blockCount, long currentOffset, long totalLength);
void progress(@NonNull DownloadTask task, long currentOffset, long totalLength);
void taskEnd(@NonNull DownloadTask task, @NonNull EndCause cause, @Nullable Exception realCause, @NonNull Listener1Assist.Listener1Model model);
}
```

#### DownloadListener2

```
class DownloadListener2 {
void taskStart(@NonNull DownloadTask task);
void taskEnd(@NonNull DownloadTask task, @NonNull EndCause cause, @Nullable Exception realCause);
}
```

#### DownloadListener3

```
class DownloadListener3 {
void started(@NonNull DownloadTask task);
void completed(@NonNull DownloadTask task);
void canceled(@NonNull DownloadTask task);
void error(@NonNull DownloadTask task, @NonNull Exception e);
void warn(@NonNull DownloadTask task);
void retry(@NonNull DownloadTask task, @NonNull ResumeFailedCause cause);
void connected(@NonNull DownloadTask task, int blockCount, long currentOffset, long totalLength);
void progress(@NonNull DownloadTask task, long currentOffset, long totalLength);
}       
```

#### DownloadListener4

```
class DownloadListener4 {
void taskStart(@NonNull DownloadTask task);
void connectStart(@NonNull DownloadTask task, int blockIndex, @NonNull Map<String, List<String>> requestHeaderFields);
void connectEnd(@NonNull DownloadTask task, int blockIndex, int responseCode, @NonNull Map<String, List<String>> responseHeaderFields);
void infoReady(DownloadTask task, @NonNull BreakpointInfo info, boolean fromBreakpoint, @NonNull Listener4Assist.Listener4Model model);
void progressBlock(DownloadTask task, int blockIndex, long currentBlockOffset)
void progress(DownloadTask task, long currentOffset);
void blockEnd(DownloadTask task, int blockIndex, BlockInfo info);
void taskEnd(DownloadTask task, EndCause cause, @Nullable Exception realCause, @NonNull Listener4Assist.Listener4Model model);
}
```

#### DownloadListener4WithSpeed

```
class DownloadListener4WithSpeed {
void taskStart(@NonNull DownloadTask task);
void connectStart(@NonNull DownloadTask task, int blockIndex, @NonNull Map<String, List<String>> requestHeaderFields);
void connectEnd(@NonNull DownloadTask task, int blockIndex, int responseCode, @NonNull Map<String, List<String>> responseHeaderFields);
void infoReady(@NonNull DownloadTask task, @NonNull BreakpointInfo info, boolean fromBreakpoint, @NonNull Listener4SpeedAssistExtend.Listener4SpeedModel model);
void progressBlock(@NonNull DownloadTask task, int blockIndex, long currentBlockOffset, @NonNull SpeedCalculator blockSpeed);
void progress(@NonNull DownloadTask task, long currentOffset, @NonNull SpeedCalculator taskSpeed);
void blockEnd(@NonNull DownloadTask task, int blockIndex, BlockInfo info, @NonNull SpeedCalculator blockSpeed);
void taskEnd(@NonNull DownloadTask task, @NonNull EndCause cause, @Nullable Exception realCause, @NonNull SpeedCalculator taskSpeed);
}
```

![][listener_img]

[listener_img]: http://ps7907cdy.bkt.clouddn.com/listener.png



### 下载步骤

1. 下载前检查 ：DownloadDispatcher
	- 检查本地是否已经存在 passIfAlreadyCompleted ? COMPLETED : 重新下载
	- 检查任务是否冲突，是否有相同的任务已经在任务队列里面了 ：SAME_TASK_BUSY
	- 检查文件是否冲突，是否有两个任务的 url 相同 ：FILE_BUSY
	- 通过一个 DownloadTask 创建一个 DownloadCall
	- 检查当前正在并行的任务数量是否超过最大并行数量，如果超过则加入到 readyAsyncCalls，否则就加入到 runningAsyncCalls，并 execute

2. 一个任务下载执行 ：DownloadCall  
	- 任务开始，进行回调 taskStart
	- 检查 task 的参数是否合理（url length > 0 ?）
	- 从数据库里拿断点信息，如果找不到则创建一个新的，并合 task 绑定
	- 创建一个 DownloadConnection 与服务器进行建立连接
	- 对远端服务器进行检查，检查是否支持断点续传，并获取文件大小， etag，是否 chunked 等信息
	- 从远端服务器来检查之前的断点信息是否能够恢复，比如之前检查的 http response 是 201，205 则无法恢复
	- 从本地文件和断点信息来检查是否能够恢复，比如本地文件是否已经被删除，断点信息是否和本地文件不匹配，则不允许恢复
	- 如果不允许恢复
		- 把本地文件删除
		- 重新把一个任务平均分块（默认策略 ：文件大小在0-1MB、1-5MB、5-50MB、50-100MB、100MB以上时分别开启1、2、3、4、5个线程进行下载。）然后把信息更新到 BreakpointInfo 中
		- 回调用户 ：downloadFromBeginning
		- 通过 task 中 BreakpointInfo 开始下载
	- 如果允许恢复，则从断点处开始下载
		- 回调用户 ：downloadFromBeginning
		- 通过 task 中 BreakpointInfo 开始下载
	- 通过  BreakpointInfo 中的信息把一个 DownloadCall 分解为多个 block 子任务进行下载

3. 一个 block 的下载执行 ：DownloadChain
	- 创建一个 DownloadConnection ，如果有重定向，则直接用重定向之后的地址进行创建
	- 增加 header，包括 User-Agent ，Range ，If-Match等
	- 开始建立连接，回调 connectStart
	- 链接结束，得到 ResponseCode，ResponseHeaderFields 等信息，并回调 connectEnd
	- 检查是否能够恢复
	- 更新信息 responseContentLength
	- 异常判定：block num 为 1 时，response 获取的 length 与存储的 info 的 length 不一致，则更新 info ，并重新下载
	-  如果没有异常，开始写文件
	-  outputStream 要先进行 seek 到 Range 处再开始写入


