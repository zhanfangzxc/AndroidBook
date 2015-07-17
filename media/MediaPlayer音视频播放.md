###MediaPlayer

MediaPlayer的状态转换

![MediaPlayer的状态转换](../img/mediaplayer.gif)

- started/paused到stopped是单向转换，无法从stopped直接转换到started，需要经历prepared重新装载才可以重新播放。
- Initialized状态需要装载数据才可以进行start播放，如果是使用prepareAsync异步装载，需要在准备完成才可以播放，所以需要一个回调方法setOnPreparedListener,会在异步装载完成后调用。
- End状态是，当不在需要播放的时候，使用release()进行释放
- Error，一般会监听setOnErrorListener()回调方法。

> MediaPlayer的唤醒锁机制

当系统处于睡眠状态的时候，设置唤醒锁，可以保持硬件的正常工作

```
MediaPlayer.setWakeMode(getApplicationContext,PowerManager.PARTIAL_WAKE_LOCK);

//添加权限
<use-permission android:name="android.permission.WAKE_LOCK"/>
```

> Wifi锁定

```
wifiLock = ((WifiManager)getSystemService(this.WIFI_SERVICE)).createWifiLock(WifiManager.WIFI_MODE_FULL,"mylock");
//需要在mediaPlayer释放的时候解锁
wifiLock.acquire();
```
> 音频焦点

使用AudioManager.requestAudioFocus()方法设定音频焦点

```
int requestAudioFocus(AudioManager.OnAudioFocusChangeListener i,int streamType,int durationHint)
```
- AudioManager.OnAudioFocusChangeListener为音频焦点变化的回调函数，需要实现一个方法onAudioFocusChange(int focusChange)在音频焦点变换的时候回调
- focusChange有一下几种状态码

```
AUDIOFOCUS_GAIN 获得音频焦点
AUDUIFOCUS_LOSS 失去音频焦点，并且会持续很长时间
AUDIOFOCUS_LOSS_TRANSIENT 失去音频焦点，但不会持续很长时间，需要暂停MediaPlayer的播放，等待重新获取音频焦点
AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK:暂时失去音频焦点，但是无需停止播放
```
具体用法

```
audioManager = (AUDIOMANAGER)getSystemService(this.AUDIO_SERVICE);
int result = audioManager.requestAudioState(new OnAudioFocusChangeListener(){
		@Override
		public void onAudioFocusChange(int focusChange){
			switch(focusChange){
				case AudioManager.AUDIOFOCUS_GAIN:
					//获得音频焦点
					if(!mediaPlayer.isPlaying()){
						mediaPlayer.start();
					}
					//还原音量
					mediaPlayer.setVolume(1.0f,1.0f);
					break;
				case AudioManager.AUDIOFOCUS_LOSS:
					//长久的失去焦点
					if(mediaPlayer.isPlaying()){
						mediaPlayer.stop();
					}
					mediaPlayer.release();
					mediaPlayer = null;
					break;
				case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT:
					//暂时失去焦点
					if(mediaPlayer.isPlaying()){
						mediaPlayer.pause();
					}
					break;
				case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK:
					//失去音频焦点
					if(mediaPlayer.isPlaying()){
						mediaPlayer.setVolume(0.1f,0.1f);
					}
					break;
			}
		}
},AudioManager.STREAM_MUSIC,AudioManager.AUDIOFOCUS_GAIN);
```

1、初始化MediaPlayer

```
static MediaPlayer create(Context context,int resid) 通过音频资源的id来创建一个MediaPlayer实例

static MediaPlayer create(Context context,Uri uri)  通过一个音频资源的Uri地址来创建一个MediaPlayer实例
```
MediaPlayer还可以通过setDataSource()方法为初始化后的MediaPlayer设置媒体资源

```
void setDataSource(String path); 通过一个媒体资源的地址指定Mediaplayer的数据源，这里的path可以是一个背地路径，也可以是网络路径

void setDataSource(Context context,Uri uri);  通过Uri指定数据源，这里的url可以是网络路径或者是一个内容提供者

void setDataSource(FileDescriptor fd):  通过一个FileDescriptor指定一个MediaPlayer的数据源
```

