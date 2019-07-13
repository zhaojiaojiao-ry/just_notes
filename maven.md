## maven生命周期和阶段

3个生命周期lifecycle，相互独立，比如执行default不会默认执行clean

- clean：清理
- default：构建
- site：建立项目站点

每个生命周期包含一些阶段phrase，阶段是有依赖顺序的，比如生命周期clean有阶段pre-clean、clean、post-clean，用户调用clean会依次执行pre-clean、clean

以下列出每个生命周期下的重要阶段

clean

- clean：清理上次构建文件

default

- compile：编译，输出到主classpath目录中
- test：单元测试
- package：打包，如jar
- install：安装包到maven本地仓库，供本地其他maven项目使用
- deploy：将包复制到远程仓库，供其他maven项目使用

site

- site生成项目站点文档

## maven命令

### 构建命令

maven构建命令是phrase粒度，比如 maven clean package，是执行clean生命周期的pre-clean、clean阶段，default生命周期的package之前的（包括package）全部阶段

### 查看依赖

基础版

maven dependency:tree

详情版

maven dependency:tree -Dverbose

指定关注的组件

maven dependency:tree -Dverbose -Dincludes=groupId:artifactId

## maven命令参数

mvn -v	查看maven版本

mvn -Pxxx	激活id为xxx的profile

mvn -Dxxx=yyy	指定java全局属性xxx=yyy

mvn -e	控制maven日志级别为error

mvn -B	非交互模式运行，当maven需要输入时，不会停下里接受用户输入，而是使用合理默认值

mvn -U	强制更新snapshot类型的插件或依赖库，否则maven一天只更新一次snapshot类型的依赖

## maven依赖项

3个属性唯一确定一个依赖项（jar包 war包）：groupId、artifactId、version

快照版本vs正式版本

- 从version看：version中有-SNAPSHOT后缀的是快照版本，否则是正式版本
- 从获取方式看
  - 正式版本：如果依赖一个正式版本，构建时会检查本地仓库是否已有该版本，如果有则不去远程仓库拉取，否则去远程仓库拉取。所以，如果正式版本远程仓库有改动，但版本号没有升级，本地是获取不到最新版本内容的，必须要升级版本号才行。
  - 快照版本：可以在settings.xml配置repository级别的updatePolicy，在构建时按照指定频率去远程仓库拉取本地仓库已有的快照版本，从而不升级版本号也能升级到最新版本内容。频率可能是always、daily、interval、never。never的行为等价于正式版本的行为。mvn -U就是指定了当次构建是强制更新。

综上，一般开发模式用快照版本，可不修改version就更新最新内容，稳定后用正式版本，同时系统不再依赖快照版本，保证依赖内容稳定。

## maven pom配置

### 基础配置

```xml
<!--父module-->
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>jjorg</groupId>
  <artifactId>jjdemoparent</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>pom</packaging>
  <modules>
    <module>jjdemosub1</module>
    <module>jjdemosub2</module>
  </modules>
</project>
<!--子module-->
<project>
  <modelVersion>4.0.0</modelVersion>
  <parent>  
    <groupId>jjorg</groupId>
  	<artifactId>jjdemoparent</artifactId>
  	<version>0.0.1-SNAPSHOT</version>
  </parent>
	<artifactId>jjdemosub1</artifactId>
  <packaging>jar</packaging>
</project>
```

packaging打包机制

- jar：默认
- pom：多module结构时，父module使用pom
- war

父子module

- 父module中，使用<modules>定义子module
- 子module中，使用<parent>定义父module

### 属性

pom属性，可在pom中直接引用，如${project.groupId}是项目的groupId

用户自定义属性，在<properties>中定义，在pom中可引用自定义属性${xxx.yyy}

```xml
<properties>
  <xxx.yyy>zzz</xxx.yyy>
</properties>
```

### 依赖管理

#### dependencies

```xml
<dependencies>
  <dependency>
    <groupId></groupId>
    <artifactId></artifactId>
    <version></version>
    <scope></scope>
    <exclusions>
      <exclusion>
        <groupId></groupId>
        <artifactId></artifactId>
      </exclusion>
    </exclusions>
  </dependency>
</dependencies>
```

- 3个属性唯一确定一个依赖项（jar包 war包）：groupId、artifactId、version

- 快照版本vs正式版本

  - 从version看：version中有-SNAPSHOT后缀的是快照版本，否则是正式版本
  - 从获取方式看
    - 正式版本：如果依赖一个正式版本，构建时会检查本地仓库是否已有该版本，如果有则不去远程仓库拉取，否则去远程仓库拉取。所以，如果正式版本远程仓库有改动，但版本号没有升级，本地是获取不到最新版本内容的，必须要升级版本号才行。
    - 快照版本：可以在settings.xml配置repository级别的updatePolicy，在构建时按照指定频率去远程仓库拉取本地仓库已有的快照版本，从而不升级版本号也能升级到最新版本内容。频率可能是always、daily、interval、never。never的行为等价于正式版本的行为。mvn -U就是指定了当次构建是强制更新。

  综上，一般开发模式用快照版本，可不修改version就更新最新内容，稳定后用正式版本，同时系统不再依赖快照版本，保证依赖内容稳定。

