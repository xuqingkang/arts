In this ARTS, I introduces the XXX's configration and program sample.

# 1.Algorithm
### Question: 



### My solution in python3 is:


# 2.Review
// TODO: I will complete it later.

# 3.Tips(关于Windows下启动springboot的脚本并发调用的问题)
我在主目录microServiceManager下有5个微服务，
比如：eureka-server\target\eureka-server-0.0.1-SNAPSHOT.jar，现在主目录下有5个对应的启动脚本。

启动脚本如下以eureka-server为例，其他四个微服务的启动脚本跟这个类似，仅仅是微服务名称不一样而已。
```
@echo off

java -jar eureka-server\target\eureka-server-0.0.1-SNAPSHOT.jar
goto end

:end
```

我希望一次性调用所有微服务的启动，因此写了下面的脚本，用call命令分别调用这些脚本。
```
@echo off
setlocal
set BIN_HOME=%~dp0
call "%BIN_HOME%start-server.bat"
call "%BIN_HOME%start-hi.bat"
call "%BIN_HOME%start-ribbon.bat"
call "%BIN_HOME%start-feign.bat"
call "%BIN_HOME%start-manager.bat"

endlocal
```

但当运行以上脚本时发现，整个cmd命令行在启动start-server.bat的执行后屏幕输出第一个微服务的日志，这个服务正常启动，但是相当于cmd控制台不继续调用别的bat脚本了。

后来发现这种需要调用多个bat的，可以使用[START "xxx" 脚本路径]的方式进行启动，达到的效果就是windows会单独弹框出来进行启动。
就像启动tomcat是startup.bat会弹出一个新的窗口来运行是一样的效果。
```
@echo off
setlocal
set BIN_HOME=%~dp0
START "springboot-server"  "%BIN_HOME%start-server.bat"
START "springboot-hi" "%BIN_HOME%start-hi.bat"
START "springboot-ribbon" "%BIN_HOME%start-ribbon.bat"
START "springboot-feign" "%BIN_HOME%start-feign.bat"
START "springboot-manager" "%BIN_HOME%start-manager.bat"

endlocal
```

# 4.Share



