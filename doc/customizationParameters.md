# 自定义参数

### 简介
通过修改release.json文件来增加,删除或修改appfi项目中docker市场的软件列表。

### 格式
```
[
  {
    "appname": "ownCloud", //自定义的docker镜像名称
    "flavor": "vanilla", //现阶段为固定参数,无需修改
    "components": [
      {
        "name": "owncloud", //dockerhub网站中该镜像的名称
        "namespace": "library", //dockerhub网站中该镜像的命名空间名称
        "imageLink": "owncloud.png",
        "tag": "latest", //根据dockerhub网站,指定该镜像采用的标签号
        "repo": null, //现阶段为固定参数,无需修改
        "overlay": true, //现阶段为固定参数,无需修改
        "config": {
          "HostConfig": {
            "Binds": [
              "/var/www/html:/var/www/html" //将docker镜像运行时的容器内部目录和外部host环境目录进行绑定
            ],
            "RestartPolicy": {
              "Name": "unless-stopped" //指定docker镜像的重启策略
            },
            "PortBindings": {
              "80/tcp": [
                {
                  "HostPort": "10086"  //将docker镜像运行时的容器内部端口和外部host环境端口进行绑定
                }
              ]
            },
            "PublishAllPorts": false //现阶段为固定参数,无需修改
          }
        },
        "volumes": [] //
      }
    ]
  },
  //重复上述配置
  ...
  ...
  ...
]
```

### 参数详细说明
+ Binds
  - 容器内部目录和外部host环境目录都必须写成绝对路径
  - 可以增加ro参数使其只读,即"/var/www/html:/var/www/html:ro"
