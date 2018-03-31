# FFmpegForAndroid
直奔主题：您的应用程序可以运行FFmpeg命令的一种方式，只有Java，不需要C代码或NDK。

###使用方式
1. 导入lib
2. 添加权限	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/><uses-permission android:name="android.permission.WAKE_LOCK" />
3. 在gradle中添加
<code>defaultConfig {
   applicationId "com.examples.ffmpeg4android_demo"
   minSdkVersion 16
   targetSdkVersion 23
   ndk {
       abiFilter "armeabi-v7a"
   }
</code>
4. 代码中调用<code>LoadJNI vk = new LoadJNI();
       		 try {
            		String workFolder = getApplicationContext().getFilesDir().getAbsolutePath();
            		String[] complexCommand = {"ffmpeg","-i", "/sdcard/videokit/in.mp4"};
            		vk.run(complexCommand , workFolder , getApplicationContext());
            		Log.i("test", "ffmpeg4android finished successfully");
        		} catch (Throwable e) {
            		Log.e("test", "vk run exception.", e);
        		}</code>


### 命令示例
#### 视频压缩：
##### //简单commad 
	ffmpeg -y -i /sdcard/videokit/in.mp4 -strict experimental -s 160x120 -r 25 -vcodec mpeg4 -b 150k -ab 48000 -ac 2 -ar 22050 /sdcard/videokit/out.mp4
 
##### //用h264压缩（支持铬）
//请注意，您需要使用EXTRAS（见下文）分发才能使用此命令。
	ffmpeg -y -i /sdcard/videokit/in.mp4 -strict experimental -vcodec libx264 -preset ultrafast -crf 24 -acodec aac -ar 44100 -ac 2 -b:a 96k -s 320x240 -aspect 4:3 /sdcard/videokit/out3.mp4

##### //作为复杂的命令，不要忘记使用setCommandComplex（complexCommand）
//使用这种格式来支持包含空格和特殊字符的文件
	String[] complexCommand = {"ffmpeg","-y" ,"-i", "/sdcard/video kit/in.mp4","-strict","experimental","-s", "160x120","-r","25", "-vcodec", "mpeg4", "-b", "150k", "-ab","48000", "-ac", "2", "-ar", "22050", "/sdcard/video kit/out.mp4"};

##### //运行压缩命令，同时保存元数据信息（包括旋转元数据）：
	fmpeg -y -i /sdcard/videokit/in.mp4 -strict experimental -map_metadata 0:g -s 160x120 -r 25 -vcodec mpeg4 -b 150k -ab 48000 -ac 2 -ar 22050 /sdcard/videokit/out1.mp4


控制质量的参数是-s（分辨率，当前设置为160x120）和-b（比特率，目前设置在150k）。
增加它们，例如-s 480x320 
和-b 900k 
提高质量（并减少压缩）。﻿


#### 音频压缩
 
	String commandStr = "ffmpeg -y -i /sdcard/vk2/in.wav -ar 44100 -ac 2 -ab 64k -f mp3 /sdcard/videokit/out.mp3";


#### 音频修剪（裁剪）
	String commandStr ={"ffmpeg","-y","-i","/storage/emulated/0/vk2/in.mp3","-strict","experimental","-acodec","copy","-ss","00:00:00","-t","00:00:03.000","/storage/emulated/0/videokit/out.mp3"};

 
#### 视频旋转（90度）：
	ffmpeg -y -i /sdcard/videokit/in.mp4 -strict experimental -vf transpose=1 -s 160x120 -r 30 -aspect 4:3 -ab 48000 -ac 2 -ar 22050 -b 2097k /sdcard/video_output/out.mp4

 
#### 视频裁剪：
	ffmpeg -y -i /sdcard/videokit/short.mp4 -strict experimental -vf crop=100:100:0:0 -s 320x240 -r 15 -aspect 3:4 -ab 12288 -vcodec mpeg4 -b 2097152 -sample_fmt s16 /sdcard/videokit/out.mp4
 
#### 从视频中抽取图片：
	ffmpeg -y -i /sdcard/videokit/in.mp4 -strict experimental -an -r 1/2 -ss 00:00:00.000 -t 00:00:03 /sdcard/videokit/filename%03d.jpg
 
#### 从视频中抽取声音：
ffmpeg -y -i /sdcard/videokit/in.avi -strict experimental -acodec copy /sdcard/videokit/out.mp3

ffmpeg -y -i /sdcard/videokit/in.mp4 -strict experimental -vn -ar 44100 -ac 2 -ab 256k -f mp3 /sdcard/videokit/out.mp3

 
#### 在视频中重新编码音频：
	ffmpeg -y -i /sdcard/in.mp4 -strict experimental -vcodec copy -acodec libmp3lame -ab 64k -ac 2 -b 1200000 -ar 22050 /sdcard/out.mp4

#### 更改视频分辨率：
	ffmpeg -y -i /sdcard/in.mp4 -strict experimental -vf transpose=3 -s 320x240 -r 15 -aspect 3:4 -ab 12288 -vcodec mpeg4 -b 2097152 -sample_fmt s16 /sdcard/out.mp4

#### 从视频剪辑时间片段：
	ffmpeg -ss 00:00:01.000 -y -i /sdcard/videokit/in.mp4 -strict experimental -t 00:00:02.000 -s 320x240 -r 15 -vcodec mpeg4 -b 2097152 -ab 48000 -ac 2 -b 2097152 -ar 22050 /sdcard/videokit/out.mp4

#### 转码音频：
	ffmpeg -y -i /sdcard/videokit/big.wav /sdcard/videokit/small.mp3


#### 水印：
 // test with watermark.png 128x128，将其添加到/ sdcard / videokit /
 
 
	String[] complexCommand = {"ffmpeg","-y" ,"-i", "/sdcard/videokit/in.mp4","-strict","experimental", "-vf", "movie=/sdcard/videokit/watermark.png [watermark]; [in][watermark] overlay=main_w-overlay_w-10:10 [out]","-s", "320x240","-r", "30", "-b", "15496k", "-vcodec", "mpeg4","-ab", "48000", "-ac", "2", "-ar", "22050", "/sdcard/videokit/out.mp4"}; 

#### 流式传输简单：
从Android设备流到PC
 
*请注意，所有流媒体示例都需要将Internet权限添加到Android项目清单文件（<uses-permission android：name =“android.permission.INTERNET”/>）

 
// 在ffmpeg4android上使用此命令（  192.168.1.11是PC IP）
ffmpeg -i /sdcard/videokit/2.mpg -strict experimental -f mpegts udp：//192.168.1.11：8090

//你可以使用任何支持流媒体的任何球员，在目标机器上，打  流，在这种情况下，我们使用ffplay //  （ 192.168.1.14是Android设备IP）



	ffplay -f mpegts -ast 1 -vst 0 -ar 48000 udp：//192.168.1.14：8090 







在一台设备上流式传输，在第二台设备上接收流并保存：
在第一台设备上：

	ffmpeg -i /sdcard/one3.mp4 -f mpegts udp：//192.168.0.107：8090


在第二个设备上：

	String [] complexCommand = {“ffmpeg”，“ - y”，“ - i”，“udp：//192.168.0.108：8090”，“-strict”，“experimental”，“-crf”，“30”， “-preset”，“ultrafast”，“-acodec”，“aac”，“-ar”，“44100”，“-ac”，“2”，“-b：a”，“96k”，“ “，”libx264“，”-r“，”25“，”-b：v“，”500k“，”-f“，”flv“，”/sdcard/videokit/t.flv“};

*这需要清单中的网络权限。 

#### H264编码（需要额外）：
ffmpeg -y -i /sdcard/Video/1.MTS -strict experimental -vcodec libx264 -preset ultrafast -crf 24 /sdcard/videokit/out.mp4

ffmpeg -y -i /sdcard/videokit/m.mkv  -strict experimental  -vcodec libx264 -preset ultrafast -crf 24 -sn /sdcard/videokit/m2.mkv

#### 字幕：
ffmpeg -y -i /sdcard/videokit/m2.mkv -i /sdcard/videokit/in.srt  -strict experimental  -vcodec libx264 -preset ultrafast -crf 24 -scodec copy /sdcard/videokit/mo.mkv

ffmpeg -y -i /sdcard/videokit/m2.mkv -i /sdcard/videokit/in.srt  -strict experimental  -scodec copy /sdcard/videokit/outm3.mkv

#### 将音频文件转换为m4a 
	ffmpeg -i /sdcard/videokit/in.mp3 /sdcard/videokit/out.m4a

#### 在一个命令中编码h264视频和aac音频（需要额外）

	ffmpeg -y -i /sdcard/videokit/in.mp4 -strict experimental -vcodec libx264 -crf 24 -acodec aac /sdcard/videokit/out.mkv





#### 老式过滤器

	commandStr = "ffmpeg -y -i /sdcard/videokit/in.mp4 -strict experimental -vf curves=vintage -s 640x480 -r 30 -aspect 4:3 -ab 48000 -ac 2 -ar 22050 -b 2097k -vcodec mpeg4 /sdcard/videokit/curve.mp4";

Black & White filter (Gray Scale):

	commandStr = "ffmpeg -y -i /sdcard/videokit/in.mp4 -strict experimental -vf hue=s=0 -vcodec mpeg4 -b 2097152 -s 320x240 -r 30 /sdcard/videokit/out.mp4";

Sepia using colorchannelmixer


	String[] complexCommand = {"ffmpeg","-y" ,"-i", "/sdcard/videokit/sample.mp4","-strict", "experimental", "-filter_complex","[0:v]colorchannelmixer=.393:.769:.189:0:.349:.686:.168:0:.272:.534:.131[colorchannelmixed];[colorchannelmixed]eq=1.0:0:1.3:2.4:1.0:1.0:1.0:1.0[color_effect]","-map", "[color_effect]","-map", "0:a", "-vcodec", "mpeg4","-b", "15496k", "-ab", "48000", "-ac", "2", "-ar", "22050","/sdcard/videokit/out.mp4"};













#### 淡入和淡出过渡

	String [] complexCommand = {“ffmpeg”，“ - y”，“ - i”，“/sdcard/videokit/in.m4v","-acodec”，“copy”，“-vf”，“fade = t = in：st = 0：d = 5，fade = t = out：st = 20：d = 5“，”/sdcard/videokit/out.mp4“};


#### 使用filter_complex加入2个使用相同大小的文件

	String [] complexCommand = {“ffmpeg”，“ - y”，“ - i”，“/sdcard/videokit/in1.mp4”，“-i”，“/sdcard/videokit/in2.mp4”，“ - strict “，”experimental“，”-filter_complex“，”[0：0] [0：1] [1：0] [1：1] concat = n = 2：v = 1：a = 1“，”/ sdcard /videokit/out.mp4" }; 

//不同编解码器的连续视频，以及不同的大小，不同的速率和不同的宽高比：


	String[] complexCommand = {"ffmpeg","-y","-i","/storage/emulated/0/videokit/sample.mp4",
    "-i","/storage/emulated/0/videokit/in.mp4","-strict","experimental",
    "-filter_complex",
    "[0:v]scale=640x480,setsar=1:1[v0];[1:v]scale=640x480,setsar=1:1[v1];[v0][0:a][v1][1:a] concat=n=2:v=1:a=1",
    "-ab","48000","-ac","2","-ar","22050","-s","640x480","-r","30","-vcodec","mpeg4","-b","2097k","/storage/emulated/0/vk2_out/out.mp4"}





#### 从图片创建视频

//请注意，此命令对图片大小很敏感。
//所有图片应该具有相同的尺寸，并且对应于特定的视频分辨率。
//例如高清视频是：1280x720，这应该是图片大小 


	commandStr =“ffmpeg -y -r 1/5 -i /sdcard/videokit/pic00%d.jpg /sdcard/videokit/out.mp4”; 

##### //与音频。（对所有玩家都适用）png也可以，这种情况下的图片大小应该是320x240 

	ffmpeg -y -r 1 -i /sdcard/videokit/pic00%d.jpg -i /sdcard/videokit/in.mp3 -strict experimental -ar 44100 -ac 2 -ab 256k -b 2097152 -ar 22050 -vcodec mpeg4 -b 2097152 -s 320x240 /sdcard/videokit/out.mp4





#### 高级过滤

	String [] complexCommand = {“ffmpeg”，“ - y”，“ - i”，“/sdcard/videokit/in.mp4","-strict","experimental”，“-vf”，“crop = iw / 2：ih：0：0，split [tmp]，pad = 2 * iw [left]; [tmp] hflip [right]; [left] [right] overlay = W / 2“，”-vb“，”20M “，”-r“，”23.956“，”/sdcard/videokit/out.mp4“};

#### 提高视频和音频速度

	String [] complexCommand = {“ffmpeg”，“ - y”，“ - i”，“/sdcard/videokit/in.mp4","-strict","experimental”，“-filter_complex”，“[0：v ] setpts = 0.5 * PTS [v]; [0：a] atempo = 2.0 [a]“，” - map“，”[v]“，” - map“，”[a]“，” - b“， “2097k”，“ - r”，“60”，“-vcodec”，“mpeg4”，“/sdcard/videokit/out.mp4”}; 

#### 叠加2个视频并排      

	String[] complexCommand = {"ffmpeg","-y" ,"-i", "/sdcard/Movies/sample.mp4","-i", "/sdcard/Movies/sample2.mp4", "-strict","experimental",
                "-filter_complex", 
                "[0:v:0]pad=iw*2:ih[bg];" + 
                "[bg][1:v:0]overlay=w",
                "-s", "320x240","-r", "30", "-b", "15496k", "-vcodec", "mpeg4","-ab", "48000", "-ac", "2", "-ar", "22050",
                "/sdcard/videokit/out.mp4"};   







#### vintage + watermark

 
     String[] complexCommand = {"ffmpeg","-y" ,"-i", "/sdcard/Movies/sample.mp4","-strict","experimental",
                "-vf", 
                "movie=/sdcard/videokit/watermark002.png [watermark];" + 
                "[in][watermark] overlay=main_w-overlay_w-10:10 [out_overlay];" +
                "[out_overlay]curves=vintage[out]",  
                "-s", "320x240","-r", "30", "-b", "15496k", "-vcodec", "mpeg4","-ab", "48000", "-ac", "2", "-ar", "22050",
                "/sdcard/videokit/out_water_vinta.mp4"};
