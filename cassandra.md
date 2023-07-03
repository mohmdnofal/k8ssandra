https://vmc.techzone.vmware.com/resource/running-k8ssandra-vmware-tanzu-kubernetes-grid-vmware-cloud-aws#workload-testing
https://medium.com/building-the-open-data-stack/set-up-multi-datacenter-cassandra-clusters-in-gke-with-k8ssandra-and-cloud-dns-5cb389b1e776


- we must use Azure CNI as pods should be routable 

AKS_NAME=cassandra
LOCATION1=northeurope #
LOCATION2=westeurope
LOCATION3=uksouth
LOCATION4=switzerlandnorth
RG1=$AKS_NAME-$LOCATION1
RG2=$AKS_NAME-$LOCATION2
RG3=$AKS_NAME-$LOCATION3
RG4=$AKS_NAME-$LOCATION4
AKS_VNET_NAME_1=$AKS_NAME-vnet-$LOCATION1
AKS_VNET_NAME_2=$AKS_NAME-vnet-$LOCATION2
AKS_VNET_NAME_3=$AKS_NAME-vnet-$LOCATION3
AKS_VNET_NAME_4=$AKS_NAME-vnet-$LOCATION4
AKS_CLUSTER_NAME1=$AKS_NAME-cluster-$LOCATION1 # name of the cluster
AKS_CLUSTER_NAME2=$AKS_NAME-cluster-$LOCATION2 # name of the cluster
AKS_CLUSTER_NAME3=$AKS_NAME-cluster-$LOCATION3 # name of the cluster
AKS_CLUSTER_NAME4=$AKS_NAME-cluster-$LOCATION4 # name of the cluster
AKS_VNET_CIDR1=10.1.0.0/22
AKS_VNET_CIDR2=10.2.0.0/22
AKS_VNET_CIDR3=10.3.0.0/22
AKS_VNET_CIDR4=10.4.0.0/22
AKS_NODES_SUBNET_NAME1=$AKS_NAME-nodes-subnet-$LOCATION1 # the AKS nodes subnet
AKS_NODES_SUBNET_NAME2=$AKS_NAME-nodes-subnet-$LOCATION2 # the AKS nodes subnet
AKS_NODES_SUBNET_NAME3=$AKS_NAME-nodes-subnet-$LOCATION3 # the AKS nodes subnet
AKS_NODES_SUBNET_NAME4=$AKS_NAME-nodes-subnet-$LOCATION4 # the AKS nodes subnet
AKS_PODS_SUBNET_NAME1=$AKS_NAME-subnet-$LOCATION1 # the AKS pods subnet
AKS_PODS_SUBNET_NAME2=$AKS_NAME-subnet-$LOCATION2 # the AKS pods subnet
AKS_PODS_SUBNET_NAME3=$AKS_NAME-subnet-$LOCATION3 # the AKS pods subnet
AKS_PODS_SUBNET_NAME4=$AKS_NAME-subnet-$LOCATION4 # the AKS pods subnet
AKS_NODES_SUBNET_PREFIX1=10.1.0.0/24
AKS_PODS_SUBNET_PREFIX1=10.1.1.0/24
AKS_NODES_SUBNET_PREFIX2=10.2.0.0/24
AKS_PODS_SUBNET_PREFIX2=10.2.1.0/24
AKS_NODES_SUBNET_PREFIX3=10.3.0.0/24
AKS_PODS_SUBNET_PREFIX3=10.3.1.0/24
AKS_NODES_SUBNET_PREFIX4=10.4.0.0/24
AKS_PODS_SUBNET_PREFIX4=10.4.1.0/24
NETWORK_PLUGIN=azure
NETWORK_POLICY=calico
SYSTEM_NODE_COUNT=2
K8S_VERSION=$(az aks get-versions  -l $LOCATION1 --query 'orchestrators[-1].orchestratorVersion' -o tsv)
SYSTEM_NODE_COUNT=3  # system node pool size (single pool with 3 nodes across AZs)
USER_NODE_COUNT=2 # 3 node pools with 2 nodes each 
USER_NODES_SKU=Standard_D4ds_v5 # node VM type 
SYSTEM_POOL_NAME=systempool
CASSANDRA_POOL_ZONE1_NAME=cassandraz1
CASSANDRA_POOL_ZONE2_NAME=cassandraz2
CASSANDRA_POOL_ZONE3_NAME=cassandraz3


#Create 4 resource groups 

```
az group create --name $RG1 --location $LOCATION1
az group create --name $RG2 --location $LOCATION2
az group create --name $RG3 --location $LOCATION3
az group create --name $RG4 --location $LOCATION4
```

