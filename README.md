"# SylarMavenTest"

主要是为了实现jmeter在maven中实现后集成到jenkins

1、jmx脚本需要在jmeter中正常执行，不需要报告；

2、运行时使用mvn verify命令执行。

pom文件中jmeter-maven-plugin的版本慎选，有issue请参看jmeter-maven-plugin的wiki

目前第一版，没有集成数据库。


