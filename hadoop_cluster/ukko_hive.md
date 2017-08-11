
**Apache Hive Installation.**

**Download and extract**
Download stable version of Hive. 
https://hive.apache.org/downloads.html 

Current stable version : Hive-1.2.2 (release on 7April 2017)

    $tar xvzf apache-hive-1.2.2-bin.tar.gz
    $mv apache-hive-1.2.2-bin apache-hive-1.2.2
    
**Setup Hive Environment**
Open bashrc file and add following lines:
	
    $vim ~/.bashrc

    export HIVE_HOME=<path_to_hive_directory>/apache-hive-1.2.2
    PATH=$PATH:$HIVE_HOME/bin
    export PATH

*(note: run $source ~/.bashrc to make it effective on the running terminal or restart system)*

**Setup HADOOP_PATH in Hive config.sh**

    $cd <hive_directory>/bin
    $vim hive-config.sh
    
Find the line with #HADOOP_HOME and add below line

    export HADOOP_HOME=<hadoop_directory> 
    (e.g. export HADOOP_HOME=/cs/home/sshresth/apache-hadoop-2.7.3)


**Hive Launch**

       $hive

Hive shell  prompt will load:

    Logging initialized using configuration in jar:file:/cs/work/home/users/apache-hive-1.2.2/lib/hive-common-1.2.2.jar!/hive-log4j.properties
    hive>
    hive> show databases;
    OK
    default
    Time taken: 1.419 seconds, Fetched: 1 row(s)
    hive>
   
Installation is completed with default configurations options. This configuration is called embedded metastore i.e Hive driver, metastore interface and the db(Derby) all use the same JVM. It is non-scalable as only a single user can connect to the derby database at any instant of time. The above “hive” command will throw an error stating database exist but lack the metastore services if run again.

---------------------------------------------------------------------------------
Another way to configure is to use an external database server for Metastore. Here we use ***Apache Derby*** database for remote/network . 

**Download and install Apache Derby**
 
**Download from**:
https://db.apache.org/derby/derby_downloads.html
 e.g db-derby-10.10.2.0 (*derby version shipped with Hive can be viewed on derby.log created in Hive embedded mode*)

**Extract**

    $ tar xvzf db-derby-10.10.2.0-bin.tar.gz
    $ mv db-derby-10.10.2.0-bin  db-derby-10.10.2.0

 
**Set Environment for Derby**
   
     $ vim ~/.bashrc
     
Add the following lines:    

      #set derby home
      export DERBY_HOME=<path_to_derby_directory>/db-derby-10.10.2.0
      export PATH=$PATH:$DERBY_HOME/bin
      export CLASSPATH=$CLASSPATH:$DERBY_HOME/lib/derby.jar:$DERBY_HOME/lib/derbytools.jar
*( “ $source ~/.bashrc ”  command to execute ~/.bashrc file)*
 
**Create directory to store Metastore**

    $mkdir $DERBY_HOME/data

 **Start Derby**
By default Derby creates database in the directory it was started from.
   
     $cd $DERBY_HOME/data
     $../bin/startNetworkServer -h 0.0.0.0 &
 
---------------------------------------------------------------------------------
**Create/Set permission to Hive directories within HDFS**
  
     $hadoop fs -mkdir  /warehouse
     $hadoop fs -chmod g+w /warehouse
     
**Configure Hive to use Network/remote Derby**
copy the template file

    $cp apache-hive-1.2.2/conf/hive-default.xml.template apache-hive-1.2.2/conf/hive-site.xml

 edit the properties on hive-site.xml file as given below:

    <property>
	    <name>javax.jdo.option.ConnectionURL</name>
	    <value>jdbc:derby://<Hadoop_server_URL>:1527/metastore_db;create=true</value>
	    <description>JDBC connect string for a JDBC metastore </description>
    </property> 
    <property>
	    <name>javax.jdo.option.ConnectionDriverName</name>
	    <value>org.apache.derby.jdbc.ClientDriver</value>
	    <description>Driver class name for a JDBC metastore</description>
    </property>
    <property>
	    <name>hive.metastore.warehouse.dir</name>
	    <value>/warehouse</value>
	    <description>location of default database for the warehouse</description>
    </property>

