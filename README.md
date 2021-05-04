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
