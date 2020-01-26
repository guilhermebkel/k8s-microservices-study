# k8s-microservices
A simple example of microservices architecture using kubernetes

## How to run locally
You'll need to install **Minikube**, **Docker**, **VirtualBox** and **Kubernetes CLI**.

After installing the needed apps, do the following:

- Start minikube:
```sh
minikube start --vm-driver=virtualbox

# After using this command you can use only minikube start to make it to work next time
```

- Link your Docker CLI with minikube:
```sh
minikube docker-env # Shows the needed envs to config (manually)
eval $(minikube docker-env) # Automatically gets docker configured for you 
```

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

## How to apply changes
```sh
kubectl apply -f $FILE_NAME

# Examples
# kubectl apply -f webapp-pod.yml
# kubectl apply -f webapp-service.yml
```

## How to see status
```sh
kubectl get all
```

## Zero down time deployment
The basic concept is:
1. You have a running Pod [1] and Service.
2. You deploy a new Pod [2] with a new label.
3. The new Pod [2] started running.
4. Change the selector of Service to point at the label of the new Pod [2].

And you're done! The Service will select the new Pod to expose on internet.

## How replica set works
You can have a maximum number of pods running at the same time using a single configuration.

Besides, if one of the pods goes down, the replica set starts a new one in order to reach the maximum pods you specified.

## How deployment works
We use the Deployment to manage our replica sets.

Basically, after applying a change to a Deployment, it will initialize a new replica set and, after it is ready, the old one goes down automatically.

# Service discovery and communication between pods
In order to communicate pods, we can not just use the IP of pods, since it gets random values everytime it restarts. So in order to achieve it, we need to use a Kubernetes Service called **kube-dns** - this service has all key pairs with ip and label of current running pods.

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
```
