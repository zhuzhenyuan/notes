1、docker服务自动重启设置
```
systemctl enable docker.service
```

2、docker容器自动启动设置
```
docker restart imageid
```

在运行docker容器时可以加如下参数来保证每次docker服务重启后容器也自动重启：
```
docker run --restart=always
```
如果已经启动了则可以使用如下命令：
```
docker update --restart=always <CONTAINER ID>
```

重启系统后
```
docker  ps -a 
```