**Create jpox.properties files**
Add the following lines on config/jpox.properties :

    javax.jdo.PersistenceManagerFactoryClass=org.jpox.PersistenceManagerFactoryImpl
      org.jpox.autoCreateSchema=false
      org.jpox.validateTables=false
      org.jpox.validateColumns=false
      org.jpox.validateConstraints=false
      org.jpox.storeManagerType=rdbms
      org.jpox.autoCreateSchema=true
      org.jpox.autoStartMechanismMode=checked
      org.jpox.transactionIsolation=read_committed
      javax.jdo.option.DetachAllOnCommit=true
      javax.jdo.option.NontransactionalRead=true
      javax.jdo.option.ConnectionDriverName=org.apache.derby.jdbc.ClientDriver
      javax.jdo.option.ConnectionURL=jdbc:derby://<Hadoop_server_URL>:1527/metastore_db;create=true
      javax.jdo.option.ConnectionUserName=APP
      javax.jdo.option.ConnectionPassword=mine
      
**Copy Derby jar file**

    $cp <derby_directory>/lib/derbyclient.jar <hive_directory>/lib/
    $cp <derby_directory>/lib/derbytools.jar <hive_directory>/lib/
    
**Start Up Hive**

    $hive
    hive>show tables;
    	>exit;

 
The metastore will be created after the first query, directory is created on derby data(i.e derby start directory) directory. 
(in case of “java.net.URISyntaxException: Relative path in absolute URI:” error message 
Add on below line on hive-site.xml file on top )

    <property>
    <name>system:java.io.tmpdir</name>
    <value>/tmp/hive/java</value>
    </property>
    <property>
    <name>system:user.name</name>
    <value>${user.name}</value>
    </property>

Some configuration files to remember

 -  Hive by default gets its configuration from <install-dir>/conf/hive-default.xml
 - Configuration variables can be changed by (re-)defining them in <install-dir>/conf/hive-site.xml
 - Log4j configuration is stored in <install-dir>/conf/hive-log4j.properties
 - hive log file is default at /tmp/hive/hive.log

**Example Use case** 

Create table with “,” delimited text file format:

    $ vim sample.sql    
   
    CREATE TABLE patent (
            patent       INT,
            gyear        INT,
            gdate        INT,
            appyear      INT,
            country      VARCHAR(3),
            postate      VARCHAR(3),
            assignee     INT,
            asscode      INT,
            claims       INT,
            nclass       INT,
            cat          INT,
            subcat       INT,
            cmade        INT,
            creceive     INT,
            ratiocit     INT,
            general      INT,
            original     INT,
            fwdaplag     INT,
            bckgtlag     INT,
            selfctub     INT,
            selfctlb     INT,
            secdupbd     INT,
            secdlwbd     INT
    )
    ROW FORMAT DELIMITED
    FIELDS TERMINATED BY ','
    STORED AS TEXTFILE;

create patent table with hive command below;

    $hive -f sample.sql

download and extract data from 
http://udbms.cs.helsinki.fi/assets/DataSet/Patent_dataset.zip

Load patent.table into the patent table:
	
    $hive 
    hive> LOAD DATA LOCAL INPATH ‘[path_to_patent.table_file]’
    OVERWRITE INTO TABLE patent;

Count the number of rows in the table patent;
 
    hive> SELECT COUNT(*) FROM patent;
        Output:
        MapReduce Jobs Launched:
        Stage-Stage-1: Map: 1  Reduce: 1   Cumulative CPU: 5.6 sec   HDFS Read: 236916469 HDFS Write: 8 SUCCESS
        Total MapReduce CPU Time Spent: 5 seconds 600 msec
        OK
        2923923
        Time taken: 23.842 seconds, Fetched: 1 row(s)
	
