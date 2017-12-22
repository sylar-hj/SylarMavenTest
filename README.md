# Jmeter+maven+jenkins集成
## 一、需要用到的工具
- jdk
- jmeter
- maven
- idea
- git
- jenkins
- github
- tomcat
***
### 1、jmeter
我使用的是apache-jmeter-3.2，3.0以上的都可以。

安装过程不赘述了，依赖于jdk。

jmeter某些低版本无法正确显示json中的中文，需要改一下文件：

> `jmeter。properties`

> `jsyntaxtextarea.font.family=Hack `

### 2、maven
我使用的是3.5.0

### 3、编译器
看个人习惯，eclipse或者idea都可以

### 4、代码管理工具
我使用的是git，代码库用的github

### 5、构建工具
jenkins最新版本

### 6、服务器
看个人喜好，我用的tomcat8

## 二、步骤

### 1、编写jmeter脚本
### 2、新建maven工程，调试项目至运行成功，涉及到的依赖如下：

```
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>3.8.1</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.13</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/eu.freme-project.bservices.controllers/xslt-converter -->
        <dependency>
            <groupId>eu.freme-project.bservices.controllers</groupId>
            <artifactId>xslt-converter</artifactId>
            <version>1.3</version>
        </dependency>


    </dependencies>

    <properties>
        <jmeter.result.jtl.dir>${project.build.directory}\jmeter\results</jmeter.result.jtl.dir>
        <jmeter.result.html.dir>${project.build.directory}\jmeter\html</jmeter.result.html.dir>
        <jmeter.result.html.dir1>${project.build.directory}\jmeter\html1</jmeter.result.html.dir1>
        <reportname>TestReport</reportname>
    </properties>

    <pluginRepositories>
        <pluginRepository>
            <id>Codehaus repository</id>
            <url>http://repository.codehaus.org/</url>
            <releases>
                <enabled>true</enabled>
                <updatePolicy>always</updatePolicy>
            </releases>
            <snapshots>
                <enabled>true</enabled>
                <updatePolicy>always</updatePolicy>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>

    <build>
        <finalName>sylar</finalName>
        <plugins>
            <plugin>
                <groupId>com.lazerycode.jmeter</groupId>
                <artifactId>jmeter-maven-plugin</artifactId>
                <version>2.5.0</version>

                <executions>
                    <!-- Run JMeter tests -->
                    <execution>
                        <id>jmeter-tests</id>
                        <phase>verify</phase>
                        <goals>
                            <goal>jmeter</goal>
                        </goals>
                    </execution>
                    <!-- Fail build on errors in test -->
                    <execution>
                        <id>jmeter-check-results</id>
                        <goals>
                            <goal>results</goal>
                        </goals>
                    </execution>
                </executions>
                <!--这里有个大坑，慎设置参数-->
                <configuration>
                    <resultsFileFormat>xml</resultsFileFormat>
                    <!--<generateReports>true</generateReports>-->
                    <ignoreResultFailures>true</ignoreResultFailures>
                    <testResultsTimestamp>false</testResultsTimestamp>
                </configuration>
            </plugin>

            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.6</source>
                    <target>1.6</target>
                    <encoding>UTF-8</encoding>
                    <compilerArguments>
                        <extdirs>src\test\jmeter\lib</extdirs>
                    </compilerArguments>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>xml-maven-plugin</artifactId>
                <version>1.0-beta-3</version>
                <executions>
                    <execution>
                        <phase>verify</phase>
                        <goals>
                            <goal>transform</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <transformationSets>
                        <transformationSet>
                            <dir>${jmeter.result.jtl.dir}</dir>
                            <stylesheet>src\test\resources\jmeter-results-detail-report_21.xsl</stylesheet>
                            <outputDir>${jmeter.result.html.dir}</outputDir>
                            <fileMappers>
                                <fileMapper
                                        implementation="org.codehaus.plexus.components.io.filemappers.FileExtensionMapper">
                                    <targetExtension>html</targetExtension>
                                </fileMapper>
                            </fileMappers>
                        </transformationSet>
                        <transformationSet>
                            <dir>${jmeter.result.jtl.dir}</dir>
                            <stylesheet>src\test\resources\jmeter.results.shanhe.me.xsl</stylesheet>
                            <outputDir>${jmeter.result.html.dir1}</outputDir>
                            <fileMappers>
                                <fileMapper
                                        implementation="org.codehaus.plexus.components.io.filemappers.FileExtensionMapper">
                                    <targetExtension>html</targetExtension>
                                </fileMapper>
                            </fileMappers>
                        </transformationSet>
                    </transformationSets>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <executions>
                    <execution>
                        <id>copy-dependencies</id>
                        <phase>prepare-package</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>${project.build.directory}/jmeter/lib</outputDirectory>
                            <overWriteReleases>false</overWriteReleases>
                            <overWriteSnapshots>false</overWriteSnapshots>
                            <overWriteIfNewer>true</overWriteIfNewer>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <classpathPrefix>lib/</classpathPrefix>
                            <mainClass>theMainClass</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>

            <plugin>
                <artifactId>maven-resources-plugin</artifactId>
                <executions>
                    <execution>
                        <id>copy-resources</id>
                        <phase>compile</phase>
                        <goals>
                            <goal>copy-resources</goal>
                        </goals>
                        <configuration>
                            <!-- test可以在环境变量中定义，也可以将下面写成绝对地址 -->
                            <outputDirectory>${project.build.directory}/jmeter/html</outputDirectory>
                            <resources>
                                <resource>
                                    <!-- basedir标识所有工程 -->
                                    <directory>${basedir}/src/test/resources</directory>
                                    <filtering>true</filtering>
                                </resource>
                            </resources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

        </plugins>

    </build>
```
注意，jmeter-maven-plugin使用的命令为mvn verify

### 3、将工程上传至github

### 4、jenkins安装
#### 用到的插件
- git plugin
- Performance Plugin
- HTML Report Plugin

jenkins正确显示html文件需要执行的命令：

`System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "")`

### 5、jenkins集成maven



##   三、已知问题：
### 1、min和max显示为NaN
xml转换html插件原因，jtl转换为html后，这两个字段的值显示不出来，待解决；
### 2、performance图形化报告为空
performance的数据源确认有数据，构建了多次，只有一次有图形展示，目前还不清楚原因。

## Jenkins未实现的其他扩展：
### 1、集成数据库依赖，对数据库进行压测；
### 2、邮件推送报告内容或报告结果；
### 3、与开发环境集成，每次源码发生变动后自动触发构建。












