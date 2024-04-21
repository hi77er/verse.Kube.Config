### Demo project accompanying a [Consul crash course video](https://www.youtube.com/watch?v=s3I1kKKfjtQ) on YouTube


AWS commands to execute the script

#### Get access to EKS cluster
```sh
# install and configure awscli with access creds
aws configure

# check existing clusters list
aws eks list-clusters --region eu-central-1 --output table --query 'clusters'

# check config of specific cluster - VPC config shows whether public access enabled on cluster API endpoint
aws eks describe-cluster --region eu-central-1 --name verse-eks-cluster --query 'cluster.resourcesVpcConfig'

# create kubeconfig file for cluster in ~/.kube
aws eks update-kubeconfig --region eu-central-1 --name verse-eks-cluster

# test configuration
kubectl get svc
```

#### Install & Configure Consul
```sh
# add hashicorp repository
helm repo add hashicorp http://helm.releases.hashicorp.com

# install and configure consul using helm charts
# --values consul-values.yaml - here we reconfigure the consul intalltion
helm install eks hashicorp/consul --version 1.0.0 --values consul-values.yaml --set global.datacenter=eks

# deleting the current microservices installation in order to replace it with the one that has consul configured - config-consul.yaml
kubectl delete -f config.yaml

# apply the consul-aware configuration config-consul.yaml
kubectl apply -f config-consul.yaml
```

#### Peer to another consul cluster
```sh
# 1. after makingsure mesh-gateway is enabled in consul-values.yaml, apply the consul-mesh-gateway.yaml 
# to each of the kubernetes clusters that will be peered
# 2. then go to "peers" page in the consul ui for each of the clusters and peer them creating a toenin 
# one of the cluters and referencing it in the other one
kubectl apply -f consul-mesh-gateway.yaml
```

#### Configure a service failover via exported service and a service resolver.
```sh
# the exported-service.yaml makes the described service visible from the peers mentioned in the yaml file
kubectl apply -f exported-service.yaml

# the service-resolver.yaml configures the actual failover within the cluster where 
# the second service was exporded to
kubectl apply -f service-resolver.yaml
```
