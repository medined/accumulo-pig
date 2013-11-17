
# Accumulo Pig Integration

The official support site for accumulo-pig is at http://people.apache.org/~elserj/accumulo-pig/. I'm fooling around
Jason's code inside this project. Please checkout the official project.

2013-Nov-16 - I updated Jason's code to work with Accumulo 1.5.0. It is fairly trivial to experiment with 
Pig and Accumulo using my Vagrant boxes. 

## Start an Accumulo Cluster

Start an Accumulo cluster using https://github.com/medined/Accumulo_1_5_0_By_Vagrant.
 
## Install Pig on the Accumulo Master

```
vagrant ssh master
cd /home/vagrant/accumulo_home/bin
tar xvfz /vagrant/files/pig-0.12.0.tar.gz
export PATH=/home/vagrant/accumulo_home/bin/pig-0.12.0/bin:$PATH
```

## Download and Compile Accumulo-Pig

```
cd /home/vagrant/accumulo_home/software
git clone https://github.com/medined/accumulo-pig
cd accumulo-pig
mvn package
```

## Create some data in Accumulo

```
accumulo shell -u root -p secret
createtable people
insert id101 attribute height 45
insert id101 attribute weight 123
insert id102 attribute height 23
insert id102 attribute weight 435
insert id102 text "" "Now is the time for all ..."
exit
```

## Use Pig to Read Accumulo

```
pig
register /home/vagrant/accumulo_home/bin/accumulo/lib/accumulo-core.jar
register /home/vagrant/accumulo_home/bin/accumulo/lib/accumulo-fate.jar
register /home/vagrant/accumulo_home/bin/accumulo/lib/accumulo-trace.jar
register /home/vagrant/accumulo_home/bin/accumulo/lib/libthrift.jar
register /home/vagrant/accumulo_home/bin/zookeeper/zookeeper-3.4.5.jar
register /vagrant/accumulo-pig/target/accumulo-pig-1.4.0.jar
DATA = LOAD 'accumulo://people?instance=instance&user=root&password=secret&zookeepers=affy-master:2181&columns=attribute' using org.apache.accumulo.pig.AccumuloStorage() AS (row, cf, cq, cv, ts, val);
HEIGHTS = FOREACH DATA GENERATE row, cq, val;
SORTED_SET = ORDER HEIGHTS BY val DESC;
dump SORTED_SET;
```






build the JAR (Note, you will need to download the accumulo src, build it, and install it into your maven repo before this will work)

    mvn package

download the JARs needed by pig

    mvn dependency:copy-dependencies -DoutputDirectory=lib  -DincludeArtifactIds=zookeeper,libthrift,accumulo-core,cloudtrace