MediaPlayer可以通过以下的方法直接操作流媒体

```
void start() 开始或恢复播放
void stop()  停止播放
void pause()  暂停播放

int getDuration() 获取流媒体的总播放时长，单位是毫秒
int getCurrentPosition() 获取当前流媒体的播放的位置，单位是毫秒
void seekTo(int mesc)  设置当前MediaPlayer的播放位置，单位是毫秒
void setLooping(boolean looping)  设置是否循环播放
boolean isLooping()  判断是否循环播放
boolean isPlaying()  判断是否正在播放
void prepare()  同步的方式装载流媒体文件
void prepareAsync()  异步的方式装在流媒体文件
void release()  回收流媒体资源
void setAudioStreamType(int streamType)  设置流媒体类型
void setWakeMode(Context context,int mode)  设置cpu唤醒的状态
setNextMediaPlayer(MediaPlayer next)  设置当前流媒体完毕，写一个播放的MediaPlayer
```

MediaPlayer需要先使用prepare()和prepareAsync()方法把流媒体装载MediaPlayer，才可以调用start方法进行播放

MediaPlayer提供的一些事件的回调函数

```
setOnCompletionListener(MediaPlayer.OnCompletion listener)  当流媒体播放完毕的时候回调

setOnErrorListener(MediaPlayer.OnErrorListener listener)  当播放中发生错误的时候回调

setOnPreparedListener(MediaPlayer.OnPreparedListener listener)  当装载流媒体完毕的时候调用

setOnSeekCompleteListener(MediaPlayer.OnSeekCompleteListener listener)  当使用seekTo()设置播放位置的时候回调
```

###简单实例

播放

```
private MediaPlayer mediaPlayer;

private void play(){
	File file = new File(path);
	if(file.exists() && file.length() >0){
		try{
			mediaPlayer = new MediaPlayer();
			mediaPlayer.setDataSource(file);
			mediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
			mediaPlayer.prepareAsync();
			mediaPlayer.setOnPreparedListener(new OnPreparedListner(){
				@Override
				public void onPrepared(MediaPlayer mp){
					//准备完毕回调
					mediaPlayer.start();
				}
			});

			mediaPlayer.setOnCompletionListener(new onCompletionListener(){
				@Override
				public boolean onCompletion(MediaPlayer mp){
					//播放完毕之后回调
				}
			});

			mediaPlayer.setOnErrorListener(new OnErrorListener(){
				@Override
				public boolean onError(MediaPlayer mp,int what,int extra){
					//发生错误，重新播放
					replay();
					return false;
				}
			});
		}catch(Exception e){
			e.printStachTrace();
		}
	}else{
		//文件不存在
	}
}
```

暂停

```
private void pause(){
	if(btn.getText().equals("继续")){
		btn.setText("暂停");
		mediaPlayer.start();
		return;
	}
	if(mediaPlayer != null && mediaPlayer.isPlaying()){
		mediaPlayer.pause();
		btn.setText("继续");
	}
}
```

重新播放

```
private void replay(){
	if(mediaPlayer != null && mediaPlayer.isPlaying()){
		mediaPlayer.seekTo(0);
		btn.setText("暂停");
		return;
	}
	play();
}
```

停止播放

```
private void stop(){
	if(mediaPlayer != null && mediaPlayer.isPlaying()){
		mediaPlayer.stop();
		mediaPlayer.release();
		mediaPlayer = null;
		btn.setEnabled(true);
	}
}
```

```
@Override
private void onDestroy(){
	if(mediaPlayer != null && mediaPlayer.isPlaying()){
		mediaPlayer.stop();
		mediaPlayer.release();
		mediaPlayer = null;
	}
	super.onDestroy();
}
```

###SurfaceView

SurfaceView是配合MediaPlayer使用的，用来显示播放过程中的每一帧画面