#create vnets and subnets 
```
az network vnet create -g $RG1 --location $LOCATION1 --name $AKS_VNET_NAME_1 --address-prefixes $AKS_VNET_CIDR1 -o none 
az network vnet subnet create -g $RG1 --vnet-name $AKS_VNET_NAME_1 --name $AKS_NODES_SUBNET_NAME1 --address-prefixes $AKS_NODES_SUBNET_PREFIX1 -o none 
az network vnet subnet create -g $RG1 --vnet-name $AKS_VNET_NAME_1 --name $AKS_PODS_SUBNET_NAME1 --address-prefixes $AKS_PODS_SUBNET_PREFIX1 -o none 


az network vnet create -g $RG2 --location $LOCATION2 --name $AKS_VNET_NAME_2 --address-prefixes $AKS_VNET_CIDR2 -o none 
az network vnet subnet create -g $RG2 --vnet-name $AKS_VNET_NAME_2 --name $AKS_NODES_SUBNET_NAME2 --address-prefixes $AKS_NODES_SUBNET_PREFIX2 -o none 
az network vnet subnet create -g $RG2 --vnet-name $AKS_VNET_NAME_2 --name $AKS_PODS_SUBNET_NAME2 --address-prefixes $AKS_PODS_SUBNET_PREFIX2 -o none 


az network vnet create -g $RG3 --location $LOCATION3 --name $AKS_VNET_NAME_3 --address-prefixes $AKS_VNET_CIDR3 -o none 
az network vnet subnet create -g $RG3 --vnet-name $AKS_VNET_NAME_3 --name $AKS_NODES_SUBNET_NAME3 --address-prefixes $AKS_NODES_SUBNET_PREFIX3 -o none 
az network vnet subnet create -g $RG3 --vnet-name $AKS_VNET_NAME_3 --name $AKS_PODS_SUBNET_NAME3 --address-prefixes $AKS_PODS_SUBNET_PREFIX3 -o none 


az network vnet create -g $RG4 --location $LOCATION4 --name $AKS_VNET_NAME_4 --address-prefixes $AKS_VNET_CIDR4 -o none 
az network vnet subnet create -g $RG4 --vnet-name $AKS_VNET_NAME_4 --name $AKS_NODES_SUBNET_NAME4 --address-prefixes $AKS_NODES_SUBNET_PREFIX4 -o none 
az network vnet subnet create -g $RG4 --vnet-name $AKS_VNET_NAME_4 --name $AKS_PODS_SUBNET_NAME4 --address-prefixes $AKS_PODS_SUBNET_PREFIX4 -o none 

```

#Peer all the VNETS 

```
#get the ids 
AKS_VNET_ID_1=$(az network vnet show --resource-group $RG1 --name $AKS_VNET_NAME_1 --query id --out tsv)
AKS_VNET_ID_2=$(az network vnet show --resource-group $RG2 --name $AKS_VNET_NAME_2 --query id --out tsv)
AKS_VNET_ID_3=$(az network vnet show --resource-group $RG3 --name $AKS_VNET_NAME_3 --query id --out tsv)
AKS_VNET_ID_4=$(az network vnet show --resource-group $RG4 --name $AKS_VNET_NAME_4 --query id --out tsv)

#Peer VNET1 to all 
az network vnet peering create --name vnet1-to-vnet2 --resource-group $RG1 --vnet-name $AKS_VNET_NAME_1 --remote-vnet $AKS_VNET_ID_2 --allow-vnet-access
az network vnet peering create --name vnet1-to-vnet3 --resource-group $RG1 --vnet-name $AKS_VNET_NAME_1 --remote-vnet $AKS_VNET_ID_3 --allow-vnet-access
az network vnet peering create --name vnet1-to-vnet4 --resource-group $RG1 --vnet-name $AKS_VNET_NAME_1 --remote-vnet $AKS_VNET_ID_4 --allow-vnet-access

#Peer VNET2 to all 
az network vnet peering create --name vnet2-to-vnet1 --resource-group $RG2 --vnet-name $AKS_VNET_NAME_2 --remote-vnet $AKS_VNET_ID_1 --allow-vnet-access
az network vnet peering create --name vnet2-to-vnet3 --resource-group $RG2 --vnet-name $AKS_VNET_NAME_2 --remote-vnet $AKS_VNET_ID_3 --allow-vnet-access
az network vnet peering create --name vnet2-to-vnet4 --resource-group $RG2 --vnet-name $AKS_VNET_NAME_2 --remote-vnet $AKS_VNET_ID_4 --allow-vnet-access

#Peer VNET3 to all 
az network vnet peering create --name vnet3-to-vnet1 --resource-group $RG3 --vnet-name $AKS_VNET_NAME_3 --remote-vnet $AKS_VNET_ID_1 --allow-vnet-access
az network vnet peering create --name vnet3-to-vnet2 --resource-group $RG3 --vnet-name $AKS_VNET_NAME_3 --remote-vnet $AKS_VNET_ID_2 --allow-vnet-access
az network vnet peering create --name vnet3-to-vnet4 --resource-group $RG3 --vnet-name $AKS_VNET_NAME_3 --remote-vnet $AKS_VNET_ID_4 --allow-vnet-access


#Peer VNET4 to all 
az network vnet peering create --name vnet4-to-vnet1 --resource-group $RG4 --vnet-name $AKS_VNET_NAME_4 --remote-vnet $AKS_VNET_ID_1 --allow-vnet-access
az network vnet peering create --name vnet4-to-vnet2 --resource-group $RG4 --vnet-name $AKS_VNET_NAME_4 --remote-vnet $AKS_VNET_ID_2 --allow-vnet-access
az network vnet peering create --name vnet4-to-vnet3 --resource-group $RG4 --vnet-name $AKS_VNET_NAME_4 --remote-vnet $AKS_VNET_ID_3 --allow-vnet-access

```

