# k8s-microservices
A simple example of a microservices architecture using kubernetes.

It is a shipping tracker, composed by:
- Webapp ```(Angular and Spring)```
- Message Queue ```(ActiveMQ)```
- Position Tracker ```(Java)```
- Position Simulator ```(Java)```
- API Gateway ```(Java)```
- Persistent Storage ```(MongoDB)```
- Logging ```(Fluentd)```
- Log Storing ```(ElasticSearch)```
- Log Visualization ```(Kibana)```
- Monitoring ```(Prometheus)```
- Monitoring Visualization ```(Grafana)```
- Alerts ```(AlertManager and Slack)```

All the services/resources above run inside a kubernetes cluster.

All the files used to config the microservices system can be found inside the **example** folder.

## Summary

- [ How to run locally ](#how-to-run-locally)
- [ How to run on production ](#how-to-run-on-production)
- [ How it works ](#how-it-works)
- [ How to apply changes ](#how-to-apply-changes)
- [ How to see status ](#how-to-see-status)
- [ Zero down time deployment ](#zero-down-time-deployment)
- [ How replica set works ](#how-replica-set-works)
- [ Service discovery and communication between pods ](#service-discovery-and-communication-between-pods)
- [ How persistent volume works ](#how-persistent-volume-works)
- [ How to get into EC2 Instance ](#how-to-get-into-ec2-instance)
- [ Using nano editor with KOPS ](#using-nano-editor-with-kops)
- [ How to install Prometheus and Grafana on EC2 Instance ](#how-to-install-prometheus-and-grafana-on-ec2-instance)
- [ How to use the AlertManager with Slack ](#how-to-use-the-alertmanager-with-slack)
- [ Helping Prometheus to verify ETCD service ](#helping-prometheus-to-verify-etcd-service)
- [ How to Auto-scale Pods ](#how-to-autoscale-pods)
- [ Readiness and Liveness Probe ](#readiness-and-liveness-probe)
- [ Useful commands ](#useful-commands)

<a name="how-to-run-locally"></a>

## How to run locally
You'll need to install **Minikube**, **Docker**, **VirtualBox** and **Kubectl**.

After installing the needed apps, do the following:

1. Start minikube
```sh
minikube start --vm-driver=virtualbox --memory 4096

# After using this command you can use only minikube start to make it to work next time
```

2. Link your Docker CLI with minikube
```sh
eval $(minikube docker-env) # Automatically gets docker configured for you 
```

3. Bootstrap all the microservices and resources in your kubernetes local cluster
```sh
kubectl apply -f ./example/dev/
```

<a name="how-to-run-on-production"></a>

## How to run on production
You'll need a **EC2 Instance**, **IAM Credentials**, **S3 Bucket**, **Kops** and **Kubectl**.

Follow the steps bellow to get your kubernetes cluster ready on AWS:

1. Create a EC2 Instance

2. Get into the instance

3. Install [Kops](https://github.com/kubernetes/kops) and the [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

4. Create a IAM User with ```AmazonEC2FullAccess, AmazonRoute53FullAccess, AmazonS3FullAccess, IAMFullAccess, AmazonVPCFullAccess``` permissions

5. Configure the IAM User on the EC2 Instance

6. Create a S3 Bucket to use as cluster storage

7. Config the environmental variables (name and bucket) on the EC2 Instance

8. Create the cluster configuration

9. Customize the cluster configuration (optional)

10. Start the cluster

11. Start the resources using ```kubectl apply``` on .yaml files from ```/example/prod```

The steps **5, 7, 8, 9, 10, 11** are covered on [Getting Started - KOPS](https://github.com/kubernetes/kops/blob/master/docs/getting_started/aws.md#creating-your-first-cluster).

<a name="how-it-works"></a>

## How it works
Basically in Kubernetes we use the concept of Pod (that's like VM which you can run multiple containers inside).

If we want to make this Pod to be visible on the internet, we need to use a Service to expose it.

Example:
```sh
# Pod that runs a Angular Application with the help of a pre-made docker image
./webapp-pod.yaml

# Service that exposes the Angular Application to internet
./webapp-service.yaml
```

<a name="how-to-apply-changes"></a>

## How to apply changes
```sh
kubectl apply -f $FILE_NAME

# Examples
# kubectl apply -f webapp-pod.yaml
# kubectl apply -f webapp-service.yaml
```

<a name="how-to-see-status"></a>

## How to see status
```sh
kubectl get all
```

<a name="zero-down-time-deployment"></a>

## Zero down time deployment
The basic concept is:
1. You have a running Pod [1] and Service.
2. You deploy a new Pod [2] with a new label.
3. The new Pod [2] started running.
4. Change the selector of Service to point at the label of the new Pod [2].

And you're done! The Service will select the new Pod to expose on internet.

<a name="how-replica-set-works"></a>

## How replica set works
You can have a maximum number of pods running at the same time using a single configuration.

Besides, if one of the pods goes down, the replica set starts a new one in order to reach the maximum pods you specified.

<a name="how-deployment-works"></a>

## How deployment works
We use the Deployment to manage our replica sets.

Basically, after applying a change to a Deployment, it will initialize a new replica set and, after it is ready, the old one goes down automatically.

<a name="service-discovery-and-communication-between-pods"></a>

## Service discovery and communication between pods
In order to communicate pods, we can not just use the IP of pods, since it gets random values everytime it restarts. So in order to achieve it, we need to use a Kubernetes Service called **kube-dns** - this service has all key pairs with ip and label of current running pods.

So, if we have a Pod called **queue** and he has its own Service with **ClusterIP** type it will be accessible inside the **VM** by its label name: **queue**.

Besides, if it has a Service with **NodeType** type it will be accessible outside the **VM** using the **Kubernetes IP** and the **Port** that was defined on its service.

<a name="how-persistent-volume-works"></a>

## How persistent volume works
If we create a pod that needs to save data (per example, a mongodb pod) we need to specify in its config file a option called **volumeMounts** and **volumes**, because, if we don't do that, the data will be saved on the container and, if it dies, the data dies with it.

With a persistent volume we can keep the data safe on the needed folder while making sure it will not die if the pod dies.

In order to make it scalable, we can create another file only to config the storage that will be used by the pod (Ex: [PodConfig](example/mongo-stack.yaml) and [StorageConfig](example/storage.yaml)).

So, we can have a **Persistent Volume** config and a **Persistent Volume Claims** config for the given pod (Basically the first gives the default config for the persistent storage and the second makes a request to use a persistent storage based on the default config)

<a name="how-to-get-into-ec2-instance"></a>

## How to get into EC2 Instance
In order to get into a ec2 instance, you'll need to create a new one, get a key pair and run the following commands
```sh
ssh -i $KEY_PAIR_FILE_NAME.pem ec2-user@$EC2_INSTANCE_IP

# Use the command below if you have permission issues when doing the command above
chmod go-rwx $KEY_PAIR_FILE_NAME.pem
```

<a name="using-nano-editor-with-kops"></a>

## Using nano editor with KOPS
```sh
echo $EDITOR
export EDITOR=nano
```

<a name="how-to-install-prometheus-and-grafana-on-ec2-instance"></a>

## How to install Prometheus and Grafana on EC2 Instance
1. Install [Helm](https://github.com/helm/helm) on the EC2 Instance

2. Add permissions to Helm on the EC2 Instance
```sh
kubectl create serviceaccount --namespace kube-system tiller

kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}' 
```

3. Add the stable repository for helm
```sh
helm repo add stable https://kubernetes-charts.storage.googleapis.com

helm repo update
```

4. Install Prometheus package
```sh
helm install monitoring stable/prometheus-operator
```

5. Change Grafana config
```sh
kubectl edit service/monitoring-grafana

# Then change the 'Type: ClusterIP' to 'Type: LoadBalancer'
```

6. Open grafana
```
Go to the LoadBalancer URL that's related to Grafana and use the following credentials to log into Grafana:

user: admin
password: prom-operator
```
More helm charts for kubernetes can be found on [helm/charts](https://github.com/helm/charts) repository

<a name="how-to-use-the-alertmanager-with-slack"></a>

## How to use the AlertManager with Slack
1. Get the pod name used by alertmanager (Ex: **prometheus-alertmanager**)

2. Delete its secret
```sh
kubectl delete secret prometheus-alertmanager
```

3. Create a new secret with the needed options (Ex: [example/prod/alermanager.yaml](./example/prod/alertmanager.yaml))
```sh
kubectl create secret generic prometheus-alertmanager --from-file=alertmanager.yaml
```

<a name="helping-prometheus-to-verify-etcd-service"></a>

## Helping Prometheus to verify ETCD service
Since the default gateway on AWS is configured such a way prometheus can not access ports 4001 and 4002, we need to add this ports manually with the help of the following steps:

1. Go to ```Security Groups```

2. Click on the **master node security group**

3. Click on **Inbound**

4. Click on **Edit**

5. Change the port range of ```4003 - 65535``` to ```4001 - 65535```

<a name="how-to-autoscale-pods"></a>

## How to Auto-scale Pods
We're able to define some rules in order to automatically scale the pods when it gets a defined maximum use of **Memory, CPU, etc**.
```sh
kubectl autoscale deployment $DEPLOYMENT_NAME --cpu-percent 400 --min 1 --max 4
```

By doing it, the pods will scale up when above the expected cpu value, but, when it gets below the expected for some time (~10min) it will scale down automatically.

In order to see the current **Horizontal Pod Autoscalling** we can use the following command:
```sh
kubectl get hpa
```

<a name="readiness-and-liveness-probe"></a>

## Readiness and Liveness Probe
When we increase the number of pod instances, after the new pods are up, they will need some seconds before being able to receive requests (what can give back a 502 response).

To solve this problem, an approach we can take in order to solve this problem is adding a ```readinessProbe``` configuration to the **Pod .yaml**.

By doing so, the requests will go only to the up pod till it gets the time you configured on ```readinessProbe```. After the new Pod has reached the configured amount of time up, it will be able to receive requests.

<a name="useful-commands"></a>

## Useful commands
```sh
minikube status # Show status of minikube

minikube ip # Shows ip of current running minikube

minikube restart # Restarts minikube

minikube stop # Stops minikube

docker ps # Show running containers

docker container stop $CONTAINER_ID # Stop given container by id
# Ex: docker container stop 23kda2135

docker container run -p $FROM_PORT:$TO_PORT -d $DOCKER_IMAGE_NAME:$DOCKER_IMAGE_RELEASE
# Ex: docker container run -p 8080:80 -d richardchesterwood/k8s-fleetman-webapp-angular:release0-5

systemctl status docker # Status of current docker service running on computer

$SERVICE_NAME service restart # Restarts given service
# Ex: docker service restart

kubectl describe pod $POD_NAME # Describes status of given pod by name
# Ex: kubectl describe pod webapp

kubectl describe replicaset $REPLICASET_NAME # Describes status of given replicaset name
# Ex: kubectl describe replicaset webapp

kubectl describe service $SERVICE_NAME # Describes status of given service name
# Ex: kubectl describe service fleetman-webapp

kubectl -it exec $POD_NAME sh # Executes the terminal inside given pod by name
# Ex: kubectl -it exec webapp sh

kubectl get pods # Gets pods

kubectl get pods --show-labels # Gets pods and labels

kubectl get pods --show-labels -l $LABEL_NAME=$LABEL_VALUE # Gets pods with the given label
# Ex: kubectl get pods --show-label -l release=0

kubectl delete svc $SERVICE_NAME # Deletes a service using its name
# Ex: kubectl delete svc fleetman-webapp

kubectl delete pod $POD_NAME # Deletes a pod using its name
# Ex: kubectl delete pod webapp-release-0-5

kubectl rollout status deploy $DEPLOYMENT_NAME # Shows a info about the current deployment running
# Ex: kubectl rollout status deploy webapp

nslookup $SERVICE_NAME_OR_POD_NAME # Gets the IP Address for the given ip/service name, must be used with the sh of the pod/service
# Ex: nslookup database

kubectl delete -f $FILES_PATH # Deletes kubernetes resources based on files of the given path
# Ex: kubectl delete -f .

kubectl logs $RESOURCE_NAME # Shows the logs of the given resource name, useful to debug
# Ex: kubectl logs queue

kubectl get pods -o wide # Shows pods and node ips

kops update cluster ${NAME} --yes # Deploys the cluster with the configurations you've made

kops delete cluster --name ${NAME} --yes # Deletes the cluster

kops edit cluster ${NAME} # Edits the default cluster configurations

kops get ig --name ${NAME} # Shows the current configuration of cluster

helm ls # Lists all helm installations

helm delete --purge $INSTALLATION_NAME # Deletes helm installations
# Ex: helm delete --purge mysql

kubectl config view --minify # Shows set configs for the kubernetes
```
