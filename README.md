# udemy_KubernetesMastery
Udemy course: Kubernetes Mastery: Hands-On Lessons From A Docker Captain

## shpod
* helper for running commands in a pod
* Usage:
  ```
  $ kubectl apply -f https://k8smastery.com/shpod.yaml
  $ kubectl attach --namespace=shpod -it shpod
  
  # when done with shpod
  $ kubectl delete -f https://k8smastery.com/shpod.yaml
  ```

## kubectl commands:
* first look:
  ```
  $ kubectl get node  
  $ kubectl get nodes -o json | jq ".items[] | {name: .metadata.name} + .status.capacity"
  $ kubectl get nodes -o yaml
  
  $ kubectl describe node/docker-desktop
  
  $ kubectl api-resources
  
  $ kubectl explain node
  $ kubectl explain node.spec
  $ kubectl explain node --recursive
  
  $ kubectl get services
  $ kubectl get pods
  $ kubectl get pods --all-namespaces
  
  $ kubectl get namespaces
  
  $ kubectl -n kube-public get configmap
  $ kubectl -n kube-public get configmap cluster-info -o yaml
  ```
* first deployment:
  ```
  $  kubectl run pingpong --image alpine ping 1.1.1.1
  
  $ kubectl logs -l run=pingpong --tail 3 -f
  ```

* Logs - stern tool
  * stern: [github](https://github.com/wercker/stern)
  ```
  $ stern pingpong
  $ stern --tail 1 --timestamps pingpong
  ```
  
* First Assignment:
  ```
  $ kubectl --help
  
  $ kubectl get node
  $ kubectl get node -o wide
  
  $ kubectl get pod --namespace kube-system
  
  $ kubectl create deployment ticktock --image=bretfisher/clock
  
  $  kubectl scale --replicas=3 deployment/ticktock
  
  $ kubectl logs --tail=1 -lapp=ticktock
  
  $ kubectl delete deployment ticktock
  ```
* Sample Application - DockerCoins
  ```
  # docker compose - install
  $ curl -o docker-compose.yaml https://k8smastery.com/dockercoins-compose.yaml
  $ docker-compose up
  

  # kubernetes - install
  $ kubectl get namespace
  $ kubectl create --namespacee coins

  $ kubectl create deployment redis --image=redis --namespace coins
  $ kubectl create deployment hasher --image=dockercoins/hasher:v0.1 --namespace coin
  $ kubectl create deployment rng --image=dockercoins/rng:v0.1 --namespace coin
  $ kubectl create deployment webui --image=dockercoins/webui:v0.1 --namespace coin
  $ kubectl create deployment worker --image=dockercoins/worker:v0.1 --namespace coin

  $ kubectl get all --namespace coin

  $ kubectl logs pod/worker-699dc8c88-m7rjk --namespace coin

  $ kubectl expose deployment redis --port 6379 --namespace=coin
  $ kubectl expose deployment rng --port=80 --namespace coin
  $ kubectl expose deployment hasher --port 80 --namespace coin

  $ kubectl logs pod/worker-699dc8c88-m7rjk --namespace coin --follow --tail 1 

  $ kubectl expose deploy/webui --type NodePort --port 80 --namespace coin
  $ kubectl get service --namespace coin


  # scaling the deployments
  $ kubectl apply -f https://k8smastery.com/shpod.yaml  # in second terminal
  $ kubectl attach -it --namespace shpod shpod

  $ watch -- kubectl get pod --namespace=coin # in second terminal

  $ kubectl scale --replicas=2 deployment/worker --namespace coin  # double mining speed
  $ kubectl scale --replicas=3 deployment/worker --namespace coin  # triple mining speed
  $ kubectl scale --replicas=8 deployment/worker --namespace coin  # doesn't increase mining speed

  # get the IP addresses for hasher and rng -- in an instance of shpod
  $ kubectl attach --namespace=shpod -it shpod
  $ HASHER=$(kubectl get svc hasher -o go-template={{.spec.clusterIP}} --namespace coin)
  $ RNG=$(kubectl get svc rng -o go-template={{.spec.clusterIP}} --namespace coin)
  $ httping -c 3 $HASHER
  $ httping -c 3 $RNG  # this seems slow
  ```
