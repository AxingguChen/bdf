**Apache Spark Installation.**

**Download and extract**
Download stable version of spark. 
https://spark.apache.org/downloads.html

Current stable version : Spark 2.1.1 (release on 2 May 2017)

        $tar xvzf  spark-2.1.1-bin-hadoop2.7.tgz
   
Run a toy example:

        $./bin/run-example SparkPi 10
       
Run shell:
        
        #Scala
        $./bin/spark-shell --master local[2]
        #Python
        $./bin/pyspark --master local[2]

**Spark on yarn**
History server:


        #File: spark-defaults.conf
        spark.eventLog.enabled           	true
        spark.eventLog.dir               		hdfs://ukko143.hpc.cs.helsinki.fi:9005/spark
        spark.history.fs.logDirectory	 	hdfs://ukko143.hpc.cs.helsinki.fi:9005/spark

Spark on yarn:
 
        
        #File: spark-defaults.conf
        spark.yarn.historyServer.address 	ukko143.hpc.cs.helsinki.fi:8088
        #File: spark-env.sh
        HADOOP_CONF_DIR=/yuxingch/hadoop-2.7.3/etc/hadoop/

Spark submit (toy case on 2.1.1):


        $./bin/spark-submit --class org.apache.spark.examples.SparkPi \
            --master yarn \
            --deploy-mode cluster \
            --driver-memory 2g \
            --executor-memory 2g \
            --executor-cores 1 \
            --queue yuxingch\
            examples/jars/spark-examples*.jar \
            10

Spark kmeans example from me:


        $./bin/spark-submit  
            --class com.intel.hibench.sparkbench.ml.DenseKMeans 
            --master yarn-client 
            --num-executors 6 
            --executor-cores 5 
            --executor-memory 5g 
            /yuxingch/benchmark/spark/HiBench-master/sparkbench/assembly/target/sparkbench-assembly-6.1-SNAPSHOT-dist.jar 
            -k 10 --numIterations 5 
            hdfs://ukko143.hpc.cs.helsinki.fi:9005/HiBench/Kmeans/Input/samples/part-00000,
            hdfs://ukko143.hpc.cs.helsinki.fi:9005/HiBench/Kmeans/Input/samples/part-00001