#create the AKS clusters 
```

#create AKS cluster1 
#get the subnet ids 
AKS_VNET_NODE_SUBNET_ID_1=$(az network vnet subnet show --name $AKS_NODES_SUBNET_NAME1 -g $RG1 --vnet-name $AKS_VNET_NAME_1 --query "id" -o tsv)
AKS_VNET_PODS_SUBNET_ID_1=$(az network vnet subnet show --name $AKS_PODS_SUBNET_NAME1 -g $RG1 --vnet-name $AKS_VNET_NAME_1 --query "id" -o tsv)


az aks create \
-g $RG1 \
-n $AKS_CLUSTER_NAME1 \
-l $LOCATION1 \
--node-count $SYSTEM_NODE_COUNT \
--network-plugin $NETWORK_PLUGIN \
--kubernetes-version $K8S_VERSION \
--generate-ssh-keys \
--vnet-subnet-id $AKS_VNET_NODE_SUBNET_ID_1 \
--pod-subnet-id $AKS_VNET_PODS_SUBNET_ID_1 \
--enable-addons monitoring \
--nodepool-name $SYSTEM_POOL_NAME \
--uptime-sla \
--zones 1 2 3 

#add 3 additional node pools for Cassandra 
##First Node Pool in Zone 1
az aks nodepool add \
--cluster-name $AKS_CLUSTER_NAME1 \
--mode User \
--name $CASSANDRA_POOL_ZONE1_NAME \
--node-vm-size $USER_NODES_SKU \
--resource-group $RG1 \
--zones 1 \
--vnet-subnet-id $AKS_VNET_NODE_SUBNET_ID_1 \
--pod-subnet-id $AKS_VNET_PODS_SUBNET_ID_1 \
--enable-cluster-autoscaler \
--max-count 4 \
--min-count 2 \
--node-count $USER_NODE_COUNT \
--node-taints app=cassandra:NoSchedule \
--labels dept=dev purpose=cassandra \
--tags dept=dev costcenter=1000 \
--no-wait

##Second Node Pool in Zone 2
az aks nodepool add \
--cluster-name $AKS_CLUSTER_NAME1 \
--mode User \
--name $CASSANDRA_POOL_ZONE2_NAME \
--node-vm-size $USER_NODES_SKU \
--resource-group $RG1 \
--zones 2 \
--vnet-subnet-id $AKS_VNET_NODE_SUBNET_ID_1 \
--pod-subnet-id $AKS_VNET_PODS_SUBNET_ID_1 \
--enable-cluster-autoscaler \
--max-count 4 \
--min-count 2 \
--node-count $USER_NODE_COUNT \
--node-taints app=cassandra:NoSchedule \
--labels dept=dev purpose=cassandra \
--tags dept=dev costcenter=1000 \
--no-wait



##Third Node Pool in Zone 3
az aks nodepool add \
--cluster-name $AKS_CLUSTER_NAME1 \
--mode User \
--name $CASSANDRA_POOL_ZONE3_NAME \
--node-vm-size $USER_NODES_SKU \
--resource-group $RG1 \
--zones 3 \
--vnet-subnet-id $AKS_VNET_NODE_SUBNET_ID_1 \
--pod-subnet-id $AKS_VNET_PODS_SUBNET_ID_1 \
--enable-cluster-autoscaler \
--max-count 4 \
--min-count 2 \
--node-count $USER_NODE_COUNT \
--node-taints app=cassandra:NoSchedule \
--labels dept=dev purpose=cassandra \
--tags dept=dev costcenter=1000 \
--no-wait

#get the credintials 
az aks get-credentials -n $AKS_CLUSTER_NAME1 -g $RG1
kubectl get nodes -l dept=dev                                      
kubectl describe nodes -l dept=dev | grep -i topology.kubernetes.io/zone

                    topology.kubernetes.io/zone=northeurope-1
                    topology.kubernetes.io/zone=northeurope-1
                    topology.kubernetes.io/zone=northeurope-2
                    topology.kubernetes.io/zone=northeurope-2
                    topology.kubernetes.io/zone=northeurope-3
                    topology.kubernetes.io/zone=northeurope-3

```
#create cluster 2 
```
#get the subnet ids 
AKS_VNET_NODE_SUBNET_ID_2=$(az network vnet subnet show --name $AKS_NODES_SUBNET_NAME2 -g $RG2 --vnet-name $AKS_VNET_NAME_2 --query "id" -o tsv)
AKS_VNET_PODS_SUBNET_ID_2=$(az network vnet subnet show --name $AKS_PODS_SUBNET_NAME2 -g $RG2 --vnet-name $AKS_VNET_NAME_2 --query "id" -o tsv)


az aks create \
-g $RG2 \
-n $AKS_CLUSTER_NAME2 \
-l $LOCATION2 \
--node-count $SYSTEM_NODE_COUNT \
--network-plugin $NETWORK_PLUGIN \
--kubernetes-version $K8S_VERSION \
--generate-ssh-keys \
--vnet-subnet-id $AKS_VNET_NODE_SUBNET_ID_2 \
--pod-subnet-id $AKS_VNET_PODS_SUBNET_ID_2 \
--enable-addons monitoring \
--nodepool-name $SYSTEM_POOL_NAME \
--uptime-sla \
--zones 1 2 3 

#add 3 additional node pools for Cassandra 
##First Node Pool in Zone 1
az aks nodepool add \
--cluster-name $AKS_CLUSTER_NAME2 \
--mode User \
--name $CASSANDRA_POOL_ZONE1_NAME \
--node-vm-size $USER_NODES_SKU \
--resource-group $RG2 \
--zones 1 \
--vnet-subnet-id $AKS_VNET_NODE_SUBNET_ID_2 \
--pod-subnet-id $AKS_VNET_PODS_SUBNET_ID_2 \
--enable-cluster-autoscaler \
--max-count 4 \
--min-count 2 \
--node-count $USER_NODE_COUNT \
--node-taints app=cassandra:NoSchedule \
--labels dept=dev purpose=cassandra \
--tags dept=dev costcenter=1000 \
--no-wait

##Second Node Pool in Zone 2
az aks nodepool add \
--cluster-name $AKS_CLUSTER_NAME2 \
--mode User \
--name $CASSANDRA_POOL_ZONE2_NAME \
--node-vm-size $USER_NODES_SKU \
--resource-group $RG2 \
--zones 2 \
--vnet-subnet-id $AKS_VNET_NODE_SUBNET_ID_2 \
--pod-subnet-id $AKS_VNET_PODS_SUBNET_ID_2 \
--enable-cluster-autoscaler \
--max-count 4 \
--min-count 2 \
--node-count $USER_NODE_COUNT \
--node-taints app=cassandra:NoSchedule \
--labels dept=dev purpose=cassandra \
--tags dept=dev costcenter=1000 \
--no-wait



##Third Node Pool in Zone 3
az aks nodepool add \
--cluster-name $AKS_CLUSTER_NAME2 \
--mode User \
--name $CASSANDRA_POOL_ZONE3_NAME \
--node-vm-size $USER_NODES_SKU \
--resource-group $RG2 \
--zones 3 \
--vnet-subnet-id $AKS_VNET_NODE_SUBNET_ID_2 \
--pod-subnet-id $AKS_VNET_PODS_SUBNET_ID_2 \
--enable-cluster-autoscaler \
--max-count 4 \
--min-count 2 \
--node-count $USER_NODE_COUNT \
--node-taints app=cassandra:NoSchedule \
--labels dept=dev purpose=cassandra \
--tags dept=dev costcenter=1000 \
--no-wait

#get the credintials 
az aks get-credentials -n $AKS_CLUSTER_NAME2 -g $RG2
kubectl get nodes -l dept=dev                                      
kubectl describe nodes -l dept=dev | grep -i topology.kubernetes.io/zone


```


