# android-blocked-upload

安卓文件上传工具，公司内部需要使用文件上传，在github上搜索了一圈没有合适的，因此自己设计开发了此代码，清洗掉公司相关的信息后放到这里


###实现原理：
------------------------------
使用java 的 RandomFileAccess 文件类， 每次上传时读取一个固定大小的数据块，然后使用Http表单提交，上传到服务器，上传时带上当前区块参数，以及文件总大小参数，
服务器也使用RandomFileAccess 文件类 初始化一个指定大小的文件，在指定位置写入此二进制区块数据，即完成大文件上传任务了

此app依赖了 RxJava 、FastJson、OkHttp 三个外部工具库
	RxJava 启动一个线程作为定时器，每秒（使用时可以自定义频率）执行一遍请求，请求带上当前文件区块，如果当前Http请求未完成状态，则跳过此次执行，等待下次执行
	使用这种机制时为了避免上传因为一次失败 断掉，也方便实现暂停上传和恢复上传，缺点就是 速度慢一点（取决于调度频率以及定义的区块大小，每秒一次定时，网络速度比较快的情况下有空闲时间导致）
	
###使用
------------------------------
Android  manifest.xml中添加service, service中使用Uploader类，操作上传功能，以及管理调度Uploader,
当前上传进度使用Notification在通知栏中显示，APP即使退出前台通知栏一直显示当前进度
Uploader类中有addTask方法 可以在Activity中绑定service后直接调用service的addTask方法添加上传任务，
也可以使用Hbuilder 的h5+ 的html 页面中 使用 startService 方法 传参数 添加上传任务

将Net.java 里面的 UPLOAD_URL 地址换成自己服务端的地址，服务端地址代码参考server目录下的类文件以及说明

###扩展使用
------------------------------
此代码的实现原理即可以用来上传，稍作改造后也可以变成断点下载，改造代码很小

###优点
------------------------------
a.支持暂停、恢复，网络断开自动暂停自动恢复上传，可靠性不错，逻辑简单，服务端实现简单，附有服务端实现代码
b.支持多文件上传，多文件路径存储在List中，在上传过程中也可以动态继续添加待上传文件，上传完自动从List中移除文件。
c. 支持显示总体上传进度，稍作改造亦可以支持每个文件上传进度，以及单个文件的暂停恢复
d. 支持 集成到 hbuilder h5+ app 使用
e. 支持上传成功后是否自动删除文件，在拍照上传/录音上传时有可能不必保留源文件，录音、拍照添加到任务列表后就可以将autoDelete=true作为参数，上传完直接删除
f. 支持附带参数，每个文件添加到任务队列时，可以附带额外业务参数，以提供给服务器做业务处理

###缺点
------------------------------
在开发此工具初期，是使用Http请求成功后立即执行下一个文件块上传的方式，测试时速度很好，网络利用率满载，用户如果需要的化可以改造成这种用法。不过如果出现网络问题时不会接续上传，暂停恢复需要做一些改造。公司内对上传速度没有要求，所以没有使用此方式实现，而是使用定时器调度上传
因为项目不大，代码不多，没有打包成aar 放到jitpack上，不方便引用，如果后续项目变大变复杂，会考虑放到jitpack上

##联系我
------------------------------
如有疑问，或者需要帮助支持，请在issue下留言


#示例截图：
------------------------------
###断开网络情况
------------------------------
![无网络链接情况上传状态图片](/imgs/device-2019-05-05-135652.png)

###网络链接情况
------------------------------
![联网情况上传状态图片](/imgs/device-2019-05-05-135710.png)

###服务器收到的文件
------------------------------
![服务器收到的文件](/imgs/server-recived-files.png)