
### Adding Spark to local machine and connecting it to the running cluster.

- Find `LAPTOP_USERNAME`

from LOCAL terminal try
`#whoami`

This will return your `LAPTOP_USERNAME`


- Copy the spark-2.4.4 into HOST machine:

*(Make sure to enable remote access if you are on MAC before moving forward)*


```
scp /opt/spark-2.4.4-bin-hadoop2.6.tgz LAPTOP_USERNAME@host.docker.internal:/Users/LAPTOP_USERNAME/DEST_FOLDER/
scp -r /etc/hadoop/conf/ LAPTOP_USERNAME@host.docker.internal:/Users/LAPTOP_USERNAME/path/to/etc/hadoop/conf
```


- Unzip the tarball that you scp’d over

```
tar -xzvf spark-2.4.4-bin-hadoop2.6.tgz
```

Set the following environment variables. Make sure you change LAPTOP_USERNAME accordingly

```
export SPARK_HOME=/Users/LAPTOP_USERNAME/spark-2.4.4-bin-hadoop2.6
```

From the cloudera container, copy the files located at /etc/hadoop/conf and store them locally at <path/to/etc/hadoop/conf>. Update hdfs-site.xml file to include

```
<property>
    <name>dfs.client.use.datanode.hostname</name>
    <value>true</value>
</property>
```

Set the following environment variable. Make sure you change LAPTOP_USERNAME accordingly

```
export HADOOP_CONF_DIR=/Users/LAPTOP_USERNAME/path/to/etc/hadoop/conf
```

#### In case your laptop does not have Java 8 and Python3.6

Your laptop needs Java 8 since that’s the version Spark needs. If you need to install Java, you should be able to  have it alongside whatever version of Java you might be running. JAVA_HOME needs to be pointing to Java 8

```
brew tap AdoptOpenJDK/openjdk
brew cask install adoptopenjdk8
export JAVA_HOME=/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home
```
Your laptop needs Python 3.6 since that’s the version Spark needs. If you’re running Python 3.8 like I was, then this one is a little tough. I installed pyenv, then 3.6.9, then made a little shim to point to 3.6.9 which I called pythonme and set an environment variable accordingly.

```
brew install pyenv
pyenv install 3.6.9
ln -s ~/.pyenv/versions/3.6.9/bin/python3.6 /usr/local/bin/pythonme
export PYSPARK_PYTHON=/usr/local/bin/pythonme
```

Lets try it all together:

1- From the spark folder:

```
./bin/spark-submit --master yarn --executor-memory 512MB --total-executor-cores 10 ./examples/src/main/python/pi.py
```

2 - If everything went fine, you should expect something like:

```
Pi is roughly 3.138920
```