```
//只需要为MediaPlayer设置如下方法
void setDisplay(SurfaceHolder holder);

//获取SurfaceHolder
SurfaceHolder.getHolder()

//维护SurfaceHolder
SurfaceHolder.Callback();

void surfaceDestroyed(SurfaceHolder holder):当SurfaceHolder被销毁的时候回调
void surfaceCreated(SurfaceHolder holder):当SurfaceHolder被创建的时候回调
void surfaceChanged(SurfaceHolder holder):当SurfaceHolder的尺寸发生变化的时候回调
```
SurfaceView的兼容性

```
surfaceHolder.getHolder.addCallback(callback);

surfaceHolder.getHolder.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);
```
播放视频事例

```
SurfaceView sv = (SurfaceView)findViewById(R.id.surfaceview);
SeekBar seekbar = (SeekBar)findViewById(R.id.seekbar);
sv.getHolder.addCallback(new MyCallback());
seekbar.setOnSeekBarChangedListener(listener);

private MyCallback callback = new Callback(){
	@Override
	public void surfaceDestroyed(SurfaceHolder holder){
		if(mediaPlayer != null &&mediaPlayer.isPlaying){
			currentPosition = mediaPlayer.getCurrentPosition();
			mediaPlayer.stop();
		}
	}

	@Override
	public void surfacedCreated(SurfaceHolder holder){
		if(currentPosition > 0){
			play(currentPosition);
			currentPosition = 0;
		}
	}

	@Override
	public void surfacedChanged(SurfaceHolder holder){
	}
}

private OnSeekBarChangedListener listener = new OnSeekBarChangeLiatener(){
	@Override
	public void onStopTrackingTouch(SeekBar seekbar){
		//当进度条停止修改的时候触发
		int progress = seekbar.getProgress;
		if(mediaPlayer != null && mediaPlayer.isPlaying()){
			mediaPlayer.seekTo(progress);
		}
	}

	@Override
	public void onStartTrackingTouch(SeekBar seekbar){
	}

	@Override
	public void onProgressChanged(SeekBar seekbar,int progress,boolean fromUser){
	}
}

//停止播放
protected void stop(){
	if(mediaPlayer != null && mediaPlayer.isPlaying()){
		mediaPlayer.stop();
		mediaPlayer.release();
		mediaPlayer = null;
		isPlaying = false;
	}
}

//开始播放
protected void play(final int position){
	File file = new File(path);
	if(!file.exists()){
		return;
	}
	try{
		mediaPlayer = new MediaPlayer();
		mediaPlayer.setAudioStream(AudioManager.STREAM_MUSIC);
		//设置播放的视频源
		mediaPlayer.setDataSource(file.getAbsolutePath());
		//设置显示视频的SurfaceHolder
		mediaPlayer.setDisplay(sv.getHolder());
		mediaPlayer.prepareAsync();
		mediaPlayer.setOnPreparedListener(new OnPreparedListener(){
			@Override
			public void onPrepared(MediaPlayer mp){
				mediaPlayer.start();
				mediaPlser.seekTo(position);
				seekbar.setMax(mediaPlayer.getDuration);
				new Thread(){
					@Override
					public void run(){
						try{
							isPlaying = true;
							while(isPlaying){
								int current = mediaPlayer.getCurrentPosition();
								seekbar.setProgress(current);
								sleep(500);
							}
						}catch(Exception e){
							e.printStacjTrace();
						}
					}
				}.start();
				play_btn.setEnabled(false);
			}
		}
	}
});

mediaPlayer.setOnCompletionListener(new OnCompletionListener(){

	@Override
	public void onCompletion(MediaPlayer mp){
		play_btn.setEnabled(true);
	}
});

mediaPayer.setOnErrorListener(new OnErrorListener(){
	@Override
	public boolean onError(MediaPlayer mp,int what,int extra){
		play(0);
		isPlaying = false;
		return false;
	}
});

//重新开始播放
protected void replay(){
	if(mediaPlayer != null && mediaPlayer.isPlaying()){
		mediaPlayer.seekTo(0);
		play_btn.settext("暂停");
		return;
	}
	isPlaying = false;
	play(0);
}

//暂停或继续
protected void pause(){
	if(play_btn.getText().toString().trim().equals("继续")){
		pause_btn.setText("暂停");
		mediaPlayer.start();
		return;
	}
	if(mediaPlayer != null && mediaPlayer.isPlaying()){
		mediaPlayer.pause();
		btn_pause.setText("继续");
	}
}

```

