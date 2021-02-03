### Environment set up for running spark-submit locally

* If you use a Mac, find your `LAPTOP_USERNAME` by running:
    * `whoami`
* If you use a Mac, enable remote access by following [this guide](https://osxdaily.com/2011/09/30/remote-login-ssh-server-mac-os-x/)
* From the Cloudera container, run the following commands to copy the two folders locally (remember to update LAPTOP_USERNAME accordingly):
   ```
    scp /opt/spark-2.4.4-bin-hadoop2.6.tgz LAPTOP_USERNAME@host.docker.internal:/Users/LAPTOP_USERNAME/spark-2.4.4-bin-hadoop2.6.tgz
    scp -r /etc/hadoop/conf/ LAPTOP_USERNAME@host.docker.internal:/Users/LAPTOP_USERNAME/path/to/etc/hadoop/conf
   ```
* From your local terminal, run the following commands:
    * `tar -xzvf spark-2.4.4-bin-hadoop2.6.tgz`
    * `export SPARK_HOME=/Users/LAPTOP_USERNAME/spark-2.4.4-bin-hadoop2.6`
    * Update `hadoop/conf/hdfs-site.xml` to include the following property:
       ```
        <property>
            <name>dfs.client.use.datanode.hostname</name>
            <value>true</value>
        </property>
        ```
    * `export HADOOP_CONF_DIR=/Users/LAPTOP_USERNAME/path/to/etc/hadoop/conf`
    
---
### Installing Java 8 and/or Python 3.6 locally

Your laptop needs Java 8 since that is the version Spark needs. If you need to install Java, you should be able to have it alongside whatever version of Java you might be running. `JAVA_HOME` needs to be pointing to Java 8.
```
brew tap AdoptOpenJDK/openjdk
brew cask install adoptopenjdk8
export JAVA_HOME=/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home
```

Your laptop needs Python 3.6 since that’s the version Spark needs. If you’re running Python 3.8, then this one is a little tough. You can install `pyenv`, `3.6.9`, then make a little shim to point to 3.6.9 which I called pythonme and set an environment variable accordingly.
```
brew install pyenv
pyenv install 3.6.9
ln -s ~/.pyenv/versions/3.6.9/bin/python3.6 /usr/local/bin/pythonme
export PYSPARK_PYTHON=/usr/local/bin/pythonme
```

---
### Running spark-submit with a PySpark model

In order to run your first PySpark model, run the following command from your local terminal:
```
$SPARK_HOME/bin/spark-submit --master yarn --executor-memory 512MB --total-executor-cores 10 $SPARK_HOME/examples/src/main/python/pi.py
```
If everything went fine, you should expect to see this line in the logs: `Pi is roughly 3.138920`
