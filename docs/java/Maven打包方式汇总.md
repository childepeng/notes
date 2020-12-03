# Java工程打包方式总结
工程构建工具采用Maven，针对不同类型的Java项目会有不同的打包方式，以下总结一下按照如下格式进行打包的几种方式：
1. jar
2. war
3. 自定义

## Jar
jar包是java文件打包的最小单元，包内包含class文件、资源文件（配置文件）、辅助文件（META-INF/*）等；构建一个Jar文件对Mavan来说很简单，默认不需要显示依赖其他插件即可通过`mvn package`实现；但是要构建一个可运行的jar文件就很复杂，因为需要考虑工程依赖、资源文件以及执行入口等问题；以下总结了几种打包方式：

### 不包含依赖
这种方式默认不需要依赖额外的maven插件，直接运行 `mvn package` 即可；

### 包含依赖包（包外依赖）
这种方式将工程依赖复制到指定目录，并在jar内添加classpath路径；通过 `java -jar xxx.jar` 直接运行（前提设置了mainClass）；可以通过以下插件配置实现
```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <configuration>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>  <!-- jar包内添加class-path属性 -->
                <classpathPrefix>lib/</classpathPrefix>
                <mainClass>cc.laop.App</mainClass>
            </manifest>
            <manifestEntries>  
                <Class-Path>.</Class-Path>  <!-- 将当前目录添加到class-path -->
            </manifestEntries> 
        </archive>
        <outputDirectory>${project.build.directory}/${project.name}</outputDirectory>
    </configuration>
</plugin>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <executions>
        <execution>
            <id>copy</id>
            <phase>package</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>
                <outputDirectory>${project.build.directory}/${project.name}/lib</outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>
```
通过`<outputDirectory>`调整jar输出路径；

### 包含依赖（包内依赖）
1. 把工程及其依赖都封装成一个jar包；以下配置会将依赖的jar包解压之后合并为同一个jar包；可以使用插件`maven-assembly-plugin`，但是这种方式存在文件重名的冲突。
```
<plugin>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>2.2</version>
    <configuration>
        <archive>
            <manifest>
                <mainClass>cc.laop.App</mainClass>
            </manifest>
        </archive>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
    </configuration>
    <executions>
        <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
2. 上面的打包形式也可以使用插件`maven-shade-plugin`,该插件可以用于合并资源，并且它可以通过变更文件名称、包名等，解决重名等冲突；
```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <relocations>
                    <!-- 如下，将包名 org.springframework 改为 com.springframework -->
                    <relocation>
                        <pattern>org.springframework</pattern>
                        <shadedPattern>com.springframework</shadedPattern>
                    </relocation>
                </relocations>
                <artifactSet>
                    <excludes>
                        <!--指定哪些依赖不需要打, groupid:artifactid-->
                        <!--<exclude>org.apache.hadoop:*</exclude>-->
                    </excludes>
                </artifactSet>
                <transformers>
                    <!-- transformer可以用来解决资源文件重名的问题，比如spring的xml声明名文件 META-INF/spring.schemas -->
                    <transformer
                            implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <!--设置main class位置-->
                        <mainClass>cc.laop.App</mainClass>
                    </transformer>
                    <transformer
                            implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                        <resource>META-INF/spring.schemas</resource>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```
- org.apache.maven.plugins.shade.resource.ManifestResourceTransformer（重复的main不被合并）
- org.apache.maven.plugins.shade.resource.AppendingTransformer（例如META-INF/spring.handlers与META-INF/spring.schemas，如果出现重名文件，进行追加，而不是文件覆盖）

3. SpringBoot打包插件，实现将工程及其依赖按照特定的目录结构封装为一个jar包
```
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <mainClass>cc.laop.App</mainClass>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>repackage</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
上述的几种打包方式，都是将jar打包成单个包；对于线上环境需要考虑软件更新以及配置变更的问题；
+ 对于动辄十几兆的包，远程上传到服务器，可能都要把人急死；通常希望的更新方式是只上传更新有变更的部分；
+ 由于配置文件也封装到了jar包内，线上环境的配置更新，对于很多新手也是个不小的问题；如果是windows服务器，可以先解压，修改完成之后再压缩成jar；如果是linux服务器，可以使用`vim`直接修改包内文件


## War
war包是JavaWeb程序的默认的封装格式，运行于Tomcat等容器中；
war包默认不需要额外的插件设置，对于javaweb工程，添加`<packaging>war</packaging`，即可实现

## 自定义打包
自定义打包实际上也是构建可运行的jar包，比如实现如下结构：
```
- project
  + bin
  + config
  + logs
  + lib
  README.md
```
自定义打包需要使用到插件`maven-assembly-plugin`
pom.xml
```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.0.2</version>
    <configuration>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <classpathPrefix>./</classpathPrefix>
                <mainClass>cc.laop.App</mainClass>
            </manifest>
            <manifestEntries>
                <Class-Path>../conf/</Class-Path>
            </manifestEntries>
        </archive>
        <excludes>
            <exclude>/*.properties</exclude>
            <exclude>/*.xml</exclude>
        </excludes>
    </configuration>
</plugin>

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>2.5.3</version>
    <executions>
        <execution>
            <id>release</id>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <finalName>${project.name}</finalName>
        <appendAssemblyId>false</appendAssemblyId>
        <descriptors>
            <descriptor>assembly.xml</descriptor>
        </descriptors>
    </configuration>
</plugin>
```
assembly.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.3"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.3
          http://maven.apache.org/xsd/assembly-1.1.3.xsd">

    <id>release package</id>
    <formats>
        <!-- 打包格式，支持zip, tar, tar.gz, tar.bz2, jar, dir, war等 -->
        <format>zip</format>
    </formats>

    <!-- 打包时，包含一层打包目录 -->
    <includeBaseDirectory>true</includeBaseDirectory>

    <!-- 包含程序运行自身所需的目录 -->
    <fileSets>
        <!-- fileSet会对工程目录进行整理，directory表示工程中的目录，outputDirectory表示打包的输出目录，配合使用表示将工程目录中的文件复制到指定的打包目录； fileMode可以设置文件的操作权限；-->
        <fileSet>
            <directory>src/main/resources</directory>
            <outputDirectory>conf</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>src/main/scripts</directory>
            <outputDirectory>bin</outputDirectory>
            <fileMode>0755</fileMode>
            <lineEnding>unix</lineEnding>
        </fileSet>
        <fileSet>
            <outputDirectory>/</outputDirectory>
            <includes>
                <include>README.md</include>
            </includes>
        </fileSet>
    </fileSets>

    <!-- 复制所有的依赖jar到发布的目录的lib目录下 -->
    <dependencySets>
        <dependencySet>
            <scope>runtime</scope>
            <outputDirectory>lib</outputDirectory>
            <!--是否把主程序本身包含进lib目录，默认true-->
            <useProjectArtifact>true</useProjectArtifact>
        </dependencySet>
    </dependencySets>
</assembly>
```

## 总结一下上面用到的几种打包插件
1. maven-jar-plugin  Maven默认的打包插件，用于创建普通的jar文件
2. maven-shade-plugin  用于构建可执行的jar包，可以实现自定义的资源合并，比如多模块合并；并且可以对依赖进行取舍，解决jar包合并时候的文件冲突
3. maven-assembly-plugin  支持定制化打包，可以实现对项目目录及文件的重现组装

