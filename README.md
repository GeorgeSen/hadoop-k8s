# Deploy PV
```shell
cd pv
#见README
kubectl apply -f ./
kubectl apply -f ./example
kubectl get pv
```

# Deploy the HDFS

```
cd k8s
kubectl apply -f hdfs-service.yaml
kubectl apply -f hdfs-namenode.yaml 
kubectl apply -f hdfs-datanode.yaml 
```

# Verify the HDFS cluster

You can visit the HDFS DashBoard with this address: http://localhost:30070/
You can enter into container by `kubectl exec -it  pod/hdfs-datanode-0 -- /bin/bash` and use `hdfs dfs -ls /` to list HDFS files.

# Scale manual
```shell
kubectl scale sts hdfs-datanode --replicas=4
```

# Balance the HDFS

```
kubectl apply -f hdfs-balance.yaml
```

# Prometheus Export

You can visit the HDFS DashBoard with this address: http://localhost:31318/

# Create a client 

```
kubectl apply -f hdfs-client.yaml 
```

