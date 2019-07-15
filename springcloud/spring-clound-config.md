
# 配置
```yaml
server:
  port: 7010
  
spring:
  cloud: 
    config: 
      server: 
        default-application-name: configServer
        git:
          search-paths: '{application}'
          uri: git@gitlab.com:utcop/utcopconfrepo.git
          timeout: 10000
         # user: jeezhau
         # ignore-local-ssh-settings: true
          private-key-file: /Users/jeekhan/.ssh/id_rsa
          strictHostKeyChecking: false
         # username: jeezhau
         # password: "abc,./139"
          force-pull: true
          basedir: /Users/jeekhan/workrepo/configrepo

```
## SSH 密钥问题
JGIT 不支持 `BEGIN OPENSSH PRIVATE KEY` 格式的 privateKey，可使用如下方式生成：
`ssh-keygen -m PEM -t rsa -b 4096 -C "email"`

## 连接超时问题
git连接默认5秒超时失败
