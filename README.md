# hadoop-xml-sum-simple
Intoruction to Hadoop. This is using a modified version of the WordCount example, to accept an XML input.

## Prerequisites
### Installing and configuring single server Hadoop 2.4.1 on Fedora 25 - 2017

#### Install Java and Hadoop
$ sudo yum install -y java-1.8.0-openjdk.x86_64 hadoop-common.noarch hadoop-common-native.x86_64 hadoop-hdfs.noarch hadoop-mapreduce.noarch hadoop-mapreduce-examples.noarch hadoop-yarn.noarch

#### Configure Hadoop
Create the default HDFS directories based on the hadoop system configuration files
$ sudo hdfs-create-dirs

Format the namenode. Namenode contains metadata about the Hadoop filesystem. Hadoop NameNode is the centralized place of an HDFS file system which keeps the directory tree of all files in the file system, and tracks where across the cluster the file data is kept. In short, it keeps the metadata related to datanodes. When we format namenode it deletes the metadata related to data-nodes. Original data in DataNode will not get affected. If you want to format your namenode first stop all hadoop services, then delete the tmp (which contains namenode and datanode) folder in your local file system, then start hadoop service
$ sudo -u hdfs hadoop namenode -format

#### Start the Hadoop services
$ sudo systemctl start hadoop-namenode hadoop-datanode hadoop-nodemanager hadoop-resourcemanager
$ sudo systemctl enable hadoop-namenode hadoop-datanode hadoop-nodemanager hadoop-resourcemanager

Create a home directory in the HDFS file system for each MapReduce user (with the right permissions for the user). It is best to do this on the NameNode. The user can use this execute MapReduce jobs.
$ sudo -u hdfs hadoop fs -mkdir /user
$ sudo -u hdfs hadoop fs -mkdir /user/hughpear
$ sudo -u hdfs hadoop fs -mkdir /user/hughpear/input
$ sudo -u hdfs hadoop fs -chown -R hughpear:hughpear /user/hughpear

Download some sample data and add it to the HDFS filesystem
$ wget https://ia902708.us.archive.org/13/items/thebibleoldandne00010gut/kjv10.txt
$ sudo -u hughpear hadoop fs -put kjv10.txt /user/hughpear/input

Verify the file system is setup correctly
$ sudo -u hdfs hadoop fs -ls -R /

## Running the program

#### Create artifacts directory
$ mkdir bin

#### Compile xmlreader
$ env HADOOP_CLASSPATH=$(hadoop classpath):/usr/lib/jvm/java/lib/tools.jar hadoop com.sun.tools.javac.Main -d ./bin/ ./WordCount.java ./XmlInputFormat.java

#### Create java archive
$ jar -cvf xmlreader.jar -C bin/ .

#### Download some sample data
$ wget http://www.w3schools.com/xml/cd_catalog.xml
$ sudo -u hughpear hadoop fs -put cd_catalog.xml /user/hughpear/input

#### Run the application
$ sudo -u hdfs hadoop fs -rm -r -f /user/hughpear/output*
$ env HADOOP_USER_NAME=hdfs hadoop jar ./xmlreader.jar WordCount /user/hughpear/input/cd_catalog.xml /user/hughpear/output1

#### View the results
$ hadoop fs -ls output1
$ hadoop fs -cat output1/part-r-00000 | less


