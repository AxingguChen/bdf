#How-to: Setup a multi-node Hadoop cluster on Ukko

orignially from https://www.cs.helsinki.fi/u/pengxu/ukko.pdf

##Prerequisites
1. Before you start, please read Ukko instructions carefully: [https://www.cs.helsinki.fi/en/compfac/highperformance-cluster-ukko](https://www.cs.helsinki.fi/en/compfac/highperformance-cluster-ukko)

2. You need to gain access to the Ukko cluster if you are not in the University network. The most
convenient way is by using HY-VPN (Use OpenVPN, not Pulse Secure):
[https://helpdesk.it.helsinki.fi/en/search?keys=HY-VPN](https://helpdesk.it.helsinki.fi/en/search?keys=HY-VPN)

3. In order to transfer files between your computer and Ukko, you also need to have a SSH software (e.g.
Putty for Windows) and a SFTP client (WinSCP for Windows, Cyberduck for OS X, FileZilla for
Linux).

##Choose your nodes
1. You should choose serveal nodes from the list [https://www.cs.helsinki.fi/ukko/hpc-report.txt](https://www.cs.helsinki.fi/ukko/hpc-report.txt).
Note that the node marked with “Reserved” is not available. Please also note the “load” factor.
2. Choose one of the chosen nodes be the namenode. All other nodes will become datanodes. Here
we choose ukko026 be the namenode, ukko027, ukko028 and ukko029 be the datanodes.

##Setup Hadoop

###PART A: On your namenode
1. Use your SSH software to login to your namenode. Navigate to /cs/work/scratch/ folder. Make you own working directory here (NB: in case you don't have folder under /cs/work/home/).

2. Download and extract Hadoop 2.7.3 binary distribution to your working directory. Change your current directory to your_working_dir/hadoop-2.7.3/: 
 
        $ wget [http://mirror.netinch.com/pub/apache/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz](http://mirror.netinch.com/pub/apache/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz)
        $ tar xvf hadoop-2.7.3.tar.gz
        $ cd hadoop-2.7.3  

3. Add $JAVA_HOME environment variable. Add the following three lines to the beginning of etc/hadoop/hadoop-env.sh:

        export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
        export PATH=${JAVA_HOME}/bin:${PATH}
        export HADOOP_CLASSPATH=${JAVA_HOME}/lib/tools.jar

4. Modify etc/hadoop/core-site.xml, put the following lines between <configuration> and </configuration>:


    <property>
        <name>fs.default.name</name>
        <value>hdfs://ukko026.hpc.cs.helsinki.fi:9000</value>
    </property>
You need to change the underlined hostname matches your choice of namenode. The port number 9000 is usually not necessarily being changed, unless it is occupied by someone else.

5. Modify etc/hadoop/mapred-site.xml, put the following lines between <configuration> and </configuration>:


    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>

6. Modify etc/hadoop/yarn-site.xml, put the following lines between <configuration> and </configuration>: 
 
 
        <property>
            <name>yarn.resourcemanager.hostname</name>
            <value>ukko026.hpc.cs.helsinki.fi</value>
        </property>
        <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
        <property>
            <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
            <value>org.apache.hadoop.mapred.ShuffleHandler</value>
        </property>
You need to change the underlined hostname matches your choice of namenode.

7. Format the namenode by using command.

        $ ./bin/hadoop namenode –format  
        
This is only required once, so you don’t need to format it every time you run Hadoop.

8. Start Hadoop on the namenode by using 

        $ ./sbin/hadoop-daemon.sh
start namenode.

9. Check the log logs/hadoop-your_username-namenode-ukko026.log to see if the namenode is up. The last two lines should be
    Starting services required for active state
    Starting CacheReplicationMonitor with interval 30000 milliseconds

10. Open your web browser and navigate to [http://ukko026.hpc.cs.helsinki.fi:50070/](http://ukko026.hpc.cs.helsinki.fi:50070/) (replace ukko026 with the name of your namenode, you can not open this unless you using office wire connection. You can check the log instead).

11. Start YARN Resource Manager on the namenode by using ./sbin/yarn-daemon.sh start resourcemanager.

12. Check the log logs/hadoop-your_username-resourcemanager-ukko026.log to see if the namenode is up. The last line should be
    IPC Server listener on 8033: starting
    
13. Open your web browser and navigate to [http://ukko026.hpc.cs.helsinki.fi:8088/](http://ukko026.hpc.cs.helsinki.fi:8088/) (replace ukko026 with the name of your namenode).

###PART B: On your datanodes
1. Navigate to your working directory and start Hadoop by using 

        $ ./sbin/hadoop-daemon.sh start datanode
If everything goes smoothly, you will able to find this node after refreshing the webpage in Step A.7.

2. The namenode can also be a datanode. Simply do Step B.1 on your namenode.

3. Start the YARN Node Manager by using 

        $ ./sbin/yarn-daemon.sh start nodemanager
If everything goes smoothly, you will able to find this node after refreshing the webpage in Step A.11, Cluster -> Nodes.

4. The namenode can also be a slave node. Simply do Step B.3 on your namenode.

###Part C: Stop Hadoop when you done
1. Stop your namenode:

       $ ./sbin/hadoop-daemon.sh stop namenode
       $ ./sbin/yarn-daemon.sh stop resourcemanager

2. Stop your datanodes:

       $ ./sbin/hadoop-daemon.sh stop datanode
       $ ./sbin/yarn-daemon.sh stop nodemanager

###Part D: Try out WordCount example
1. Generate wc.jar on your local computer. The source code is available at
[https://hadoop.apache.org/docs/stable/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html](https://hadoop.apache.org/docs/stable/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html)

2. Upload wc.jar (with your SFTP client) to your Hadoop working directory on Ukko.

3. Create input folder and two files:
$ mkdir /input

        $ cat Hello World Bye World > /input/file01
        $ cat Hello Hadoop Bye Hadoop > /input/file02

4. … then put “input” folder into HDFS (on your namenode):

        $ ./bin/hadoop fs –put input /

5. … Run wc.jar (on your namenode):
 
       $ ./bin/hadoop jar wc.jar WordCount /input /output

6. Show result:
 
       $ ./bin/hadoop fs –cat /output/part-r-00000

###Additional instructions
1. The jps command is a shortcut for checking the status of individual service, instead of reading logs. Bear in mind however that it only indicates the process has been started, not running correctly. Always check the log when something is not working as expected.

2. You may also browse the HDFS filesystem in web browser: On the menubar, find “Utilities” > “Browse the file system”.

3. ./bin/hadoop fs –ls / command will show all files/folders under HDFS’s root.

4. ./bin/hadoop fs –rm –r -f /output command will delete “/output” folder in HDFS. NB: Double-check your input before pressing ENTER!

5. More HDFS command is available at [https://hadoop.apache.org/docs/stable/hadoop-projectdist/hadoop-common/FileSystemShell.html](https://hadoop.apache.org/docs/stable/hadoop-projectdist/hadoop-common/FileSystemShell.html).

6. If you choose more than 3 datanodes, you may want to change the number of data replication of HDFS. This can be done by adding the following configuration to etc/hadoop/hdfssite.xml:


    <property>
        <name>dfs.replication</name>
        <value>10</value>
    </property>

7. The default split for input file is 128MB. You may alter it by adding the following configuration to etc/hadoop/hdfs-site.xml (33554432 = 32MB):


    <property>
        <name> dfs.blocksize</name>
        <value>33554432</value>
    </property>

8. Hadoop use only one reducer by default (to verify this, see the number of “part-***” files in the output directory or use YARN console webpage).
If you find that the reduce stage is too slow or simply run out of memory (java.lang.OutOfMemoryError), you may try to change it by adding e.g. job.setNumReduceTasks(64); to your source code. NB: The final output will also be distributed when you use more than one reducer. You may need to combine them.

9. If you have problems on HDFS disk capacity limitation, point the datanode directory to your folder instead of tmp one:


     <property>
         <name>dfs.datanode.data.dir</name>
         <value>file:///cs/home/youIDinUkko/tmp/${hostname}</value>
      </property>
      
10. If you have problems on Yarn disk capacity limtation (normally unhealthy node error), point the nodemanager directory to your folder instead of tmp one:


    <property>
        <name>yarn.nodemanager.local-dirs</name>
        <value>/cs/home/youIDinUkko/nm-local-dir/${hostname}/</value>
    </property>