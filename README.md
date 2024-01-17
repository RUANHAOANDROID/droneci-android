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
      - mc config host add mio $ADDRESS $ACCESS_KEY $SECRET_KEY
      - cd app/build/outputs/apk/release
      - apk_file=`ls | head -1`
      - mc cp $apk_file mio/uchi
volumes:
  - name: sdk
    host:
      path: /mnt/user/appdata/drone/tools/android/sdk
  - name: gradle-cache
    host:
      path: /mnt/user/appdata/drone/tools/gradle/cache-droneci
```