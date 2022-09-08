#### Kubernetes Storage

_The file system in a container is Ephemeral , meaning the file system is non-persistent storage. Once the Pod is destroyed all the data inside it will also be gone._

**Volume Implementation Concepts:**
1. Volume
2. Persistent Volume
3. Persistent Volume Claim
4. Storage Classes

|Docker|Kubernetes|
|------|----------|
|Bind Volume| Host Path|
|Tempfs|emptyDir|
|Volume|Volume Types|

_Volumes are at Pod Spec. volumeMounts are at container spec._

**Types of Provisioning:**

1. Static
2. Dynamic

Persistent Volume is created at Cluster level and Persistent Volume Claim is created at namespace level.

_What are the things needed When creating a Persistent Volume ?_

1. Storage Size
2. Access mode
3. Source

**Persistent Volume Reclaim Policy:**

1. retain
2. delete
3. recycle

```yaml
#First Create a PV
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol # name of the PV.
spec:
  capacity: # How much amount of Storage is needed.
    storage: 1Gi # How much amount of Storage is allocated for this PV.
  accessModes:
  - ReadWriteMany # The mode of the PV.
  nfs: # Type of Storage
    path: /opt/sfw/ # Mount path of the Storage.
    server: <ServerIP of NFS> # From where we are getting the Storage.
```
_For types of modes please refer __[accessModes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes)___
```yaml
# Now Create a PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs # name of the PVC.
spec:
  accessModes:
    - ReadWriteMany # Based on the type of access mode the PVC will claim the PV automatically.
  resources: 
    requests: # Creating a request to claim the available PV.
      storage: 500Mi # The amount of memory that need to be claimed.
```
```yaml
# Now create a Pod to use the PVC.

apiVersion: v1
kind: Pod
metadata:
  name: pv-pvc-pod
spec:
  containers:
  - name: busybox
    image: busybox
    # We will issue a command to keep the Pod running.
    command: ['sh','-c','while true;do echo this is PVC and PV example > /output/pv_pvc.txt;sleep 5;done']
    volumeMounts:
     - name: my-pvc-vol # Name of the Volume defined for Pod.
       mountPath: /output/ # path where we want to attach the PVC.
  volumes:
    - name: my-pvc-vol # Name of the Volume defined for Pod which we will use inside the container. 
      persistentVolumeClaim: # Type of Storage.
        claimName: pvc-nfs # Name of the PVC created for the use of the pod.
```
```yaml
# Create a deployment which uses volumes concept.
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ubuntu-deployment # name of the Deployment.
  labels:
    app: test # label to identify the deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: test # Telling deployment to check for the pods with the label "pp: test".
  template:
    metadata:
      name: ubuntu-pod
      labels:
        app: test # We are mapping the pod with its respective deployment.
    spec:
      containers:
      - name: ubuntu
        image: ubuntu
        command: ['sh','-c','while true;do echo $(hostname)+$(date)>> /output/hostdetails.txt;sleep 5;done']
        volumeMounts:
        - name: pod-vol
          mountPath: /output/
      volumes:
      - name : pod-vol
        persistentVolumeClaim:
          claimName: pvc-2
```


 