#create cluster 3
```
#get the subnet ids 
AKS_VNET_NODE_SUBNET_ID_3=$(az network vnet subnet show --name $AKS_NODES_SUBNET_NAME3 -g $RG3 --vnet-name $AKS_VNET_NAME_3 --query "id" -o tsv)
AKS_VNET_PODS_SUBNET_ID_3=$(az network vnet subnet show --name $AKS_PODS_SUBNET_NAME3 -g $RG3 --vnet-name $AKS_VNET_NAME_3 --query "id" -o tsv)


az aks create \
-g $RG3 \
-n $AKS_CLUSTER_NAME3 \
-l $LOCATION3 \
--node-count $SYSTEM_NODE_COUNT \
--network-plugin $NETWORK_PLUGIN \
--kubernetes-version $K8S_VERSION \
--generate-ssh-keys \
--vnet-subnet-id $AKS_VNET_NODE_SUBNET_ID_3 \
--pod-subnet-id $AKS_VNET_PODS_SUBNET_ID_3 \
--enable-addons monitoring \
--nodepool-name $SYSTEM_POOL_NAME \
--uptime-sla \
--zones 1 2 3 \
--yes \
--no-wait 

#add 3 additional node pools for Cassandra 
##First Node Pool in Zone 1
az aks nodepool add \
--cluster-name $AKS_CLUSTER_NAME3 \
--mode User \
--name $CASSANDRA_POOL_ZONE1_NAME \
--node-vm-size $USER_NODES_SKU \
--resource-group $RG3 \
--zones 1 \
--vnet-subnet-id $AKS_VNET_NODE_SUBNET_ID_3 \
--pod-subnet-id $AKS_VNET_PODS_SUBNET_ID_3 \
--enable-cluster-autoscaler \
--max-count 4 \
--min-count 2 \
--node-count $USER_NODE_COUNT \
--node-taints app=cassandra:NoSchedule \
--labels dept=dev purpose=cassandra \
--tags dept=dev costcenter=1000 \
--no-wait

##Second Node Pool in Zone 2
az aks nodepool add \
--cluster-name $AKS_CLUSTER_NAME3 \
--mode User \
--name $CASSANDRA_POOL_ZONE2_NAME \
--node-vm-size $USER_NODES_SKU \
--resource-group $RG3 \
--zones 2 \
--vnet-subnet-id $AKS_VNET_NODE_SUBNET_ID_3 \
--pod-subnet-id $AKS_VNET_PODS_SUBNET_ID_3 \
--enable-cluster-autoscaler \
--max-count 4 \
--min-count 2 \
--node-count $USER_NODE_COUNT \
--node-taints app=cassandra:NoSchedule \
--labels dept=dev purpose=cassandra \
--tags dept=dev costcenter=1000 \
--no-wait



##Third Node Pool in Zone 3
az aks nodepool add \
--cluster-name $AKS_CLUSTER_NAME3 \
--mode User \
--name $CASSANDRA_POOL_ZONE3_NAME \
--node-vm-size $USER_NODES_SKU \
--resource-group $RG3 \
--zones 3 \
--vnet-subnet-id $AKS_VNET_NODE_SUBNET_ID_3 \
--pod-subnet-id $AKS_VNET_PODS_SUBNET_ID_3 \
--enable-cluster-autoscaler \
--max-count 4 \
--min-count 2 \
--node-count $USER_NODE_COUNT \
--node-taints app=cassandra:NoSchedule \
--labels dept=dev purpose=cassandra \
--tags dept=dev costcenter=1000 \
--no-wait

#get the credintials 
az aks get-credentials -n $AKS_CLUSTER_NAME3 -g $RG3
kubectl get nodes -l dept=dev                                      
kubectl describe nodes -l dept=dev | grep -i topology.kubernetes.io/zone


```


