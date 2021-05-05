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
  ```
