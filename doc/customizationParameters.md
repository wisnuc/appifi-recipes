# 自定义参数

### 简介

通过修改release.json文件来增加,删除或修改appfi项目中docker市场的软件列表,该软件列表来源于[*Docker Hub*](https://hub.docker.com/)

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
        "imageLink": "null", //现阶段用户手工指定图片,为固定参数,无需修改
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
            "PublishAllPorts": false //如果为false,则上面的`PortBindings`参数生效,为true时,使用dockerhub网站中的默认值,无需`PortBindings`参数
          }
        },
        "volumes": [] //现阶段为固定参数,无需修改
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

+ HostConfig中的PortBindings参数

  值为true或false,当为true时,说明采用dockerhub项目中的默认配置,用户无需配置该参数;当为false时,说明采用用户自定义配置,dockerhub项目中的默认配置失效
  
  - 当`PublishAllPorts`为false时<p>
  
    对于tcp和udp,采用/udp和/tcp的方式进行区分,即`22/udp`或者`80/tcp`
  
  - 当`PublishAllPorts`为true时<p>
  
    无需配置该参数
  
  
+ HostConfig中的RestartPolicy参数的name子参数

  当容器退出时执行该参数,参数共计3种
  
  - always<p>
  
    一直重启
  
  - unless-stopped<p>
  
    只有当用户手工停止该容器时才会停止,否则一直重启
  
  - on-failure<p>
  
    当容器退出时的返回值不为0才会重启
  

+ HostConfig中的Binds参数

  - 格式为`host-src:container-dest`,前者为运行该docker的host设备的路径(可以根据需要任意修改),后者为docker内部路径(从dockerhub项目中获取到的固定值,不能修改)<p>

  - 容器内部目录和外部host环境目录都必须写成绝对路径<p>
  
  - 可以增加ro参数使挂载目录只读,即`/var/www/html:/var/www/html:ro`<p>
  
  - 为了挂载路径的安全性,appifi对host环境目录的路径头部增添额外的路径字段,以transmission为例,具体路径为`/run/wisnuc/volumes/随机uuid/wisnuc/appdata/dockerhub/dperson/transmission/latest/vanilla`,因此,host环境下的挂载点不会从根目录开始,请用户注意<p>
  

### 操作事例1 - elasticsearch

+ 选择docker镜像

  - 登录dockerhub网站[*官网链接*](https://hub.docker.com/)<p>
  
    通过搜索或`Explore`选择需要的镜像,这里使用`elasticsearch`镜像[*链接*](https://hub.docker.com/_/elasticsearch/)
    
  - 获取必要的参数信息<p>
  
    即上面提到的`components`参数里的`name`,`namespace`和`tag`子参数,`HostConfig`参数里的`Binds`和`PortBindings`子参数,该项目中分别为:
    
    ```
      name: elasticsearch
      namespace: library
      tag: latest

      Binds: null
      PortBindings: null
    ```
    
    ![1](https://github.com/wisnuc/appifi-recipes/blob/release/doc/pic/elasticsearch.jpg)<p>
    
    备注:
    
      因为`PortBindings`为空,所以`PublishAllPorts`置为false,上述信息的获取需要用户了解Dockerfile的相关知识,具体信息请查阅[官方链接](https://docs.docker.com/engine/reference/builder/)
    
  - 创建release.js<p>
  
    创建或编辑原有release.js文件,添加上面步骤得到的参数,即
    
    ```
      module.exports = [
       {
         appname: 'elasticsearch',
         flavor: 'vanilla',
         components: [
           {
             name: 'elasticsearch',
             namespace: 'library',
             imageLink: 'elasticsearch.png',
             tag: 'latest',
             repo: null,
             overlay: true,
             config: {
               HostConfig: {
                 RestartPolicy: {
                   Name: 'unless-stopped'
                 },
                 PublishAllPorts: true
               }
             },
             volumes: []
           }
         ]
       },
      ]
    ```
    
  - 生成release.json<p>

    在终端中进入该项目根目录,执行命令`./gen release.js`,即可生成所需要的release.json文件
    
    备注:

    该步骤会覆盖原有文件,请用户做好备份
   
  - 将该文件通过网页上传至设备中,点击`刷新`按钮,新的配置即可生效<p>
