### Enable Kerberos for your Cloudera Spark cluster

From the Cloudera Spark container, run the following commands:
* `sudo /home/cloudera/cloudera-manager --express --force`
* `sudo service cloudera-scm-server restart`
* `/home/cloudera/kerberos`
	*  After this command completes, you will see the following information which you will use later in the installation process:
```
KDC Type:                MIT KDC
KDC Server Host:         quickstart.cloudera
Kerberos Security Realm: CLOUDERA
```

Next, you need to enable Kerberos from the Cloudera Manager UI:
* Open `http://quickstart.cloudera:7180/`
* Use the following credentials:
```
username = cloudera
password = cloudera
```
* Go to the `Administration` tab, click `Security`, and click `Enable Kerberos`
* Check all four boxes as there requirements have already been satisfied when you ran `/home/cloudera/kerberos`
* For the next step:
	* Enter the KDC Server Host: `quickstart.cloudera`
	* Change the Kerberos Security Realm: `CLOUDERA`
	* Continue
* For the next step:
	* Check `Manage krb5.conf through Cloudera Manager`
	* Continue
* For the next step, enter the following information:
	* Username: `cloudera-scm/admin`
	* Password: `cloudera`
	* Continue
* Wait for `Import KDC Account Manager Credentials Command` to finish
* In the last step, choose to restart the cluster
	* If the last step of Restart hangs, try to refresh the page. If there is no response, you may need to restart the Cloudera Quickstart Manager manually using this command (it will take 2-3 minutes to finish):
		* `sudo service cloudera-scm-server restart`
* You can check that both Kerberos and Cloudera Manager are working by checking ports 88 and 7180 using these commands:
	* `sudo netstat -tulpn | grep 88`
	* `sudo netstat -tulpn | grep 7180`
* Finally, you have to restart all the services from the Cloudera Manager. Or you can just restart the following service in this order: Zookeeper, YARN, and HDFS.
* Now run this command from the Cloudera Spark container, you should be blocked by Kerberos raising a PrivilegedActionException:
	* `hdfs dfs -ls /`

		hdfs dfs -ls /
		
---
### Add a new Kerberos principal/user
You have to be using the "root" user to do the following

* Enter the Kerberos interactive shell
	* `kadmin.local`		
* Add a principal with name "hdfs" by running this command and providing a password for this user when prompted
	* addprinc hdfs@CLOUDERA
Please remember the pricipal's password as your clients will login using it 
* To check that the principal was created, run: 
	* list_principals
* You can test the login with that user by running
	* `exit`
	* `kinit hdfs@CLOUDERA`
	
* If you type "klist" you will see that there is a ticker using which you are logged in. You can also delete that ticker using "kdestroy" command and then you can re-login.  
* You can export the keytab file of the hdfs principal by running  
	* `xst -k hdfs.keytab hdfs@CLOUDERA`
* You can copy the keytab file outside of the container to your laptop by running 
	* `sudo docker cp [container_id]:/etc/krb5.conf /etc/`
	* `sudo docker cp [container_id]:/hdfs.keytab /etc/`

* Finally, you can login as hdfs@CLOUDERA using Kerberos at your local terminal by running  
	* `kinit -k -t /etc/hdfs.keytab hdfs@CLOUDERA`

Next, you need to create an HDFS folder for your new user.

---
In order to test your Cloudera Spark cluster with these updates (Kerberos enabled), download from the Cloudera Spark cluster the Hadoop files and store them in a new directory. Then, update the HADOOP_CONF_DIR environment variable to point to your new directory containing the latest Hadoop files. [This guide](https://github.com/modelop/spark-runtime-service/blob/master/LOCAL_ENV_SETUP.md) contains instructions on downloading the Hadoop files.
