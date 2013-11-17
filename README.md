# Accumulo Pig Integration

The official support site for accumulo-pig is at http://people.apache.org/~elserj/accumulo-pig/. I'm fooling around
Jason's code inside this project. Please checkout the official project.

2013-Nov-17: I added an example (see below) showing that you can read from more than one Accumulo table but that 
the UNION of those aliases does not work.

2013-Nov-16: I updated Jason's code to work with Accumulo 1.5.0. It is fairly trivial to experiment with 
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
createtable person
insert pp101 attribute height 60
insert pp101 attribute weight 123
insert pp102 attribute height 65
insert pp102 attribute weight 435
insert pp102 text "" "Noman killed the cyclops."
createtable house
insert hh101 attribute baths 1.5
insert hh101 attribute rooms 8
insert hh101 attribute height 180
insert hh102 attribute baths 3
insert hh102 attribute rooms 9
insert hh102 attribute height 156
insert hh102 text "" "Windage waits for no man."
exit
```

## Starting Pig

```
pig
register /home/vagrant/accumulo_home/bin/accumulo/lib/accumulo-core.jar
register /home/vagrant/accumulo_home/bin/accumulo/lib/accumulo-fate.jar
register /home/vagrant/accumulo_home/bin/accumulo/lib/accumulo-trace.jar
register /home/vagrant/accumulo_home/bin/accumulo/lib/libthrift.jar
register /home/vagrant/accumulo_home/bin/zookeeper/zookeeper-3.4.5.jar
register /vagrant/accumulo-pig/target/accumulo-pig-1.4.0.jar
```

## Use Pig to Read Accumulo

```
PERSON = LOAD 'accumulo://person?instance=instance&user=root&password=secret&zookeepers=affy-master:2181&columns=attribute' using org.apache.accumulo.pig.AccumuloStorage() AS (row, cf, cq, cv, ts, val);
B = FOREACH PERSON GENERATE row, cq, val;
SORTED_BY_VALUE = ORDER B BY val DESC;
dump SORTED_BY_VALUE;
```

## Reading from more than one Accumulo table

```
PERSON = LOAD 'accumulo://person?instance=instance&user=root&password=secret&zookeepers=affy-master:2181&columns=attribute' using org.apache.accumulo.pig.AccumuloStorage() AS (row, cf, cq, cv, ts, val);
HOUSE = LOAD 'accumulo://house?instance=instance&user=root&password=secret&zookeepers=affy-master:2181&columns=attribute' using org.apache.accumulo.pig.AccumuloStorage() AS (row, cf, cq, cv, ts, val);

PERSON_ROWS = FOREACH PERSON GENERATE row, cq, val;
PERSON_HEIGHTS_ROWS = FILTER PERSON_ROWS BY cq == 'height';
PERSON_HEIGHTS = FOREACH PERSON_HEIGHTS_ROWS GENERATE val, row;

HOUSE_ROWS = FOREACH HOUSE GENERATE row, cq, val;
HOUSE_HEIGHTS_ROWS = FILTER HOUSE_ROWS BY cq == 'height';
HOUSE_HEIGHTS = FOREACH HOUSE_HEIGHTS_ROWS GENERATE val, row;
```

You can display the person heights:

```
dump PERSON_HEIGHTS
(60,pp101)
(65,pp102)
```

And you can display the house heights:
```
dump HOUSE_HEIGHTS
(180,hh101)
(156,hh102)
(180,hh101)
(156,hh102)
```

But you can't union the two aliases:

```
HEIGHTS = UNION PERSON_HEIGHTS, HOUSE_HEIGHTS;
dump HEIGHTS;
(60,pp101)
(65,pp102)
(60,pp101)
(65,pp102)
```