#create cluster 4
```
#get the subnet ids 
AKS_VNET_NODE_SUBNET_ID_4=$(az network vnet subnet show --name $AKS_NODES_SUBNET_NAME4 -g $RG4 --vnet-name $AKS_VNET_NAME_4 --query "id" -o tsv)
AKS_VNET_PODS_SUBNET_ID_4=$(az network vnet subnet show --name $AKS_PODS_SUBNET_NAME4 -g $RG4 --vnet-name $AKS_VNET_NAME_4 --query "id" -o tsv)


az aks create \
-g $RG4 \
-n $AKS_CLUSTER_NAME4 \
-l $LOCATION4 \
--node-count $SYSTEM_NODE_COUNT \
--network-plugin $NETWORK_PLUGIN \
--kubernetes-version $K8S_VERSION \
--generate-ssh-keys \
--vnet-subnet-id $AKS_VNET_NODE_SUBNET_ID_4 \
--pod-subnet-id $AKS_VNET_PODS_SUBNET_ID_4 \
--node-resource-group MC_cassandra_$LOCATION4
--enable-addons monitoring \
--nodepool-name $SYSTEM_POOL_NAME \
--uptime-sla \
--zones 1 2 3 \
--yes \
--no-wait 

#add 3 additional node pools for Cassandra 
##First Node Pool in Zone 1
az aks nodepool add \
--cluster-name $AKS_CLUSTER_NAME4 \
--mode User \
--name $CASSANDRA_POOL_ZONE1_NAME \
--node-vm-size $USER_NODES_SKU \
--resource-group $RG4 \
--zones 1 \
--vnet-subnet-id $AKS_VNET_NODE_SUBNET_ID_4 \
--pod-subnet-id $AKS_VNET_PODS_SUBNET_ID_4 \
--enable-cluster-autoscaler \
--max-count 4 \
--min-count 2 \
--node-count $USER_NODE_COUNT \
--node-taints app=cassandra:NoSchedule \
--labels dept=dev purpose=cassandra \
--tags dept=dev costcenter=1000 \
--no-wait

##Second Node Pool in Zone 2
az aks nodepool add \
--cluster-name $AKS_CLUSTER_NAME4 \
--mode User \
--name $CASSANDRA_POOL_ZONE2_NAME \
--node-vm-size $USER_NODES_SKU \
--resource-group $RG4 \
--zones 2 \
--vnet-subnet-id $AKS_VNET_NODE_SUBNET_ID_4 \
--pod-subnet-id $AKS_VNET_PODS_SUBNET_ID_4 \
--enable-cluster-autoscaler \
--max-count 4 \
--min-count 2 \
--node-count $USER_NODE_COUNT \
--node-taints app=cassandra:NoSchedule \
--labels dept=dev purpose=cassandra \
--tags dept=dev costcenter=1000 \
--no-wait



##Third Node Pool in Zone 3
az aks nodepool add \
--cluster-name $AKS_CLUSTER_NAME4 \
--mode User \
--name $CASSANDRA_POOL_ZONE3_NAME \
--node-vm-size $USER_NODES_SKU \
--resource-group $RG4 \
--zones 3 \
--vnet-subnet-id $AKS_VNET_NODE_SUBNET_ID_4 \
--pod-subnet-id $AKS_VNET_PODS_SUBNET_ID_4 \
--enable-cluster-autoscaler \
--max-count 4 \
--min-count 2 \
--node-count $USER_NODE_COUNT \
--node-taints app=cassandra:NoSchedule \
--labels dept=dev purpose=cassandra \
--tags dept=dev costcenter=1000 \
--no-wait

#get the credintials 
az aks get-credentials -n $AKS_CLUSTER_NAME4 -g $RG4
kubectl get nodes -l dept=dev                                      
kubectl describe nodes -l dept=dev | grep -i topology.kubernetes.io/zone


```