###VideoView

```
int getCurrentPosition() 获取当前的视频位置
int getDuration() 获取当前视频的总长度
isPlaying() 当前VideoView是否播放视频
void pause() 暂停
void seekTo(int msec) 从第几秒开始播放
void resume() 重新播放
void setVideoPath(String path)
void setVideoURI(Uri uri) 以Uri的方式设置VideoView播放视频源，可以是网络或者本地的Uri
void start() 开始播放
void stopPlayback() 停止播放
setMediaController(MediaController controller) 设置MediaController控制器
setOnCompletionListener(MediaPlayer.OnCompletionListener listener) 监听播放完成事件
setOnErrorListener(MediaPlayer.OnErrirListener listener)  监听播放发生错误的事件
setOnPreparedListener(MediaPlayer.OnPreparedListener listener)  监听视频装载完成的事件；
```

示例

```
seekbar = (SeekBar)findViewById(R.id.seekbar);
seekbar.setOnSeekChangeListener(){
	@Override
	public void onStopTrackingTouch(SeekBar seekbar){
		int progress = seekbar.getProgress();
		if(videview != null && videoview.isPlaying()){
			videoview.seekTo(progress);
		}
	}

	@Override
	public void onStartTrackingTouch(SeekBar seekbar){

	}

	@Override
	public void onProgressChanged(SeekBar seekbar,int progress,boolean fromUser){

	}
}

protected void play(int mesc){
	File file = new File(path);
	if(!file.exists()){
		return;
	}
	videoview.setVideoPath(file.getAbsolutePath());
	videoview.start();
	videoview.seekTo(mesc);
	seekbar.setMax(videoview.getDuration());
	new Thread(){
		@Override
		public void run(){
			try{
				isPlaying = true;
				while(isPlaying){
					int current = videoview.getCurrentPosition();
					seekbar.setProgress(current);
					sleep(500);
				}
			}catch(Exception e){
				e.printStackTrace();
			}
		}
	}.start();
}

videoview.setOnCompletionListener(new OnCompletionListener(){
	@Override
	public void onCompletion(MediaPlayer mp){
		//播放完成后回调
	}
})

videoview.setOnErrorListener(new OnErrorListener(){
	@Override
	public boolean onError(MediaPlayer mp,int what,int extra){
		play(0);
		isPlaying = false;
		return false;
	}
})

protected void replay(){
	if(videoview != null && videoview.isPlaying()){
		videoview.seekTo();
		return;
	}
	isPlaying = false;
	play(0);
}

protected void pause(){
	if(btn_pause.getText().toString().trim().equals("继续")){
		btn_pause.setText("暂停");
		videoview.start();
		return;
	}
	if(videoview != null && videoview.isPlaying()){
		videoview.pause();
	}
}

protected void stop(){
	if(videoview != null && videoview.isPlaying()){
		videoview.stopPlayback();
		isPlaying = false;
	}
}
```

###MediaController

- boolean isShowing() 当前悬浮控制栏是否显示
- void setMediaPlayer(MediaController.MediaPlayerController player) 设置控制的组件
- void setPrevNextListener(View.OnClickListener next,View.OnClickListener prev)  设置上一个视频，下一个视频的切换事件

```
MediaController controller = new MediaController(this);
videoview.setMediaController(controller);
controller.setMediaPlayer(videoview);
controller.setPrevNextListeners(new OnclickListener(){
	@Override
	public void onClick(View v){

	}
},new OnClickListener(){
	@Override
	public void onClick(View v){

	}
})
```