print the register statements we will need in pig

    for JAR in lib/*.jar target/accumulo-pig-1.4.0.jar ; 
    do 
        echo register `pwd`/$JAR; 
    done

Example output

    register /home/developer/workspace/accumulo-pig/lib/accumulo-core-1.4.0.jar
    register /home/developer/workspace/accumulo-pig/lib/cloudtrace-1.4.0.jar
    register /home/developer/workspace/accumulo-pig/lib/libthrift-0.6.1.jar
    register /home/developer/workspace/accumulo-pig/lib/zookeeper-3.3.1.jar
    register /home/developer/workspace/accumulo-pig/target/accumulo-pig-1.4.0.jar

Run Pig, copy the register statements above and paste them into the pig terminal.  Then you can LOAD from and STORE into accumulo.

    $ pig
    2012-03-02 08:15:25,808 [main] INFO  org.apache.pig.Main - Logging error messages to: /home/developer/workspace/accumulo-pig/pig_1330694125807.log
    2012-03-02 08:15:25,937 [main] INFO  org.apache.pig.backend.hadoop.executionengine.HExecutionEngine - Connecting to hadoop file system at: hdfs://127.0.0.1/
    2012-03-02 08:15:26,032 [main] INFO  org.apache.pig.backend.hadoop.executionengine.HExecutionEngine - Connecting to map-reduce job tracker at: 127.0.0.1:9001
    grunt> register /home/developer/workspace/accumulo-pig/lib/accumulo-core-1.4.0.jar
    grunt> register /home/developer/workspace/accumulo-pig/lib/cloudtrace-1.4.0.jar
    grunt> register /home/developer/workspace/accumulo-pig/lib/libthrift-0.6.1.jar
    grunt> register /home/developer/workspace/accumulo-pig/lib/zookeeper-3.3.1.jar
    grunt> register /home/developer/workspace/accumulo-pig/target/accumulo-pig-1.4.0.jar
    grunt> 
    grunt> DATA = LOAD 'accumulo://webpage?instance=inst&user=root&password=secret&zookeepers=127.0.0.1:2181&columns=f:cnt' 
    >>    using org.apache.accumulo.pig.AccumuloStorage() AS (row, cf, cq, cv, ts, val);
    grunt> 
    grunt> DATA2 = FOREACH DATA GENERATE row, cf, cq, cv, val;
    grunt> 
    grunt> STORE DATA2 into 'accumulo://webpage_content?instance=inst&user=root&password=secret&zookeepers=127.0.0.1:2181' using org.apache.accumulo.pig.AccumuloStorage();
    2012-03-02 08:18:44,090 [main] INFO  org.apache.pig.tools.pigstats.ScriptState - Pig features used in the script: UNKNOWN
    2012-03-02 08:18:44,093 [main] INFO  org.apache.pig.newplan.logical.rules.ColumnPruneVisitor - Columns pruned for DATA: $4
    2012-03-02 08:18:44,108 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MRCompiler - File concatenation threshold: 100 optimistic? false
    2012-03-02 08:18:44,110 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MultiQueryOptimizer - MR plan size before optimization: 1
    2012-03-02 08:18:44,110 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MultiQueryOptimizer - MR plan size after optimization: 1
    2012-03-02 08:18:44,117 [main] INFO  org.apache.pig.tools.pigstats.ScriptState - Pig script settings are added to the job
    2012-03-02 08:18:44,118 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.JobControlCompiler - mapred.job.reduce.markreset.buffer.percent is not set, set to default 0.3
    2012-03-02 08:18:44,120 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.JobControlCompiler - creating jar file Job7611629033341757288.jar
    2012-03-02 08:18:46,282 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.JobControlCompiler - jar file Job7611629033341757288.jar created
    2012-03-02 08:18:46,286 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.JobControlCompiler - Setting up single store job
    2012-03-02 08:18:46,375 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MapReduceLauncher - 1 map-reduce job(s) waiting for submission.
    2012-03-02 08:18:46,876 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MapReduceLauncher - 0% complete
    2012-03-02 08:18:46,878 [Thread-17] INFO  org.apache.pig.backend.hadoop.executionengine.util.MapRedUtil - Total input paths (combined) to process : 1
    2012-03-02 08:18:47,887 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MapReduceLauncher - HadoopJobId: job_201203020643_0001
    2012-03-02 08:18:47,887 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MapReduceLauncher - More information at: http://127.0.0.1:50030/jobdetails.jsp?jobid=job_201203020643_0001
    2012-03-02 08:18:54,434 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MapReduceLauncher - 50% complete
    2012-03-02 08:18:57,484 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MapReduceLauncher - 100% complete
    2012-03-02 08:18:57,485 [main] INFO  org.apache.pig.tools.pigstats.SimplePigStats - Script Statistics: 

    HadoopVersion    PigVersion    UserId    StartedAt    FinishedAt    Features
    0.20.2    0.9.2    developer    2012-03-02 08:18:44    2012-03-02 08:18:57    UNKNOWN

    Success!

    Job Stats (time in seconds):
    JobId    Maps    Reduces    MaxMapTime    MinMapTIme    AvgMapTime    MaxReduceTime    MinReduceTime    AvgReduceTime    Alias    Feature    Outputs
    job_201203020643_0001    1    0    3    3    3    0    0    0    DATA,DATA2    MAP_ONLY    accumulo://webpage_content?instance=inst&user=root&password=secret&zookeepers=127.0.0.1:2181,

    Input(s):
    Successfully read 288 records from: "accumulo://webpage?instance=inst&user=root&password=secret&zookeepers=127.0.0.1:2181&columns=f:cnt"

    Output(s):
    Successfully stored 288 records in: "accumulo://webpage_content?instance=inst&user=root&password=secret&zookeepers=127.0.0.1:2181"

    Counters:
    Total records written : 288
    Total bytes written : 0
    Spillable Memory Manager spill count : 0
    Total bags proactively spilled: 0
    Total records proactively spilled: 0

    Job DAG:
    job_201203020643_0001


    2012-03-02 08:18:57,492 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MapReduceLauncher - Success!
    grunt> 


