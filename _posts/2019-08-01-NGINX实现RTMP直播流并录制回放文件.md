---
layout: post
title: NGINX实现RTMP直播流并录制回放文件
data: 2019-08-01
tag: nginx、rtmp
---



## NGINX实现RTMP直播流并录制回放文件

利用nginx添加RTMP进行实时流数据传输，需要为nginx添加rtmp模块，所以需要将nginx添加此模块后进行编译



### 1.重新编译nginx

停止原来的nginx：`nginx -s stop`

进入到一开始编译安装nginx的安装包路径，并将 `nginx-rtmp-module` 下载到nginx编译安装路径同级目录下

![n](/images/posts/rtmp/n.png)

进入到nginx安装路径:` cd nginx-1.12.1`

使用 `./configure --add-module=../nginx-rtmp-module`指令添加rtmp模块到nginx主体文件中

然后`make` 

### 2.替换原有nginx二进制

如果是第一次安装nginx，可以直接使用 `make install` 来进行nginx的安装

Linux上默认会安装到 `/usr/local/nginx` 中

Mac下默认安装到 `/usr/local/opt/nginx-full/bin/nginx`，Mac下nginx配置文件默认所在为 `/usr/local/etc/nginx/nginx.conf`

如果之前已经安装过nginx，则复制nginx替换掉之前的nginx二进制：`cp ./nginx-1.12.1/objs/nginx /usr/local/nginx/sbin/nginx`  （是否需要备份原有的nginx二进制请自行决定）

### 3.配置RTMP

配置RTMP服务：

```json
rtmp {	#RTMP协议，与HTTP协议同一级
  server {
    listen 1935;
    #直播流配置，直播间
    application live {
      live on;
      #为rtmp引擎设置最大连接数，默认off
      max_connections 1024;
      
      #开启录制功能，会将直播的信息保存成一个flv文件
      record all;
      #视频录制存放目录,注意 因为需要生成文件，所以需要nginx以某种可以让其他服务读写文件的用户权限启动
      record_path /Users/ZhXiQi/Desktop/video;
      #每次录制是否唯一文件名，会以 房间号-时间戳 为名称，房间号由推流端指定，跟在 live后面，如 live/room1
      record_unique on;
      #将直播录制的视频转为mp4格式，主要为FFmpeg指令的使用，未验证
      #exec_record_done 为录制完成后执行的指令
      exec_record_done ffmpeg -y -i $path -acodec libmp3lame -ar 44100 -ac 1 -vcodec libx264 $path/$basename.mp4;

      #注意，以下两个设置必须保证配置的url能请求的通，否则会导致推流失败
      #推流开始回调，后面IP为后端项目请求地址
      on_publish http://127.0.0.1:9090/rtmp/hello;
      #推流结束回调，推流结束，通知服务端进行相应的操作
      on_done http://127.0.0.1:9090/rtmp/hello;
    }
		#hls直播协议
    application hls{
      live on;
      hls on;
      hls_path /Users/ZhXiQi/Downloads;
      hls_fragment 1s;
    }
		#回看应用，设置此项后，后续可以通过支持rtmp协议的播放器来播放此路径下的文件
		application vod {
				#回看
        play /home/hyperchain/video;
    }
  }
}
/**
其中：
$name 　　		推流名称				 (room)
$path 　　		记录文件路径			(/Users/ZhXiQi/Desktop/video/room-1389499351.flv)
$filename 　　省略目录的路径　　	(room-1389499351.flv)
$basename 　　扩展名省略的文件名	(room-1389499351)
$dirname 　　	目录路径　　 			(/Users/ZhXiQi/Desktop/video)
更具体参数可查询官方文档
*/
```

### 4.检查并启动nginx

使用 `/usr/local/nginx/sbin/nginx` 里的二进制文件，对 `/usr/local/nginx/conf/nginx.conf` 配置文件做检查

`/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf`

启动

`nginx` 

### 5.测试

可以使用 ffmpeg来进行推流测试

`ffmpeg -re -i /Users/ZhXiQi/Desktop/1564711041674615.mp4 -vcodec libx264 -acodec aac -strict -2 -f flv rtmp://localhost:1935/rtmplive/room?username=ZhXiQi&password=123`

如果录制文件夹下生成了刚才推送的内容，则表明此次安装成功，其中`room`为房间号，可以任意指定，后面跟的参数可以在 `on_publish` 或者 `on_done` 所调用的后台服务中通过 `request.getParameter("username")` 这种方式获取数据，以达到鉴权的效果

### 6.待完善

使用过程中，发现iOS手机端如果使用的是横屏录屏的话，nginx直播录制的文件分辨率依旧是竖屏，导致界面压缩，尚未明确分辨率设置是否由此nginx方设置还是由客户端推流的时候进行设置

