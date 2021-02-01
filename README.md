# cloudera-spark

Pre-requisites:

Add `quickstart.cloudera	127.0.0.1` into host machine `/etc/hosts`

### Local Spark with cluster

- Copy the spark-2.4.4 into HOST machine:

- from terminal try:
`whoami`

Make sure you change LAPTOP_USERNAME accordingly (Make sure to enable remote access if you are on MAC ) , whoami  - should return LAPTOP_USERNAME

```
scp /opt/spark-2.4.4-bin-hadoop2.6.tgz LAPTOP_USERNAME@host.docker.internal:/Users/LAPTOP_USERNAME/DEST_FOLDER/
scp -r /etc/hadoop/conf/ LAPTOP_USERNAME@host.docker.internal:/Users/LAPTOP_USERNAME/path/to/etc/hadoop/conf
```
A laptop configured to point to the Spark container

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


---

### Cloudera Spark image 

1- `cd` into `cloudera-spark` root folder. 

2- Build the image with.

`# docker build -t mikelemikelo/cloudera-spark:latest . `

3- Once done, execute this every time you would like to have the image running:


```
# docker run --hostname=quickstart.cloudera --privileged=true -ti -p 7077:7077 -p 8188:8088 -p 8032:8032 -p 8020:8020 -p 50010:50010 -p 8042:8042 -p 7180:7180 -p 88:88/udp -p 88:88  mikelemikelo/cloudera-spark:latest /usr/bin/docker-quickstart-light 
```

Ports being used:

*cloudera welcome tutorial - 80
*oozie - 8888
*cloudera manager - 7180
*hdfs REST api- 8020
*webhdfs - 50070
*kerberos - 88 (both tcp and udp)
*secure data transfer - 1004 / 1006
*custom spring boot webserver - 8990


---

### Kerberos 

Once the cluster is up and running, from within the POD, execute:

1- Start cloudera manager:
```
# sudo /home/cloudera/cloudera-manager --express --force;  sudo service cloudera-scm-server restart
```

2- Install Kerberos:
```
# /home/cloudera/kerberos
```

* here you don’t have to do anything except remember the output of the script where they provide crucial information for later installation

*You will get this output:*

Success! Kerberos is now running. You can enable Kerberos in a Cloudera Manager cluster from the drop-down menu for that cluster on the CM home page. It will ask you to confirm that this script performed the following steps:

```
* set up a working KDC.
* checked that the KDC allows renewable tickets.
* installed the client libraries.
* created a proper account for Cloudera Manager.
```

Then, it will prompt you for the following details (accept defaults if not specified here):

```
KDC Type:                MIT KDC
KDC Server Host:         quickstart.cloudera
Kerberos Security Realm: CLOUDERA
```

Later, it will prompt you for KDC account manager credentials:
```
Username: cloudera-scm/admin (@ CLOUDERA)
Password: cloudera
```

___

### Enable Kerberos in Cloudera Manager

Open in your browser: localhost:7180

To login into Cloudera Manager use:
```
username = cloudera
password = cloudera
```

Now go to **Administration** tab and choose **Security**.  
Then click **Enable Kerberos**.

1. Check all 4 of the boxes as they were all created in the previous step.

2. In the next step:  
	1) Enter the	KDC Server Host from the script (quickstart.cloudera)  
	2) Change the Kerberos Security Realm to the one provided in the script  (CLOUDERA) 

3. In the next step:  
check the ***Manage krb5.conf through Cloudera Manager*** Box and Continue. 


4. In the next step:  
enter the Username and Password from the script (cloudera-scm/admin and cloudera)


5. In the next step:  
wait for  **Import KDC Account Manager Credentials Command** to finish and Continue.

6. In the last step choose to Restart the cluster.  


	**If the last step of Restart hangs, try to refresh the page, if there is no response, you may need to restart the Cloudera Quickstart Manager manually using this command (it will take 2-3 minutes to finish)**  
			
		sudo service cloudera-scm-server restart
		
7. You can check that both Kerberos and Cloudera Manager are working by checking ports 88 and 7180 using this command:
		
		sudo netstat -tulpn | grep 88  
	
	and 

		sudo netstat -tulpn | grep 7180

8. Lastly you just have to restart all of the services in Cloudera Manager.  
Or atleast only restart Zookeeper, YARN and HDFS (in that order). 

9. Now if you try the command 

		hdfs dfs -ls /
		
You should be blocked by Kerberos raising a PrivilegedActionException.

---

### Adding a Kerberos user

You have to be using the "root" user to do the following.  

Enter the Kerberos interactive shell type 

	kadmin.local
		
Now add a principal with name "test" by typing this command and then providing a password for this user  

	addprinc hdfs@CLOUDERA
		
Please remember the pricipal's password as your clients will login using that.  
To check that the principal was created, run: 

	list_principals

You can test the login with that user by exiting the kadmin.local shell and typing: 

	kinit hdfs@CLOUDERA 
	
Then if you type "klist" you will see that there is a ticker using which you are logged in.
You can also delete that ticked using "kdestroy" command and then you can re-login.  

- 
You can also export the keytab of a principal using  

	xst -k hdfs.keytab hdfs@CLOUDERA
	
and then copy the keytab outside of the container into your PC, eg. 

	docker cp [container_id]:/hdfs.keytab /Users/YOUR_LOCAL_USER/Desktop/

Now you can finally login as test@CLOUDERA using Kerberos at your local (client) machine by using  

	kinit -k -t /Users/YOUR_LOCAL_USER/Desktop/hdfs.keytab hdfs@CLOUDERA

Now we need to add a HDFS folder for that user.

---
