# Deploying Cassandra with a StatefulSet
This tutorial shows you how to run Apache Cassandra on Kubernetes.

# Creating a local StorageClass for Cassandra
If you are using a different type of PV, you can skip this process.  
<code>
  kubectl apply -f ./manifests/local-storage-class.yaml
</code>

# Creating a local persistent volume for Cassandra
If you are using a different type of PV, you can skip this process.
In the case of local PV, it must be created for each node.  
<code>
  kubectl apply -f ./manifests/local-persistent-volume.yaml
</code>

# Creating a namespace for Cassandra
<code>
  kubectl create namespace database
</code>

# Creating a service for Cassandra
<code>
  kubectl apply -f ./manifests/cassandra-service.yaml
</code>

# Create the Cassandra StatefulSet from the cassandra-statefulset.yaml file:
<code>
  kubectl apply -f ./manifests/cassandra-statefulset.yaml
</code>

# How to run cassandra-stress benchmark

|                       | Command |
| --------------------- | ------- |
| Populate the database | <code>cassandra-stress write n=40000000 -pop seq=1..40000000 -node IP_ADDR_LIST -port native=30042 jmx=30043 -schema "replication(strategy=NetworkTopologyStrategy, DC1=1)"</code>|
| Mix-Load              | <code>cassandra-stress mixed ratio\(write=1, read=3\) duration=30m -pop seq=1..40000000 -schema keyspace="keyspace1" -node IP_ADDR_LIST -port native=30042 jmx=30043 -rate threads=100</code>|
| Read-Load             | <code>cassandra-stress read duration=30m -pop seq=1..40000000 -schema keyspace="keyspace1" -node IP_ADDR_LIST -port native=30042 jmx=30043 -rate threads=100</code>|
| Write-Load            | <code>cassandra-stress write duration=30m -pop seq=1..40000000 -schema keyspace="keyspace1" -node IP_ADDR_LIST -port native=30042 jmx=30043 -rate threads=100</code>|

If you need more information, please refer to [this paper](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=8284700&tag=1).

If you need to run cqlsh inside the pod, run the following command.

<code>
  kubectl exec -it -n database cassandra-0 -- /bin/bash
</code>

# Cleaning up

1. Run the following commands to delete everything in the Cassandra StatefulSet:  
<code>
  grace=$(kubectl get pod cassandra-0 -o=jsonpath='{.spec.terminationGracePeriodSeconds}') \
  && kubectl delete statefulset -n database -l app=cassandra \
  && echo "Sleeping ${grace} seconds" 1>&2 \
  && sleep $grace \
  && kubectl delete persistentvolumeclaim -n database -l app=cassandra
</code>

2. Run the following command to delete the Service you set up for Cassandra:  
<code>
  kubectl delete service -n database -l app=cassandra
</code>