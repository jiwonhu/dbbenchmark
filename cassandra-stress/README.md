# cassandra-stress benchmarking sample
Deploys a single-pod in k8s cluster. Runs Cassandra container and cassandra-stress container so that they can share the same ip.
It works well in IPv4 cluster, but hits an issue in IPv6 cluster. Cassandra could not fetch its own hostname.

## Run Cassandra stress tests
```shell
kubectl exec -ti cassandra-0 -c cass-stress -- bash
```
### 1. Write 1 million rows
Play with different thread counts.
```shell
cassandra-stress write n=1000000 -rate threads=30
```

### 2. Read
```shell
# Read 200k rows
cassandra-stress read n=200000 -rate threads=30
# Read rows for 3 minutes
cassandra-stress read duration=3m -rate threads=50
# Read 200k rows without a warmup of 50k rows first.
cassandra-stress read n=200000 no-warmup -rate threads=50
```

### 2. Read-write 1 million rows
Read write ratio is 1:1 by default.
```shell
cassandra-stress mixed n=200000 -rate threads=30
```

To clean up, `kubectl delete` the yaml then `kubectl apply`.
