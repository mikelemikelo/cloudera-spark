# cloudera-spark

Pre-requisites:

Add `quickstart.cloudera	127.0.0.1` into host machine `/etc/hosts`


---

### Building Cloudera Spark cluster image

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


Congrats! At this point you have a running local Cloudera Spark cluster.

---

## 2 PART - KERBEROS ENV

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

http://quickstart.cloudera:7180/

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
		
Now add a principal with name "hdfs" by typing this command and then providing a password for this user  

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

	docker cp [container_id]:/etc/krb5.conf /etc/
	docker cp [container_id]:/hdfs.keytab /etc/

Now you can finally login as test@CLOUDERA using Kerberos at your local (client) machine by using  

	kinit -k -t /etc/hdfs.keytab hdfs@CLOUDERA

Now we need to add a HDFS folder for that user.

---

In order to test your Spark cluster with the updates, update your local HADOOP_CONF_FILES with the updated cluster ones after Kerberos.


