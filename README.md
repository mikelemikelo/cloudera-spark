# Environment set up for spark-runtime-service

Add the following line to your laptop's `/etc/hosts` file:
`quickstart.cloudera	127.0.0.1`

For the image to be used, you have two options:
* Build the image by following these steps:
  * git clone [cloudera-spark](https://github.com/mikelemikelo/cloudera-spark.git)
  * `cd cloudera-spark`
  * `docker build -t mikelemikelo/cloudera-spark:latest .`
* Pull down the image from DockerHub
  * `docker pull mikelemikelo/cloudera-spark:latest`
  
To start the Docker image running a Spark cluster, run the following command which will also shell you into the machine:
```docker run --hostname=quickstart.cloudera --privileged=true -ti -p 8180:8080 -p 19888:19888 -p 7077:7077 -p 8188:8088 -p 8032:8032 -p 8020:8020 -p 50010:50010 -p 8042:8042 -p 7180:7180 -p 88:88/udp -p 88:88  mikelemikelo/cloudera-spark:latest /usr/bin/docker-quickstart-light```

Congratulations! You now have running a local Cloudera Spark cluser.

---
### The following ports are now being used:
* Cloudera welcome tutorial - 80
* Oozie - 8888
* Cloudera Manager - 7180
* HDFS REST Api - 8020
* WebHDFS - 50070
* Kerberos - 88 (both TCP and UDP)
* Secure Data Transfer - 1004 / 1006
* Custom Spring Boot Webserver - 8990

---
If you would like to submit jobs to the Spark cluster you just started, please follow [this guide](https://github.com/modelop/spark-runtime-service/blob/master/LOCAL_ENV_SETUP.md) for set up instructions.

If you would like to enable Kerberos for the Spark Cluster you just started, please follow [this guide](https://github.com/modelop/spark-runtime-service/blob/master/keberos_cluster.md) for setup instructions.
