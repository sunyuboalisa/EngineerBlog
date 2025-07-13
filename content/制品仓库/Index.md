---
longform:
  format: scenes
  title: 制品仓库
  workflow: Default Workflow
  sceneFolder: /
  scenes: []
  ignoredFiles: []
---
# npm 私有仓库
## Verdaccio

```sh
npm install -g verdaccio
yarn global add verdaccio
```
这个可以批量同步内网机和外网机的npm包
首先在外网机配置并启动verdaccio
然后去项目上执行npm install，缓存项目中的依赖
之后在storage中复制出包，并拷贝到外网机verdaccio服务器的storage中。

JAVA 环境配置
下载jdk
配置JAVA_HOME
配置PATH
验证安装 java --version

maven配置
配置PATH
镜像配置
```xml
<mirror>
  <id>aliyunmaven</id>
  <mirrorOf>*</mirrorOf>
  <name>阿里云公共仓库</name>
  <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```
# Maven 私有仓库
## Reposilite Repository
第一次用jar包方式启动，并配置用户名和密码
java -Xmx128M -jar .\reposilite-3.5.25-all.jar --token alisa:#Alisa96312
然后登录ui后，在console中执行
token-generate alisa m
记住生成的token然后重新启动
java -Xmx128M -jar .\reposilite-3.5.25-all.jar
这次登录用刚刚的token
接下来配置maven的settings.xml文件
```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.2.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.2.0 http://maven.apache.org/xsd/settings-1.2.0.xsd">
  <localRepository>D:/DevEnvironment/.m3/repository</localRepository>
  <servers>
    <server>
      <id>reposilite-repository-releases</id>
      <username>alisa</username><password>MZYI1CxGi54YtOWBOFY0RcuyjDyKA0s5ulS/YT+1EhNwT8udgyy7NzAD66TH3It8</password>
    </server>
    <server>
      <id>reposilite-repository-snapshots</id>
      <username>alisa</username><password>MZYI1CxGi54YtOWBOFY0RcuyjDyKA0s5ulS/YT+1EhNwT8udgyy7NzAD66TH3It8</password>
    </server>
  </servers>
  
  <mirrors>
    <mirror>
      <id>reposilite</id>
      <mirrorOf>*</mirrorOf>
      <name>Reposilite Mirror</name>
      <url>http://localhost:8080/releases</url>
    </mirror>
  </mirrors>
  
  <profiles>
    <profile>
      <id>reposilite</id>

      <repositories>
        <repository>
          <id>reposilite-repository-releases</id>
          <name>Reposilite Releases</name>
          <url>http://localhost:8080/releases</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>false</enabled></snapshots>
        </repository>
        <repository>
          <id>reposilite-repository-snapshots</id>
          <name>Reposilite Snapshots</name>
          <url>http://localhost:8080/snapshots</url>
          <releases><enabled>false</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </repository>
      </repositories>

      <pluginRepositories>
        <pluginRepository>
          <id>reposilite-repository-plugins</id>
          <name>Reposilite Plugin Repo</name>
          <url>http://localhost:8080/releases</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>
  <activeProfiles>
    <activeProfile>reposilite</activeProfile>
  </activeProfiles>
</settings>
```

