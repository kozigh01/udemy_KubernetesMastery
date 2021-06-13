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


  # get/deploy the manifest file for DockerCoin
  $ curl -o dockercoins.yaml https://k8smastery.com/dockercoins.yaml
  $ kubectl apply -f https://k8smastery.com/dockercoins.yaml --namespace coin
  ```
* Kubernetes Dashboard
  site: [github](https://github.com/kubernetes/dashboard)
  ```
  # get the manifest
  $ curl -o insecure-dashboard.yaml https://k8smastery.com/insecure-dashboard.yaml
  $ kubectl apply -f https://k8smastery.com/insecure-dashboard.yaml
  $ kubectl get service dashboard  # get port to access via localhost
  ```
* Kubernetes DaemonSet
  ```
  $ kubectl get deploy/rng -o yaml --namespace=coin > rng.yml
  # copy to new file and change kind to DaemonSet
  $ kubectl apply -f rng-daemonset.yml --namespace=coin  # has validation errors
  $ kubectl apply -f rng-daemonset.yml --namespace=coin --validation=false
  $ kubectl get all --namespace coin  # now have daemonset in addition to deployment for rng
  $ kubectl describe service rng --namespace=coin   # notice the selector value
  $ kubectl get pods --namespace coin --selector app=rng
  $ kubectl get pods --namespace coin -l app=rng
  $ kubectl delete deployment/rng --namespace=coin
  ```
* DockerCoins cleanup
  ```
  $ kubectl delete -f https://k8smastery.com/dockercoins.yaml --namespace coin
  $ kubectl delete daemonset/rng --namespace coin
  ```
* Rolling Updates
  ```
  $ kubectl apply -f https://k8smastery.com/dockercoins.yaml --namespace coin  
  $ kubectl get deploy --namespace coin -o json | jq ".items[] | {name: .metadata.name} + .spec.strategy.rollingUpdate"

  # trigger update (while watching pods/deployments/replicasets)
  $ kubectl set image deploy worker --namespace coin worker=dockercoins/worker:v0.2

  # trigger update to non-existing image
  $ kubectl set image deploy worker --namespace coin worker=dockercoins/worker:v0.3
  $ kubectl rollout status deploy worker --namespace coin   # in seperate terminal

  # trouble shooting
  $ kubectl get pods --namespace coin   # trouble shooting
  $ kubectl logs --namespace coin worker-6b6cd45fb6-qk4j2   # more trouble shooting
  $ kubectl describe deploy/worker --namespace coin   # more trouble shooting

  # return to previous versions
  $ kubectl rollout undo deploy/worker --namespace coin   # undo last update
  $ kubectl rollout history deploy worker --namespace coin
  $ kubectl describe replicaset worker --namespace coin | grep -A3 Annotations  # annotaions info
  $ kubectl rollout undo deployment worker --namespace coin --to-revision=1
  ```
* ConfigMap - Downward API example (introspection like)
  Note: namespace can be used in fully qualified domain name: curl api-backend.$MY_POD_NAMESPACE.svc.cluster.local
  ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: envar-demo
      labels:
        purpose: demonstrate-envars
      namespace: my-ns
    spec:
      containers:
      - name: envar-demo-container
        image: gcr.io/google-samples/node-hello:1.0
        env:
        - name: DEMO_FAREWELL
          value: "Such a sweet sorrow"
        - name: MY_POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
  ```
* ConfigMap - storing entire files
  ```
  $ kubectl create configmap my-app-config --from-file=app.config

  # with key different than the file name
  $ kubectl create configmap my-app-config --from-file=app.config=config.d/config-1.config --dry-run

  # ConfigMap with multiple keys - one per file in the config.d directory
  $ kubectl create configmap my-app-config --from-file=config.d --dry-run -o yaml 
  ```
* ConfigMap - storing individual values
  ```
  $ kubectl create configmap my-app-config --from-literal=foreground=red --from-literal=background=blue --dry-run -o yaml

  # from key value file
    $ kubectl create configmap my-app-config --from-env-file=config.d/app.conf --dry-run -o yaml
  ```
* ConfigMap - haproxy example: config map with file contents
  ```
  # create the configmap
  $ cd configmap-haproxy
  $ curl -O https://k8smastery.com/haproxy.cfg
  $ kubectl create configmap haproxy --from-file=haproxy.cfg
  $ kubectl get configmap haproxy -o yaml

  # use the configmap
  $ kubectl apply -f pod-haproxy.yaml  # or: kubectl apply -f https://k8smastery.com/haproxy.yaml
  $ kubectl attach --namespace=shpod -it shpod
  shpod$ kubectl get pod haproxy -o wide
  shpod$ IP=$(kubectl get pod haproxy -o json | jq -r .status.podIP)
  shpod$ curl $IP
  shpod$ curl $IP
  shpod$ curl $IP

  # cleanup
  $ kubectl delete configmap/haproxy pod/haproxy
  ```
* ConfigMap - individual key/values
  ```
  # create configmap
  $ kubectl create configmap registry --from-literal=http.addr=0.0.0.0:80
  $ kubectl get configmap registry -o yaml

  # use configmap
  $ cd configmap-registry 
  $ kubectl apply -f pod-registry.yaml
  $ kubectl attach --namespace=shpod -it shpod
  shpod$ kubectl get pod registry -o wide
  shpod$ IP=$(kubectl get pod registry -o json | jq -r .status.podIP)
  shpod$ curl $IP/v2/_catalog
  shpod$ exit

  # cleanup
  $ kubectl delete configmap/registry pod/registry
  ```
