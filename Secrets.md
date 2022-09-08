**Secrets** *are one of the Kubernetes Object similar to ConfigMap which contains sensitive data such as API Tokens, Passwords and keys.*  

 - *Even though they are used for handling sensitive data they can be edited and seen by anyone who has access to create a Pod.  

 - *Secrets can be created independently of the Pods that use them other wise the sensitive details need to be added as environment variables 
   at the Pod level which are exposed.*

 - *As **Secrets** can be accessed by anyone who has the role of creating a Pod. It is highly recommended to use service accounts and
   use https for user authentication and validation.*

 - *Secrets are stored unencrypted in the **etcd** data store and anyone who has access to the API should be able to access and modify the
   secrets.*

 - *Secrets can be created in 3 ways:*

		   1. using Kubectl command
		   2. With Manifest File
		   3. Using Kustomize*
***Note**: Individual Secrets should not exceed the maximum size of 1MiB.*

 - *Create a Secret and describe it to check how it looks. Before that make sure the values that you are entering are base 64 encoded.*

		For Example am using the below details.
		name = mojojojo
		password= PowerPuffGirls
Here these two values need to be converted to base 64 as follows.

```bash
$ echo mojojojo | base64
bW9qb2pvam8K

$ echo PowerPuffGirls | base64
UG93ZXJQdWZmR2lybHMK
```

 - *So now the encoded values are as given below respectively.

			name = bW9qb2pvam8K
			password= UG93ZXJQdWZmR2lybHMK
			Describing the Secret:*
```bash
$ kubectl describe secret my-secret
Name:         my-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
name:      9 bytes
password:  15 bytes
```

*Now the Secrets Yaml File looks like given below.*
```yaml
# Obviously this is not for PROD environment. Please dont take this too seriously.
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
data:
  name: bW9qb2pvam8K
  password: UG93ZXJQdWZmR2lybHMK
```
*Now create the Secret*
```bash
$ kubectl apply -f my-secret.yaml 
secret/my-secret created
```
*These values will be decoded at the application level. Now attach it to the pod and print the values.*

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command: ['sh','-c','echo username is $uname and password is $upassword;sleep 3600']
      env:
        - name: uname
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: name
        - name: upassword
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: password

```
*Output from the Pod logs:*

```bash
$ kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
secret-pod   1/1     Running   0          21s

$ kubectl logs secret-pod
username is mojojojo and password is PowerPuffGirls
```
*Even though the the values are given as base64 encoded, Kubernetes will automatically decode the values and will be used by the application.
What values do we find inside the Pod*?
```bash
$ kubectl exec -it secret-pod /bin/bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.

root@secret-pod:/# echo $uname
mojojojo

root@secret-pod:/# echo $upassword
PowerPuffGirls
```
*Attaching the Secret as a Volume to a Pod.*
```yaml
apiVersion: v1  
kind: Pod  
metadata:  
  name: secret-vol-pod  
spec:  
  containers:  
    - name: ubuntu  
      image: ubuntu  
      command: ['sh','-c','sleep 3600']  
      volumeMounts:  
      - mountPath: /output/  
        name: my-secret-vol  
  volumes:  
    - name: my-secret-vol  
      secret:  
        secretName: my-secret
```
*Now enter the Pod and check the values:*
```bash
$ kubectl apply -f secret-vol-pod.yaml 
pod/secret-vol-pod created

$ kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
secret-vol-pod   1/1     Running   0          5s

$ kubectl exec -it secret-vol-pod /bin/bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.

root@secret-vol-pod:/# cd /output/

root@secret-vol-pod:/output# ls
name  password

root@secret-vol-pod:/output# cat name 
mojojojo

root@secret-vol-pod:/output# cat password 
PowerPuffGirls
```