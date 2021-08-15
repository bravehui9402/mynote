1、将solr安装包下server/solr-webapp/webapp下的solr服务打war包；

```shell
jar cvf  solr.war ./*
```

2、将solr.war复制到tomcat/webapps目录中，启动tomcat，解压war包；

3、修改tomcat/webapp/solr/WEB-INF/web.xml的配置solr_home的位置；

```
<env-entry>
   	<env-entry-name>solr/home</env-entry-name>
   	<env-entry-value>“你的solrhome位置”</env-entry-value>
   	<env-entry-type>java.lang.String</env-entry-type>
 </env-entry>
```

取消安全配置

```
<!--
  <security-constraint>
  <web-resource-collection>
      <web-resource-name>Disable TRACE</web-resource-name>
    <url-pattern>/</url-pattern>
      <http-method>TRACE</http-method>
    </web-resource-collection>
    <auth-constraint/>
  </security-constraint>
  <security-constraint>
    <web-resource-collection>
      <web-resource-name>Enable everything but TRACE</web-resource-name>
      <url-pattern>/</url-pattern>
      <http-method-omission>TRACE</http-method-omission>
    </web-resource-collection>
  </security-constraint>-->
```

4、将solr-7.7.2/server/solr中所有的文件复制到solrHome

5、拷贝日志工具相关jar包：将solr-7.7.2/server/lib/ext下的jar包拷贝至上面Tomcat下Solr的/WEB-INF/lib/目录下

6、拷贝metrics相关jar包：将solr-7.7.2/server/lib下metrics相关jar包也拷贝至/WEB-INF/lib/目录下

7、拷贝dataimport相关jar包:solr-7.7.2/dist下dataimport相关jar包也拷贝至/WEB-INF/lib/目录下

8、拷贝log4j2配置文件：将solr-7.7.2/server/resources目录中的log4j配置文件拷入web工程目录WEB-INF/classes（自行创建目录） ，并且修改日志文件的路径

9、在tomcat的bin\catalina.bat中配置日志文件的环境参数

```
set "JAVA_OPTS=%JAVA_OPTS% -Dsolr.log.dir={路径}\logs"
```

10、将tomcat复制4份到solrcloud中，便于集中管理。这4个tomcat中都部署了solr；

```
cp -r apache-tomcat-8.5.50 solrCloud/tomcat1
cp -r apache-tomcat-8.5.50 solrCloud/tomcat2
cp -r apache-tomcat-8.5.50 solrCloud/tomcat3
cp -r apache-tomcat-8.5.50 solrCloud/tomcat4
```

11、复制4个solrhome到solrcloud中。分别对应每一个solr的solrhome。

```
cp -r solrhome solrCloud/solrhome1
cp -r solrhome solrCloud/solrhome2
cp -r solrhome solrCloud/solrhome3
cp -r solrhome solrCloud/solrhome4
```

12、修改tomcat的配置文件，修改tomcat的端口，保证在一台计算机上，可以启动4个tomcat；编辑server.xml

```
修改停止端口
对外提供服务端口
AJP端口
<Server port="8005"shutdown="SHUTDOWN">
<Connector port="8080"protocol="HTTP/1.1" connectionTimeout="20000" redirect
<Connector port="8009" protocol="AJP/1.3"redirectPort="8443" />
tomcat1  8105  8180  8109
tomcat1  8205  8280  8209
tomcat1  8305  8380  8309
tomcat1  8405  8480  8409
```

13、为tomcat中每一个solr指定正确的solrhome，目前是单机版solrhome的位置。

​	编辑solr/web.xml指定对应的solrhome

```
	 <env-entry>
		<env-entry-name>solr/home</env-entry-name>
		<env-entry-value>/usr/local/solrcloud/solrhomeX</env-entry-value>
		<env-entry-type>java.lang.String</env-entry-type>
  	</env-entry>
```

14、修改每个solrhome中solr.xml的集群配置,对应指定tomcat的ip和端口

​	编辑solrhome中solr.xml

```
 <solrcloud>
    <str name="host">192.168.200.1288</str>
    <int name="hostPort">8X80</int>
 </solrcloud>
```

15、让Zookeeper管理Solr集群。编辑每一个tomcat中bin/catalina.sh文件，指定Zookeeper集群地址。

```
  JAVA_OPTS="-DzkHost=121.4.88.116:2181,121.4.88.116:2182,121.4.88.116:2183"
  需要指定客户端端口即Zookeeper对外提供服务的端口
```

16、让zookeeper统一管理solr集群的配置文件。

```
将conf目录中的配置文件，上传到Zookeeper，以后集群中的配置文件就以Zookeeper中的为准；
搭建好集群后，solrCore中是没有配置文件的。

进入到solr安装包中
/root/solr-7.7.2/server/scripts/cloud-scripts

./zkcli.sh -zkhost 121.4.88.116:2181,121.4.88.116:2182,121.4.88.116:2183 -cmd upconfig -confdir /app/solr/solr-7.7.3/example/example-DIH/solr/solr/conf -confname myconf

-zkhost：指定zookeeper的地址列表；
upconfig ：上传配置文件；
-confdir ：指定配置文件所在目录；
-confname：指定上传到zookeeper后的目录名；
```

