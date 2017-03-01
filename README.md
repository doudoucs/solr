# solr
solr搭建流程及使用

主要记录搭建环境中的遇到的坑


1、solr 依赖java环境、因此需要安装JDK以及设置对应的JAVA_HOME环境变量

2、brew install solr

3、安装完成后：localhost:8983/solr 进入solr的管理界面

4、建立一个名为test_core的core

1)在solr的安装目录下找server/solr目录，并进入，在该目录下mkdir -p test_core/conf 

2）将server\solr\configsets\data_driven_schema_configs\conf下的所有东西复制到新建的conf中去

3）将olr-6.0\solr-6.0.0\example\example-DIH\solr\db\conf下的admin-extra*.jar三个文件也复制到conf中去

4）mysql所需的jar包和solr-dataimporthandler-6.0.0.jar和solr-dataimporthandler-extras-6.0.0.jar都复制到项目solr-webapp/webapp/WEB-INF/lib下

5)在solrconfig.xml约75行，加入<lib dir="tomcat/webapps/solr/WEB-INF/lib/" regex=".*\.jar" />  就是将WEB-INF\lib里面的jar包配置到项目中，tomcat对应的WEB-INF/lib我没找到。。。。。

5、在mysql中建立一张表，title 

6、在solrconfig.xml的 ``` <requestHandler name="/select" class="solr.SearchHandler"> ``` 之上添加

```
<requestHandler name="/dataimport" class="org.apache.solr.handler.dataimport.DataImportHandler">  
　     <lst name="defaults">  
　        <str name="config">data-config.xml</str>  
　     </lst>  
　</requestHandler>  
 
 ```
 
 7、在conf下新建data-config.xml文件
 内容如下：
 
 ```
 <?xml version="1.0" encoding="UTF-8"?>
<dataConfig>
  <dataSource name="source1" type="JdbcDataSource" driver="com.mysql.jdbc.Driver" url="jdbc:mysql://127.0.0.1:3306/solr_core" user="root" password="root" batchSize="-1" />
  　　<document>
          <entity name="title" pk="id"  dataSource="source1"
          query="select id,name from title"
          deltaImportQuery="select id,name from title where id='${dih.delta.id}'"
          deltaQuery="select id from title where updateTime> '${dataimporter.last_index_time}'">

    　　　      <field column="id" name="id"/>
    　　　      <field column="name" name="name"/>
    　　　  </entity>
  　　</document>
</dataConfig>

```

8、conf文件下的managed-schema配置field信息
<field name="id" type="int" indexed="true" stored="true" required="true" multiValued="false" />
<field name="name" type="string" indexed="true" stored="true"/>


9、访问http://127.0.0.1:8080/solr/ 这时就可以新建名为test_core的core啦。

10、选中新建的core后就可以点击dataimport新建索引啦，但是到这步后，失败啦。。。看日志报错为org.apache.solr.common.SolrInputDocument: method <init>()V not found，找不到解决办法。。。。。
