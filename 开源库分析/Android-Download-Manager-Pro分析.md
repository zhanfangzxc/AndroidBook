###Android-Download-Manager-Pro分析

>类

- DataBaseHelper extends SQLiteOpenHelper

	- 创建Task表
	- 创建Chunk表

- ChunkDataSource：对Chunk表的一些操作

	- insertChunks(Task task):插入块
	- chunkRelatedTask(int taskID):根据taskID查询块
	- delete(int chunkID):根据chunkId删除
	- close():关闭数据库

- TaskDataSource：对Task表的一些操作

	- openDatabase(DataBaseHelper dbHelper):开启数据库
	- insertTask(Task task):插入task
	- update(Task task):更新task
	- getTasksInState(int state):根据状态查询task
	- getUnnotifiedCompleted():获取未通知完成的task
	- getUnCompletedTasks(int priority):根据优先级查询未完成的task
	- getTaskInfo(int id):根据id获取task信息
	- getTaskInfoWithName(String name):根据name获取task信息
	- delete(int taskID):根据id删除task
	- containsTask(String name):是否包含该名字的Task
	- checkUnNotifiedTask():将unNotified的task的状态更新为notify

- TaskState：task状态

	- init:初始化
	- ready:准备好
	- downloading:正在下载
	- paused:暂停
	- download_finished:下载结束
	- end

- QueueSort:队列排序

	- HighPriority:高优先级
	- LowPriority:低优先级
	- HighToLowPriority:优先级从高到低排序
	- LowToHighPriority:优先级从低到高排序
	- earlierFirst:任务从前到后排序
	- oldestFirst:任务从后到前排序

- DownloadManagerListenerModerator:下载管理监听器

	- onDownloadStart(long taskID) 开始下载
	- onDownloadPause(long taskID) 暂停下载
	- onDownProcess(long taskId,precent,downloadLength) 进度更新
	- onDownloadFinished(long taskID)  下载完成
	- onDownloadRebuildStart(long taskId)  开始重建文件
	- onDownloadRebuildFinished(long taskId)  重建文件结束
	- onDownloadCompleted(long taskID) 下载所有的操作完成
	- connectionLost(long taskId) 断开连接

- QueueObserver

	- wakeUp(int taskID):唤醒taskId的task

- QueueModerator implements QueueObserver

	- startQueue():开启下载队列
	- wakeUp():重新开启下载队列
	- pause():暂停下载队列

- Moderator

	- setQueueObserver(QueueModerator queueObserver):设置监听器
	- start(Task task,DownloadManagerListenerModerator listener):开始下载task
	- pause(int taskID):根据taskId暂停task
	- connectionLost(int taskID):断开连接
	- downloadByteThreshold:已下载的大小
	- process(int taskId,long byteRead):计算已经下载的百分比
	- rebuild(Chunk chunk):？？？？？？？？？？？
	- rebuildIsDone(Task task,List<Chunk> taskChunks):????????????
	- wakeUpObserver(int taskID):唤醒taskId的task

- AsyncStartDownload extends Thread

	- run():在task的各种状态下做一些处理
	- getTaskFileInfo(Task task):获取任务的文件信息
	- convertTaskToChunks(Task task):将Task转换成Chunks
	- makeFileForChunks(int firstId,Task task):创建块文件
	- deleteChunk(Task task):根据Task删除Chunk
	- generateNewChunk(Task task)

- AsyncWorker:执行下载任务的thread

	- run():下载任务

	```
	@Override
	public void run(){
		URL url = new URL(task.url);
		HttpUrlConnection connection = (HttpURLConnection)url.openConnection();
		connection.setConnectionTimeout(1000);
		connection.setReadTimeout(1000);
		if(chunk.end != 0){
			connection.setRequestProperty("Range","bytes="+chunk.begin+"-"+chunk.end);
		}
		connection.connect();
		File cf = new File(FileUtils.address(task.save_address,String.valueOf(chunk.id)));
		InputStream remoteFileIn = connection.getInputStream();
		FileOutputStream chunkFile = new FileOutputStream(cf,true);
		int len = 0;
		watchDog = new ConnectionWatchDog(5000,this);
		watchDog.start();
		while(!this.isInterrupted()&&(len = remoteFileIn.read(buffer))>0){
			watchDog.reset();
			chunkFile.write(buffer,0,len);			process(len);	//更新进度
		}
		chunkFile.flush();
		chunkFile.close();
		watchDog.interrupt();
		connection.disconnect();
		if(!this.isInterrupted()){
			observer.rebuild(chunk);
		}
	}catch(SocketTimeoutException e){
		e.printStackTrace();
		observer.connectionLost(task.id);
		pauseRelatedTask();
	}catch(Exception){
		e.printStackTrace();
	}
	```
	- process(int read):计算进度
	- pauseRelatedTask():暂停task
	- connectionTimeOut():连接超时

- Rebuilder extends Thread

	```
	@Override
	public void run(){
		observer.downloadManagerListener.OnDownloadRebuildStart(task.id);
		File file = FileUtils.create(task.save_address,task.name+"."+task.extension);
		FileOutputStream finalFile = null;
		try{
			finalFile = new FileOutputStream(file,true);
		}catch(FileNotFoundException e){
			e.printStackTrace();
		}

		byte[] readBuffer = new byte[1024];
		int read = 0;
		for(Chunk chunk:taskChunks){
			FileInputStream chFileIn = FileUtils.getInputStream(task.save_address,String.valueOf(chunk.id));
			try{
				while((read == chFile.read(readBuffer))>0){
					finalFile.write(readBuffer,0,read);
				}
			}catch(IOException e){
				e.printStackTrace();
			}

			if(finalFile != null){
				try{
					finalFile.flush();
				}catch(IOException e){
					e.printStackTrace();
				}
			}
		}
		observer.reBuildIsDone(task,taskChunks);
	}
	```

- DownloadManagerPro:下载管理器

	- init(String sdCardFolderAddress,int maxChunks,DownloadManagerListener listener):初始化
	- addTask(String saveName,String url,int chunk,String sdCardFolderAddress,boolean overrite,boolean priority):添加task
	- startDownload(int token):开始下载
	- startQueueDowbload(int downloadTaskPerTime,int priority):开启队列下载
	- pauseDownload(int token):暂停下载
	- pauseQueueDownload():暂停队列
	- singleDownloadStatus(int token):报告下载状态
	- downloadTasksInSameState(int state):下载想通状态的任务
	- lastCompletedDownloads():返回刚完成的下载列表
	- readTaskList(List<Task> tasks)
	- notifiedTaskChecked():检查没有notified的task的状态
	- delete(int token,boolean deleteTaskFile):删除task信息和task文件
	- dispose():关闭数据库
	- uncompleted():返回未完成的task列表
	- insertNewTask(String taskName,String url,int chunk,String save_address,boolean priority):向task表中插入一条task信息
	- setMaxChunk(int chunk):设置块数量
	- getUniqueName(string name):返回唯一的文件名
	- deleteSameDownloadNameTask(String saveName):删除相同文件名的task

> [使用说明](https://github.com/majidgolshadi/Android-Download-Manager-Pro)
