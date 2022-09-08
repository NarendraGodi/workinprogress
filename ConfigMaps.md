**ConfigMap**: _Configmap is a Kubernetes Object used for storing Non-Confidential data in the form of key-value pairs._

 - Config map is used for setting up configuration data apart from the
   Application code.
 -  *Pods can consume the data in the ConfigMaps in the form of Environment variables, Command Line Arguments or as configuration  files in a volume.*  
 -  *ConfigMaps are not suitable for holding large data. The size of the config Map should not exceed a maximum of 1MiB.*
 -  *Unlike other Kubernetes Objects which have spec a ConfigMap will have data field.*  
-   *A ConfigMap has data or binarayData fields.*

Lets create a ConfigMap which contains few environment variables and attach them to a Pod.
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
        name: config-env
data:
        name: ganesh
        org: TCS
        place: Hyderabad
```
*Now create the Configmap Object.*
```bash
$ kubectl apply -f cm-ex.yaml 
configmap/config-env created
$ kubectl get cm
NAME         DATA   AGE
config-env   3      7s
```
*Create a pod and and make is access the ConfigMap.*
```yaml
apiVersion: v1
kind: Pod
metadata:
        name: cm-pod
spec:
        containers:
        - image: ubuntu
          name: ubuntu
          command: ['sh','-c','echo Hi, My Name is $uname and am currently working for $uorg and am in $uplace;sleep 3600']
          env:
          - name: uname
            valueFrom:
              configMapKeyRef:
                      name: config-env
                      key: name
          - name: uorg
            valueFrom:
              configMapKeyRef:
                      name: config-env
                      key: org
          - name: uplace
            valueFrom:
              configMapKeyRef:
                      name: config-env
                      key: place
```
*Now run the Pod and check for the Logs.*
```bash
kubectl apply -f cm-pod.yaml 
pod/cm-pod created
# Check for the logs of the Pod.
$ kubectl logs cm-pod
Hi, My Name is ganesh and am currently working for TCS and am in Hyderabad
# You can see that the environemnt variables have been pulled form the config map and been assigned to uname,uorg and uplace respectively.
```
*ConfigMaps can also be mounted as Volumes and this will create a number of files for each variable by taking the key value as the file name and the value will be stored inside the file.*
*Create a Pod to accept the ConfigMap created earlier as a Volume.*
```yaml
apiVersion: v1
kind: Pod
metadata:
        name: cm-pod-vol
spec:
        containers:
        - image: ubuntu
          name: ubuntu
          command: ['sh','-c','sleep 3600']
          volumeMounts:
                  - name: cm-vol
                    mountPath: /output/
        volumes:
          - name: cm-vol
            configMap:
               name: config-env
```
*Now Run the Pod and login into it and check for the files created.*
```bash
$ kubectl apply -f cm-vol-pod.yaml 
pod/cm-pod-vol configured
$ kubectl exec -it cm-pod-vol /bin/bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@cm-pod-vol:/# cd /output/
root@cm-pod-vol:/output# ls
name  org  place
root@cm-pod-vol:/output# cat name 
ganesh
root@cm-pod-vol:/output# 
```

 - *If there are multiple containers in the Pod, then each container needs its own* `volumeMounts` and in this case the Pod will be
   automatically updated
 - whenever the ConfigMap has been updated.
   *ConfigMaps consumed as environment variables are not updated automatically and require a pod restart.*
   **Note:** A container using a ConfigMap as a [subPath](https://kubernetes.io/docs/concepts/storage/volumes#using-subpath) volume mount will not receive ConfigMap updates.
 *Lets update the ConfigMap and see if the volumeMounts are being updated in the Pod.*
 ```yaml
apiVersion: v1
kind: ConfigMap
metadata:
        name: config-env
data:
        name: ganesh
        org: TCS
        place: Bangalore
```
*The place has been changed from Hyderabad to Bangalore.
Now run the ConfigMap and check the update in the Pod.*

```bash
$ kubectl apply -f cm-ex.yaml
configmap/config-env created

$ kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
cm-pod       1/1     Running   0          59m
cm-pod-vol   1/1     Running   0          36m

$ kubectl exec -it cm-pod-vol /bin/bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@cm-pod-vol:/# cd /output/
root@cm-pod-vol:/output# ls
name  org  place
root@cm-pod-vol:/output# cat place 
Bangalore
root@cm-pod-vol:/output#
```
*Which process will update these values ?
Answer: kubelet*
*Immutable ConfigMaps are the ones which cannot be modified once created. Any ConfigMap that is non-Immutable  can be edited manually. Please see the below example how a immutable Configmap looks like.*
 ```yaml
apiVersion: v1
kind: ConfigMap
metadata:
        name: config-env
data:
        name: ganesh
        org: TCS
        place: Bangalore
immutable:  true 
```
*You will get the below error if you try to edit the ConfigMap that has been marked as immutable.*
```bash
$ kubectl get cm
NAME         DATA   AGE
config-env   3      3m6s
$ kubectl edit cm config-env
error: configmaps "config-env" is invalid
A copy of your changes has been stored to "/tmp/kubectl-edit-41gnw.yaml"
error: Edit cancelled, no valid changes were saved.
```