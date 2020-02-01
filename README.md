# k8s-microservices
A simple example of a microservices architecture using kubernetes.

It is a shipping tracker, composed by:
- Webapp ```(Angular and Spring)```
- Message Queue ```(ActiveMQ)```
- Position Tracker ```(Java)```
- Position Simulator ```(Java)```
- API Gateway ```(Java)```
- Persistent Storage ```(MongoDB)```

All the services/resources above run inside a kubernetes cluster.

All the files used to config the microservices system can be found inside the **example** folder.

## Summary

- [ How to run locally ](#how-to-run-locally)
- [ How it works ](#how-it-works)
- [ How to apply changes ](#how-to-apply-changes)
- [ How to see status ](#how-to-see-status)
- [ Zero down time deployment ](#zero-down-time-deployment)
- [ How replica set works ](#how-replica-set-works)
- [ Service discovery and communication between pods ](#service-discovery-and-communication-between-pods)
- [ How persistent volume works ](#how-persistent-volume-works)
- [ How to get into EC2 Instance ](#how-to-get-into-ec2-instance)
- [ How to install KOPS on EC2 Instance ](#how-to-install-kops-on-ec2-instance)
- [ How to install KUBECTL on EC2 Instance ](#how-to-install-kubectl-on-ec2-instance)
- [ Permissions KOPS user needs from AWS ](#permissions-kops-user-needs-from-aws)
- [ Using nano editor with KOPS ](#using-nano-editor-with-kops)
- [ Useful commands ](#useful-commands)

<a name="how-to-run-locally"></a>

## How to run locally
You'll need to install **Minikube**, **Docker**, **VirtualBox** and **Kubernetes CLI**.

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
kubectl apply -f ./example/workload.yml
kubectl apply -f ./example/services.yml
kubectl apply -f ./example/mongo-stack.yml
kubectl apply -f ./example/storage.yml
```

<a name="how-it-works"></a>

## How it works
Basically in Kubernetes we use the concept of Pod (that's like VM which you can run multiple containers inside).

If we want to make this Pod to be visible on the internet, we need to use a Service to expose it.

Example:
```sh
# Pod that runs a Angular Application with the help of a pre-made docker image
./webapp-pod.yml

# Service that exposes the Angular Application to internet
./webapp-service.yml
```

<a name="how-to-apply-changes"></a>

## How to apply changes
```sh
kubectl apply -f $FILE_NAME

# Examples
# kubectl apply -f webapp-pod.yml
# kubectl apply -f webapp-service.yml
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

In order to make it scalable, we can create another file only to config the storage that will be used by the pod (Ex: [PodConfig](example/mongo-stack.yml) and [StorageConfig](example/storage.yml)).

<a name="how-to-get-into-ec2-instance"></a>

## How to get into EC2 Instance
In order to get into a ec2 instance, you'll need to create a new one, get a key pair and run the following commands
```sh
ssh -i $KEY_PAIR_FILE_NAME.pem ec2-user@$EC2_INSTANCE_IP

# Use the command below if you have permission issues when doing the command above
chmod go-rwx $KEY_PAIR_FILE_NAME.pem
```

<a name="how-to-install-kops-on-ec2-instance"></a>

## How to install KOPS on EC2 Instance
```ssh
curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64

chmod +x ./kops

sudo mv ./kops /usr/local/bin/
```

<a name="how-to-install-kubectl-on-ec2-instance"></a>

## How to install KUBECTL on EC2 Instance
```ssh
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl
```

<a name="permissions-kops-user-needs-from-aws"></a>

## Permissions KOPS user needs from AWS
```
AmazonEC2FullAccess
AmazonRoute53FullAccess
AmazonS3FullAccess
IAMFullAccess
AmazonVPCFullAccess
```

<a name="using-nano-editor-with-kops"></a>

## Using nano editor with KOPS
```sh
echo $EDITOR
export EDITOR=nano
```

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
```
