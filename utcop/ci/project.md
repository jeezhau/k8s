**项目基础**

# 基本理念
- 应用代码与应用配置分离；
- 采用 CNCF 的项目构建理念，服务自治且轻量；

# 项目结构
在项目中应该在顶层目录包含 ci 目录，其中放置 Docker 构建信息 与 k8s 定义文件模版，以springboot项目为例：
```
├── src/main/java
│   └── com.utcop     # 项目源码
├── src/main/resources
│   └── applications.yml      #通用配置
│   └── applications-env.yml.j2  #环境自定义配置，作为其他环境的模板
│   └── applications-dev.yml     #开发环境自定义配置
├── ci
│   └── dockerfiles
│       └── Dockerfile	  # Dockerfile 文件
│       └── build.sh		  # docker image build 与 push 脚本 
│   └── k8s
│       └── app.yaml.j2		# k8s deployment、service 部署模板文件 
│       └── build.sh		  # k8s deployment、service 文件生成脚本 
│   └── config
│       └── build.sh		  # 具体环境文件生成脚本 ，可选
│   └── globalvalues    # 全局配置信息，目前采用 shell 变量；包含当前环境，docker registry 等配置
│   └── values          # 项目信息，目前采用 shell 变量，包含项目名称、版本、服务名称等
└── └── build.sh		    # 构建脚本，调用 dockerfiles 与 k8s 中的脚本
```
- ci/dockerfiles  归纳 docker 相关文件，包括 Dockerfile 、构建脚本、以及 Dockerfile 使用到的文件；
- ci/k8s          放置kubernetes服务定义文件模版、生成脚本等；
- ci/config       处理具体部署环境的配置文件的脚本可选于该目录下；
- ci/globalvalues 全局配置信息，构建脚本使用；
- ci/values       当前项目配置信息，构建脚本使用；
- ci/build.sh     项目构建脚本，调用子目录构建脚本；

# 无状态服务
项目所构建的服务应该是无状态的，可自我恢复的；不使用服务内部的 session 之类的状态管理，如果需请使用 redis 作为分布式 session 管理；