#install cert-manager on each cluster
```
helm repo add jetstack https://charts.jetstack.io
helm repo update

kubectx $AKS_CLUSTER_NAME1
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true

kubectx $AKS_CLUSTER_NAME2
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true

kubectx $AKS_CLUSTER_NAME3
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true

kubectx $AKS_CLUSTER_NAME4
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true

#to verify on any cluster 
kubectl get pods -n cert-manager 
NAME                                       READY   STATUS    RESTARTS   AGE
cert-manager-64f9f45d6f-jgcs8              1/1     Running   0          117s
cert-manager-cainjector-56bbdd5c47-rkn59   1/1     Running   0          117s
cert-manager-webhook-d4f4545d7-plz8g       1/1     Running   0          117s

```

#install the k8ssandra operator 
```
helm repo add k8ssandra https://helm.k8ssandra.io/stable
helm repo update

#install the control plane (we will install the control plane in cluster 4)

kubectx $AKS_CLUSTER_NAME4
helm install k8ssandra-operator k8ssandra/k8ssandra-operator -n k8ssandra-operator --create-namespace

```

#install the data planes on Cluster 1,2, and 3 
```
helm show values k8ssandra/k8ssandra-operator > values_sample.yaml

kubectx $AKS_CLUSTER_NAME1
helm install k8ssandra-operator k8ssandra/k8ssandra-operator -n k8ssandra-operator --create-namespace --set controlPlane=false

kubectx $AKS_CLUSTER_NAME2
helm install k8ssandra-operator k8ssandra/k8ssandra-operator -n k8ssandra-operator --create-namespace --set controlPlane=false

kubectx $AKS_CLUSTER_NAME3
helm install k8ssandra-operator k8ssandra/k8ssandra-operator -n k8ssandra-operator --create-namespace --set controlPlane=false

kubectl -n k8ssandra-operator get all
```

