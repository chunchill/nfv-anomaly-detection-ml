# The process for building the dataset
1. Login on the Jump Server
```
sshpass -p 123456 ssh -p 9000 root@100.64.249.217
```
2. Run clearwater Workload
```
# In the Jump Server
## check the clearwater pods status
kubectl get pods

## delete clearwater pods
kubectl delete -f cw

## deploy clearwater pods
kubectl apply -f cw

# create clearwater users
## Enter into node24
./connect.sh 24

## Enter into the container clearwater-cassandra
docker exec -it $(docker ps |grep clearwater-cassandra |awk {'print $1'}) bash

## Bulk Create users（e.g.：20000 users）
/usr/share/clearwater/crest-prov/src/metaswitch/crest/tools/stress_provision.sh 20000

# stress testing
## enter into node32
./connect.sh 32

## enter into the clearwater-sprout container
docker exec -it $(docker ps |grep clearwater-sprout |awk {'print $1'} | head -1) bash

## install the stress tools
apt-get update
apt-get install clearwater-sip-stress-coreonly -y

## start the stress test（20000 users，100 mins，clearwater-sprout container ip:10.42.56.48）
/usr/share/clearwater/bin/run_stress default.svc.cluster.local \
            20000 100  --initial-reg-rate 100 \
            --icscf-target 10.42.56.48:5052 \
            --scscf-target 10.42.56.48:5054
```
3. randomly inject bottlenecks
```
# for 100min
./stress.py 100
```
4. collecting data
http://100.64.249.217:12002/swagger-ui.html
5. clean the dataset
