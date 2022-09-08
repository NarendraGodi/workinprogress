## Taints and Tolerations

_So far the scheduling criteria has been applied at the Pod level, to make Pod to target a particular node based on the Pod's requirement._  

**Taints** _are applied at the node level to repel certain pods. For Pods to run on the nodes which have Taints should come with Tolerations inorder to execute on that particular node._  

In this case we have 3 nodes as given below.
```bash
$ kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
master     Ready    master   25d   v1.19.1
worker-1   Ready    <none>   25d   v1.19.1
worker-2   Ready    <none>   25d   v1.19.1
```  

_Both the Worker nodes will be labeled as red and yellow respectively_
```bash
$ kubectl label node worker-1 color=red
node/worker-1 labeled
$ kubectl label node worker-2 color=yellow
node/worker-2 labeled
$ kubectl get nodes worker-1 --show-labels
NAME       STATUS   ROLES    AGE   VERSION   LABELS
worker-1   Ready    <none>   25d   v1.19.1   beta.kubernetes.io/os=linux,color=red
$ kubectl get nodes worker-2 --show-labels
NAME       STATUS   ROLES    AGE   VERSION   LABELS
worker-2   Ready    <none>   25d   v1.19.1   beta.kubernetes.io/os=linux,color=yellow

```  
* _Taint the nodes as process=p1:NoSchedule and process=p2:NoSchedule respectively_  

**Taint Syntax:** kubectl taint node <node-name> key=value:NoSchedule
```bash
$ kubectl taint node worker-1 process=p1:NoSchedule
node/worker-1 tainted
$ kubectl taint node worker-2 process=p2:NoSchedule
node/worker-2 tainted
```  
* _Please create a pod to target a node with the label color=red with nodeSelector and run it_

```yaml
apiVersion: v1
kind: Pod
metadata:
  test-pod
spec:
  containers:
     - image: nginx
       name: nginx
  nodeSelector:
    color: red
```
_After running verify the Pod's Status.
```bash
$ kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
test-pod   0/1     Pending   0          15s
```  
_We can see that the pod has been created but not running.Please verify by describing the Pod._
```bash
kubectl describe pod test-pod
# In the last line it has the events section which tells us the reason
Events:
  Type     Reason            Age                From               Message
  ----     ------            ----               ----               -------
  Warning  FailedScheduling  19s (x3 over 99s)  default-scheduler  0/3 nodes are available: 1 node(s) had taint {process: p1}, that the pod didn't tolerate, 2 node(s) didn't match node selector.
```
_It clearly says that the out of the 3 nodes 2 nodes did not match the nodeSelector and 1 has taint on it. So now please create a Pod with tolerations_  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
     - image: nginx
       name: nginx
  nodeSelector:
    color: red
  tolerations:
    - key: process
      operator: Equals
      value: p1
      effect: NoSchedule
```  
_Now check for the status_
```bash
$ kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
test-pod   1/1     Running   0          75s
```