#downside, Cassandra Operator resides in Docker Hub, probably you want to mirror the image to your own registry to avoid throttling 
#downside, no resiliency is configured as part of the operator 



#create client config resources 
wget https://raw.githubusercontent.com/k8ssandra/k8ssandra-operator/main/scripts/create-clientconfig.sh
chmod +x create-clientconfig.sh

#configure contexts in the control plane cluster (4)

./create-clientconfig.sh --src-context $AKS_CLUSTER_NAME1 --dest-context $AKS_CLUSTER_NAME4 --namespace k8ssandra-operator
./create-clientconfig.sh --src-context $AKS_CLUSTER_NAME2 --dest-context $AKS_CLUSTER_NAME4 --namespace k8ssandra-operator
./create-clientconfig.sh --src-context $AKS_CLUSTER_NAME3 --dest-context $AKS_CLUSTER_NAME4 --namespace k8ssandra-operator


#deploy cassandra cluster 

kubectx $AKS_CLUSTER_NAME4

cat <<EOF | kubectl apply -n k8ssandra-operator  -f -
apiVersion: k8ssandra.io/v1alpha1
kind: K8ssandraCluster
metadata:
  name: demo
spec:
  cassandra:
    serverVersion: 4.0.3
    storageConfig:
      cassandraDataVolumeClaimSpec:
        storageClassName: managed-premium
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 5Gi
    config:
      jvmOptions:
        heapSize: 512M
    networking:
      hostNetwork: false
    datacenters:
      - metadata:
           name: dc1-northeurope
        k8sContext: cassandra-cluster-northeurope
        size: 3
        stargate:
          size: 3
          heapSize: 256M
          allowStargateOnDataNodes: true          
          tolerations:
          - key: "app"
            operator: "Equal"
            value: "cassandra"
            effect: "NoSchedule"          
        tolerations:
          - key: "app"
            operator: "Equal"
            value: "cassandra"
            effect: "NoSchedule"
        racks:
          - name: northeurope-1
            nodeAffinityLabels:
                topology.kubernetes.io/zone: northeurope-1
                agentpool: cassandraz1
          - name: northeurope-2
            nodeAffinityLabels:
                topology.kubernetes.io/zone: northeurope-2
                agentpool: cassandraz2
          - name: northeurope-3
            nodeAffinityLabels:
                topology.kubernetes.io/zone: northeurope-3
                agentpool: cassandraz3                        
      - metadata:
           name: dc2-westeurope
        k8sContext: cassandra-cluster-westeurope
        size: 3
        stargate:
          size: 3
          heapSize: 256M
          allowStargateOnDataNodes: true
          tolerations:
          - key: "app"
            operator: "Equal"
            value: "cassandra"
            effect: "NoSchedule"          
        racks:
          - name: westeurope-1
            nodeAffinityLabels:
                topology.kubernetes.io/zone: westeurope-1
                agentpool: cassandraz1
          - name: westeurope-2
            nodeAffinityLabels:
                topology.kubernetes.io/zone: westeurope-2
                agentpool: cassandraz2
          - name: westeurope-3
            nodeAffinityLabels:
                topology.kubernetes.io/zone: westeurope-3
                agentpool: cassandraz3                    
      - metadata:
           name: dc3-uksouth
        k8sContext: cassandra-cluster-uksouth
        size: 3
        stargate:
          size: 3
          heapSize: 256M
          allowStargateOnDataNodes: true
          tolerations:
          - key: "app"
            operator: "Equal"
            value: "cassandra"
            effect: "NoSchedule"          
        racks:
          - name: uksouth-1
            nodeAffinityLabels:
                topology.kubernetes.io/zone: uksouth-1
                agentpool: cassandraz1
          - name: uksouth-2
            nodeAffinityLabels:
                topology.kubernetes.io/zone: uksouth-2
                agentpool: cassandraz2
          - name: northeurope-3
            nodeAffinityLabels:
                topology.kubernetes.io/zone: uksouth-3
                agentpool: cassandraz3                    
EOF


kubectl apply -n k8ssandra-operator  -f cassandra.yaml
#validate 

kubectx $AKS_CLUSTER_NAME4
kubectl describe k8ssandracluster demo -n k8ssandra-operator

