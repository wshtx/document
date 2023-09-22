# Docker常用命令

![image-20221215174722890](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/image-20221215174722890.png)

![image-20221215174347795](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/image-20221215174347795.png)

- 查看当前正在运行的 Docker 容器的进程号（PID）：

```
$ docker inspect --format '{{ .State.Pid }}'  4ddf4638572d
25686
```

