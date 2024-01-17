Android drone CI/CD  最佳实践

```yaml
kind: pipeline
type: docker
name: android-drone
#设置平台
platform:
  os: linux
  arch: amd64
#开始
steps:
  - name: build       #步骤名称
    image: gradle:8.0 #使用Gradle镜像并指定版本
    volumes:                          #挂载卷
      - name: sdk                     # SDK
        path: /drone/src/sdk          # SDK 路径
      - name: gradle-cache            # gradle缓存
        path: /home/gradle/.gradle    # “gradle镜像”的缓存路径
    environment:                      #环境变量
      ANDROID_HOME: /drone/src/sdk    #Android SDK 
    commands:                         #执行命令
      - chmod +x gradlew              #赋权限
      - ./gradlew assembleRelease --stacktrace    #打Release包并打印堆栈
  - name: upload-minio                            #定义上传到MINIO的步骤
    image: minio/mc:RELEASE.2024-01-16T16-06-34Z  #指定镜像和版本
    environment:                                  #变量-这里为Drone->Settings->Secrets中提前定义的变量，放置密钥泄露
      ADDRESS:
        from_secret: MINIO_ADDRESS                #Drone->Settings->Secrets MINIO_ADDRESS
      ACCESS_KEY:
        from_secret: MINIO_ACCESS_KEY             #Drone->Settings->Secrets ACCESS_KEY
      SECRET_KEY:
        from_secret: MINIO_SECRET_KEY             #Drone->Settings->Secrets SECRET_KEY
    commands:
      - mc config host add mio $ADDRESS $ACCESS_KEY $SECRET_KEY   #设定MINIO
      - mc cp app/build/outputs/apk/release/*.apk mio/uchi #mc cp拷贝到MINIO相应路径
volumes:                                                          #卷挂载
  - name: sdk                                                     #宿主机卷名SDK
    host:                                                         
      path: /mnt/user/appdata/drone/tools/android/sdk             #宿主机路径
  - name: gradle-cache                                            #宿主机卷名gradle cache
    host: 
      path: /mnt/user/appdata/drone/tools/gradle/cache-droneci    #宿主机路径
```