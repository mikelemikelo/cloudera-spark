# cloudera-spark

1- `cd` into `cloudera-spark` root folder. 
2- Build the image with.

`# docker build -t mikelemikelo/cloudera-spark:latest . `

3- Once done, execute this every time you would like to have the image running:

```
# docker run --hostname=quickstart.cloudera --privileged=true -ti -p 7077:7077 -p 8188:8088 -p 8032:8032 -p 8020:8020 -p 50010:50010 -p 8042:8042 -p 7180:7180 -p 88:88/udp -p 88:88  mikelemikelo/cloudera-spark:latest /usr/bin/docker-quickstart-light 
```


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
