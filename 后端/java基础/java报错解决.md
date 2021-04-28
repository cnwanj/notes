# 1.idea重启导致端口被占用The Tomcat connector configured to listen on port 8080 failed to start.

```
# 查看被占用端口
netstat -ano | findstr 8080

看到端口的LISTENING 13964
```

> 解决：打开任务管理器，在详细信息中，找到java.exe对应的LISTENING，然后结束任务，这样就可以重启服务了。

