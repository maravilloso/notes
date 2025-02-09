Kubernetes
========

# Configuration / Contexts
```shell
# Show configuration
kubectl config view

# or
cat ~/.kube/config

# Show all contexts for the default configuration
kubectl config get-contexts

# Show current context
kubectl config current-context

# Change/switch to another context
kubectl config use-context mmlyle-preprod
```
# List all services in the namespace
```shell
  kubectl get services
```
# Pods information
```shell
  # List all pods in the current namespace
  kubectl get pods -o wide
  
  # Search for concrete pod name/s in all namespaces
  kubectl get pods -o wide --all-namespaces | grep bbox
  
  # Get the IP of a runnig pod
  kubectl get pod dmstg-deployment-54fb7c845c-64c6c --template '{{.status.podIP}}'
  
  # Detailed information about a pod
  kubectl describe pods migration-tool-77c9954d5c-b5p92
```
# Copying file to/from pods
```shell
  # Copy files to remote pod
  kubectl cp myfile.txt -n qvantel migration-tool-68744c599b-74wz5:/home/cassandra
  
  # Downloading file from pod
  kubectl cp platform/qvt-postgredb-0:pg_counter.pl ./pg_counter.pl
```
# Commands / sessions in pods
```shell
  # Command line session in remote pod
  kubectl exec -it -n qvantel migration-tool-68744c599b-74wz5 -- bash
  
  # alternatively you can define the default namespace for that context...
  kubectl config set-context --current --namespace=qvantel
  #... and then all the subsequent command will use that namespace 
  kubectl exec -it migration-tool-68744c599b-74wz5 -- bash
  
  # Execute a single command on the remote pod
  kubectl exec migration-tool-68744c599b-74wz5 -- ls /
```
# Port forwarding
```shell
  kubectl port-forward -n platform qvt-postgredb-legacyhistory-0 5432:5432
```
___
# minikube
  Enables working in a local K8s cluster for developing purposes.
  To make a docker image available in the minikube register space, so it doesn't goes to public one by default we need to switch to minikube's docker instead:

```shell
  eval $(minikube -p minikube docker-env)
```
  and then build the image again inside it:
```shell
  docker build -t qvbvntv-maria106-pdi71 .
```
  as explained [here](https://medium.com/swlh/how-to-run-locally-built-docker-images-in-kubernetes-b28fbc32cc1d).

- ### Deploy a pod from a YAML spec
 
	```shell
	  kubectl create -f deployment.yaml
	```
- ### Delete the same pod
	
	```shell
	  kubectl delete -f deployment.yaml
	```
- ### Get local IP and port of a pod running in minikube
	
	```shell
	  $ minikube service datmigstg
	  |-----------|-----------|-------------|---------------------------|
	  | NAMESPACE |   NAME    | TARGET PORT |            URL            |
	  |-----------|-----------|-------------|---------------------------|
	  | default   | datmigstg |        3306 | http://192.168.49.2:32036 |
	  |-----------|-----------|-------------|---------------------------|
	```
- ### Check logs of a pod (i.e. to find out why it couldn't start properly)
	
	```shell
	  kubectl logs mypod --all-containers
	```
- ### Completely deletes minikube cluster and all profiles
	
	```shell
	  minikube delete --all --purge
	```