# Sysbench benchmarking sample
Deploys a single-pod statefulset in k8s cluster. Mysql container and sysbench container runs in a same pod so that they share the same ip.
This config was tested against IPv4 and IPv6 cluster.
I thought of spinning up a single pod running both, but it is against container philosophy. Also mysql 5.7 and sysbench had package conflicts.

## To run Sysbench OLTP tests
Note: Sysbench by default use localhost, which uses socket to communicate with mysql. Specify ::1 ip address to communicate over host:port.

### Prerequisite
```shell
kubectl exec -ti sysbench-0 -c sysbench -- bash
# inside sysbench container
cat > config << EOF
db-driver=mysql
mysql-host=::1
mysql-port=3306
mysql-user=sysbench
mysql-password=sysbench
mysql-db=sysbench
threads=100
```

### 1. Read only
Play with different thread counts such as 1, 30, 50 or 100. Differnet table size and numbers
```shell
sysbench --config-file=config --table_size=25000 --tables=100 oltp_read_only prepare
sysbench --config-file=config --table_size=25000 --tables=100 oltp_read_only --time=180 --threads=30 --report-interval=1
sysbench --config-file=config --table_size=25000 --tables=100 oltp_read_only cleanup
```
### 2. Write only
```shell
sysbench --config-file=config --table_size=100000 --tables=100 oltp_write_only prepare
sysbench --config-file=config --table_size=100000 --tables=100 oltp_write_only run --time=180 --threads=30 --report-interval=5
sysbench --config-file=config --table_size=100000 --tables=100 oltp_write_only cleanup
```
### 3. Read Write
Default read:write ratio is 1:1. Reduced table size to 25k to fit into the pod.
```shell
sysbench --config-file=config --table_size=25000 --tables=50 oltp_read_write prepare
sysbench --config-file=config --table_size=25000 --tables=50 oltp_read_write run --time=180 --threads=30 --report-interval=10 
sysbench --config-file=config --table_size=25000 --tables=50 oltp_read_write cleanup
```
### 4. Point Select
Primary key lookup of data present in main memory. Since the primary key in mysql is stored together with the data, the only thing mysql has to do is to navigate the in-memory B-tree, find the record, and return the result.
Basically, "SELECT c FROM %s WHERE id=?" queries
```shell
sysbench --config-file=config --table-size=100000 --tables=32 oltp_point_select prepare
sysbench --config-file=config --table-size=100000 --tables=32 oltp_point_select prewarm
sysbench --config-file=config --table-size=100000 --tables=32 oltp_point_select run --threads=30 --db-ps-mode=auto --rand-type=uniform
sysbench --config-file=config --table-size=100000 --tables=32 cleanup
```