kubectx $AKS_CLUSTER_NAME1
kubectl -n k8ssandra-operator get pods
kubectl -n k8ssandra-operator get svc 
kubectl -n k8ssandra-operator get secrets
kubectl -n k8ssandra-operator get pods -o wide
kubectl get servicemonitor -n k8ssandra-operator


CASS_USERNAME=$(kubectl get secret demo-superuser -n k8ssandra-operator -o=jsonpath='{.data.username}' | base64 --decode)
CASS_PASSWORD=$(kubectl get secret demo-superuser -n k8ssandra-operator -o=jsonpath='{.data.password}' | base64 --decode)
kubectl exec --stdin --tty demo-dc1-northeurope-northeurope-1-sts-0 -n k8ssandra-operator -c cassandra -- nodetool -u $CASS_USERNAME -pw $CASS_PASSWORD status
kubectl exec --stdin --tty demo-dc1-northeurope-northeurope-1-sts-0 -n k8ssandra-operator -c cassandra -- nodetool -u $CASS_USERNAME -pw $CASS_PASSWORD info
kubectl exec --stdin --tty demo-dc1-northeurope-northeurope-1-sts-0 -n k8ssandra-operator -c cassandra -- nodetool -u $CASS_USERNAME -pw $CASS_PASSWORD ring



#deploy Prometheus 
kubectx $AKS_CLUSTER_NAME1
git clone https://github.com/prometheus-operator/kube-prometheus.git
cd kube-prometheus
kubectl apply --server-side -f manifests/setup
kubectl wait \
	--for condition=Established \
	--all CustomResourceDefinition \
	--namespace=monitoring
kubectl apply -f manifests/

kubectl get pods -n monitoring -w
kubectl get servicemonitor -n k8ssandra-operator


# Enable kube prometheous to monito your namespace 

namespace=k8ssandra-operator


cat <<EOF | kubectl apply -f - 
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: prometheus-k8s
  namespace: ${namespace}
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: prometheus-k8s
subjects:
- kind: ServiceAccount
  name: prometheus-k8s
  namespace: monitoring

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: prometheus-k8s
  namespace: ${namespace}
rules:
- apiGroups:
  - ""
  resources:
  - services
  - endpoints
  - pods
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - extensions
  resources:
  - ingresses
  verbs:
  - get
  - list
  - watch
EOF



kubectl -n k8ssandra-operator apply -f https://docs-v2.k8ssandra.io/tasks/monitor/grafana-dashboards.yaml

kubectl -n monitoring port-forward svc/grafana 3000:3000
kubectl port-forward -n monitoring services/prometheus-k8s 10000:9090
kubectl -n monitoring apply -f grafana-dashboards.yaml





#to uninstall 
kubectl delete --ignore-not-found=true -f manifests/ -f manifests/setup




# Remove taints temporarily

az aks nodepool update -n cassandraz1 -g $RG1 --cluster-name $AKS_CLUSTER_NAME1 --node-taints ""
az aks nodepool update -n cassandraz2 -g $RG1 --cluster-name $AKS_CLUSTER_NAME1 --node-taints ""
az aks nodepool update -n cassandraz3 -g $RG1 --cluster-name $AKS_CLUSTER_NAME1 --node-taints ""

az aks nodepool update -n cassandraz1 -g $RG2 --cluster-name $AKS_CLUSTER_NAME2 --node-taints ""
az aks nodepool update -n cassandraz2 -g $RG2 --cluster-name $AKS_CLUSTER_NAME2 --node-taints ""
az aks nodepool update -n cassandraz3 -g $RG2 --cluster-name $AKS_CLUSTER_NAME2 --node-taints ""

az aks nodepool update -n cassandraz1 -g $RG3 --cluster-name $AKS_CLUSTER_NAME3 --node-taints ""
az aks nodepool update -n cassandraz2 -g $RG3 --cluster-name $AKS_CLUSTER_NAME3 --node-taints ""
az aks nodepool update -n cassandraz3 -g $RG3 --cluster-name $AKS_CLUSTER_NAME3 --node-taints ""




# Perform some tests 

kubectx $AKS_CLUSTER_NAME1
kubectl get services -n k8ssandra-operator demo-dc1-northeurope-stargate-service

kubectl port-forward -n k8ssandra-operator svc/demo-dc1-northeurope-stargate-service 8080 8081 8082 9042 &
97846

#generate stargate access token 
curl -L -X POST 'http://localhost:8081/v1/auth' -H 'Content-Type: application/json' --data-raw '{"username": "demo-superuser", "password": "gfL4bmG1VQswN7viMdqZ"}'


