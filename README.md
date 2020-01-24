# k8s-microservices
A simple example of microservices architecture using kubernetes

## Needed to work locallly
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

kubectl describe pod $POD_NAME # Describers status of given pod by name
# Ex: kubectl describe pod webapp

kubectl -it exec $POD_NAME sh # Executes the terminal inside given pod by name
# Ex: kubectl -it exec webapp sh
```