- scope标识这个依赖要参与project的哪个阶段

  - compile：编译、运行依赖，默认就是compile
  - test：测试代码的编译、运行依赖，如：junit
  - runtime：编译不依赖，运行依赖，如：jdbc driver，编译时只需要接口，不需要具体driver实现
  - provided：编译依赖，运行不依赖，运行时环境里有现成的依赖包可用，所以打包时可以exclude这个依赖

- 通过<exclusion>可以排除不想依赖的module

#### dependencyManagement

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId></groupId>
      <artifactId></artifactId>
      <version></version>
      <scope></scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

- 使用场景：有多个module需要共同管理，module使用了共同的依赖，可以在总module中管理version&scope，不用在各个module中单独管理

#### dependencyManagement和dependencies的区别

- dependencies：决定是否引入的
- dependencyManagement：只是定义，并不引入，只有dependencies决定引入的且没有定义version、scope的，可以在parent module的dependencyManagement中找到version、scope来继承
- 举例
  - parent module有dependencies：module引入自己的dependencies和parent module的dependencies，自己的dependencies的version和scope优先
  - parent module有dependencyManagement：module引入自己的dependencies，如果没有指定version和scope的，会从parent module的dependencyManagement中获取

#### 使用scope=import,type=pom解决单继承问题

使用父module中dependencyManagement管理所有子module需要的依赖，子module中通过parent指向父module，可以达到单继承的效果，但可能导致父module中dependencyManagement定义过多，pom过长

解法：将子module所需的依赖分类，每类定义在单独的pom中，在子module中使用dependencyManagement自行引入这些pom，可以达到多继承的效果

注意：scope=import和type=pom要一起使用，且只能用在dependencyManagement里的dependency上

### 构建配置

```xml
<build>
  <finalName>jjdemosub1</finalName>
  <directory>${baseDir}/target</directory>
  <resources>
    <resource>
      <directory>${baseDir}/src/main/resources</directory>
      <targetPath></targetPath>
      <includes>
        <include>*.xml</include>
      </includes>
      <excludes>
        <exclude>**/*.properties</exclude>
      </excludes>
      <filtering>true</filtering>
    </resource>
  </resources>
  <filters>
    <filter>config.properties</filter>
  </filters>
</build>
```

finalName：构建文件名称，默认是${artifactId} - \${version}

directory：构建文件保存目录，默认是${baseDir}/target，即module根目录下的target目录

#### resources

resource：构建中使用的配置等资源文件

- directory：资源文件所在目录，默认是${baseDir}/src/main/resources目录
- targetPath：资源文件构建后的目标目录，不填则与directory一样
- include：资源文件匹配模式，成功匹配的资源文件才会被处理，\**匹配任意路径，\*匹配非路径的其他0到多个字符
- exclude：资源文件匹配模式，成功匹配的资源文件不会被处理
- filtering：true/false，决定资源文件中的${key}是否会被替换，如果为true的话，要被替换
  - 使用什么替换？
    - filter文件中的配置项，key=value
    - pom中定义的属性，<properties><key>value</key></properteis>
    - 命令行定义的属性，-Dkey=value

filter：当resource.filtering=true时，用这个文件中的key=value配置，来替换resource文件中的${key}

#### plugin

plugin：maven的每个lifecycle phrase都默认绑定了plugin和plugin的goal

- maven-clean-plugin：clean
- maven-compiler-plugin：compile
- maven-surefile-plugin：test
- maven-jar-plugin：jar
- maven-install-plugin：install
- maven-deploy-plugin：deploy

如果针对plugin有特殊配置，在<plugin>中指定

- executions：定义将插件提供的指定goal绑定到phrase执行，并指定plugin的配置参数
  - goal：绑定插件提供的哪个goal到phrase
  - phrase：绑定插件到哪个phrase
  - configuration：自定义配置参数值

子module会继承父module的plugin和pluginManagement中的属性设置

```xml
<plugins>
  <plugin>
    <artifactId>插件名</artifactId>
    <version>插件版本</version>
    <executions>
      <execution>
        <id></id>
        <phrase></phrase>
        <goals>
          <goal></goal>
        </goals>
        <configuration>
      		<配置key>配置value</配置key>
    		</configuration>
      </execution>
    </executions>
  </plugin>
</plugins>
```

plugin列表：https://maven.apache.org/plugins/index.html

### 多profile

```xml
<profiles>
  <profile>
    <id>dev</id>
    <activation>
      <activeByDefault>true</activeByDefault>
    </activation>
    <dependencies>
    </dependencies>
    <build>
    </build>
  </profile>
</profiles>
```

通过profile定义不同环境使用不同的依赖、构建配置

id：标识一个profile，在命令行中使用-Pdev可以启动对应的profile配置

activeByDefault：标识是否是默认profile

dependencies和build，参考project级别的配置含义

#### maven的多profile和springboot的多profile

maven：通过maven -P指定profile，查找pom中对应id的profile和project级别的配置，一起构成配置，前者覆盖后者的配置

springboot：通过-Dspring.profiles.active=指定profile，查找application-${profile}.properties和application.properties，一起构成配置，前者覆盖后者的配置

