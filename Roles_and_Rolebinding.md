
## Roles and Role Binding  
  
 - **What is Authentication?**  
*In Computing the process or action of verifying the identity of a user or process is called Authentication.*  
 - **What is Authorisation?**  
*In Computing permissions to perform certain actions on a set of resources is called Authorisation.*  
 - **How does one access a Kubernetes Cluster ?**  
*Kubernetes Cluster can be accessed by a resource who has the config file in place for the kubectl to use in managing or performing actions on a cluster based on roles assigned to the user*.  
**What is a Role?**  
*In IT there are will be a number of SMEs who handles the application on day-to-day basis and not everyone is allowed to access all the available access on an Application.*  
Example:   
	  1. *A DB Administrator will have Read,Write,Execute,Delete* Permissions.   
	  2. *A DB Developer will have Read,Write,Execute permissions but not delete.*  
	  3. *A DB Monitoring will have Read only access and no other permissions.*

 <span style="color:#117A65">These set of  permissions each of these member has, are called Roles.</span>  
 - **What is the best way to handle when there are multiple SMEs in a   Project, and they need access to certain application resources?**  
Well, it is not easy to provide each and every individual separately. So instead we can create a Group for Admins,Developers,Monitoring teams respectively and create rules within these groups on what actions they can perform on the resources.     
And whenever a new resource joins the organisation, he/she can be added to these groups and their IDs will inherit all the properties defines in the groups.  
  
<span style="color:#117A65">This way of adding a certain user to their designated groups is called Role Binding.</span>  
  
 - **What does a Config File consists of ?**  
A config file consists of information about the cluster, user , API end point and Context information.


### Configuring a User to connect Kubernetes Cluster:

#### Installing Kubectl:
Generating a private key ⇒ openssl genrsa -out vamsi.key
```bash
$ openssl genrsa -out vamsi.key
Generating RSA private key, 2048 bit long modulus (2 primes)
..........................................................................+++++
```
#### Create a Certificate Signin Request(CSR) :
openssl req -new -key vamsi.key -out vamsi.csr -subj "/CN=vamsi/O=development"
Above command will generate a csr file.
```bash
$ openssl req -new -key vamsi.key -out vamsi.csr -subj "/CN=vamsi/O=development"
vamsi@jump-server:~$ ls -l
total 24
-rw-r--r-- 1 vamsi vamsi  915 Sep  7 14:26 vamsi.csr
-rw------- 1 vamsi vamsi 1675 Sep  7 11:08 vamsi.key
```

#### Generate a Self signed Certificate :
For this we need Certificate Authority keys for the kubernetes Cluster (ca.crt, ca.key)
The ca.crt and  ca.key files can be copie
d from Master (/etc/kubernetes/pki)node to the jump server which we have created.

```bash
~$ openssl x509 -req -in vamsi.csr \
> -CA ca.crt  -CAkey ca.key -CAcreateserial -out vamsi.crt -days 45
Signature ok
subject=CN = vamsi, O = development
Getting CA Private Key
~$ ls -la
total 48
-rw-r--r-- 1 vamsi vamsi 1066 Sep  7 11:22 ca.crt
-rw------- 1 vamsi vamsi 1675 Sep  7 11:22 ca.key
-rw-r--r-- 1 vamsi vamsi   41 Sep  7 11:27 ca.srl
-rw-r--r-- 1 vamsi vamsi 1017 Sep  7 11:27 vamsi.crt
-rw-r--r-- 1 vamsi vamsi  915 Sep  7 11:12 vamsi.csr
-rw------- 1 vamsi vamsi 1675 Sep  7 11:08 vamsi.key

```
A Self signed Certificate will be created with the combination  of user.csr, ca.crt, ca.key
When the kubectl command executes it will look for the config file in the .kube folder and if we observe there is no .kube folder yet.
```bash
$ ls -la
total 48
-rw-r--r-- 1 vamsi vamsi 1066 Sep  7 11:22 ca.crt
-rw------- 1 vamsi vamsi 1675 Sep  7 11:22 ca.key
-rw-r--r-- 1 vamsi vamsi   41 Sep  7 11:27 ca.srl
-rw-r--r-- 1 vamsi vamsi 1017 Sep  7 11:27 vamsi.crt
-rw-r--r-- 1 vamsi vamsi  915 Sep  7 11:12 vamsi.csr
-rw------- 1 vamsi vamsi 1675 Sep  7 11:08 vamsi.key
```
Now we can create a a config file using the "kubectl config" command 
```bash
$ kubectl config set-credentials vamsi --client-certificate=vamsi.crt --client-key=vamsi.key
User "vamsi" set.
vamsi@jump-server:~$ ls -la
total 52
drwxr-xr-x 2 vamsi vamsi 4096 Sep  7 13:35 .kube
-rw-r--r-- 1 vamsi vamsi  807 Sep  7 04:59 .profile
-rw-r--r-- 1 vamsi vamsi 1066 Sep  7 11:22 ca.crt
-rw------- 1 vamsi vamsi 1675 Sep  7 11:22 ca.key
-rw-r--r-- 1 vamsi vamsi   41 Sep  7 11:27 ca.srl
-rw-r--r-- 1 vamsi vamsi 1017 Sep  7 11:27 vamsi.crt
-rw-r--r-- 1 vamsi vamsi  915 Sep  7 11:12 vamsi.csr
-rw------- 1 vamsi vamsi 1675 Sep  7 11:08 vamsi.key
```
Now we can see the .kube file and let's see what is there in the .kube folder.
```bash
~$ cd .kube/
~/.kube$ ls
config
~/.kube$ cat config 
apiVersion: v1
clusters: null
contexts: null
current-context: ""
kind: Config
preferences: {}
users:
- name: vamsi
  user:
    client-certificate: /home/vamsi/vamsi.crt
    client-key: /home/vamsi/vamsi.key
```
The Config file has been generated but there is no cluster , context and current context fields.
Now we can pass the Cluster , api end point and certificate authority.
```bash
~/.kube$ kubectl config set-cluster kubecluster --server=https://10.128.0.2:6443 --certificate-authority=ca.crt
Cluster "kubecluster" set.
~/.kube$ cat config 
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/vamsi/ca.crt
    server: https://10.128.0.2:6443
  name: kubecluster
contexts: null
current-context: ""
kind: Config
preferences: {}
users:
- name: vamsi
  user:
    client-certificate: /home/vamsi/vamsi.crt
    client-key: /home/vamsi/vamsi.key
```
But still the kubectl will not be able to fetch any data as the current context is not set. So let's set the  context.

```bash
~/.kube$ kubectl config set-context vamsi-context --cluster kubecluster --user vamsi
Context "vamsi-context" created.
vamsi@jump-server:~/.kube$ cat config 
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/vamsi/ca.crt
    server: https://10.128.0.2:6443
  name: kubecluster
contexts:
- context:
    cluster: kubecluster
    user: vamsi
  name: vamsi-context
current-context: ""
kind: Config
preferences: {}
users:
- name: vamsi
  user:
    client-certificate: /home/vamsi/vamsi.crt
    client-key: /home/vamsi/vamsi.key
```
But Still the kubectl will not work as we can see the current-context is empty , so the current-context has to be set.
```bash
# The Available contexts are as given below.
kubectl config get-contexts
CURRENT   NAME            CLUSTER       AUTHINFO   NAMESPACE
          vamsi-context   kubecluster   vamsi      
# Now we have to set this context as current-context
~/.kube$ kubectl config use-context vamsi-context
Switched to context "vamsi-context".

~/.kube$ kubectl get nodes
Error from server (Forbidden): nodes is forbidden: User "vamsi" cannot list resource "nodes" in API group "" at the cluster scope

~/.kube$ cat config
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/vamsi/ca.crt
    server: https://10.128.0.2:6443
  name: kubecluster
contexts:
- context:
    cluster: kubecluster
    user: vamsi
  name: vamsi-context
current-context: vamsi-context
kind: Config
preferences: {}
users:
- name: vamsi
  user:
    client-certificate: /home/vamsi/vamsi.crt
    client-key: /home/vamsi/vamsi.key
```
We got the error message as given below:  
<span style="color:#6E2C00">Error from server (Forbidden): nodes is forbidden: User "vamsi" cannot list resource "nodes" in API group "" at the cluster scope</span>

The meaning of the above error message is that  the user is able to connect to the cluster  but he does not have the necessary permissions to fetch the data regarding the nodes.

This is where the roles come in to the picture, so we need create a role  to access the nodes related data and bind that role to the user.

 - **Roles are at Namespace level and Cluster Roles are at cluster level**.
 - **RoleBinding are at Namespace level and ClusterRoleBinding are at cluster level.**
If rolebinding for both namespace level and cluster level have been defined then ClusterRoleBinding will take the precedence.

Now let's create a role for the user vamsi where he can fetch the details of pods and deployments

```yaml
# Role : ns1_role.yaml
apiVersion: rbac.authorization.k8s.io/v1  
kind: Role  
metadata:  
  name: ns1-role  
  namespace: ns1  
rules:  
-  apiGroups: ["","apps"]  
   resources: ["pods","deployments"]  
   verbs: ["list","create"]
```
Now apply the role in the cluster:
```bash
# This command need to be executed in the Master Node

~$ kubectl apply -f ns1_role.yaml 
role.rbac.authorization.k8s.io/ns1-role created

~$ kubectl get roles
No resources found in default namespace. # No roles in default namespace.

~$ kubectl get roles -n ns1
NAME       CREATED AT
ns1-role   2022-09-07T15:23:51Z
```
Now let's create a deployment in ns1 namespace.

```bash
$ kubectl create deploy my-deploy --image=nginx --replicas=2 -n ns1
deployment.apps/my-deploy created
```
Now from the jump-server , let's see if vamsi is able to list pods and deployments.
```bash
~$ kubectl get pods -n ns1 # vamsi cannot fetch data from any other namespace including the default namespace except for ns1 namespace.
Error from server (Forbidden): pods is forbidden: User "vamsi" cannot list resource "pods" in API group "" in the namespace "default"
```
Vamsi are still not able to  fetch the pods because we have created a role but we did not bind that role to vamsi. So let's bind the role to vamsi.

```yaml
# Role binding : ns1_role_binding.yaml
apiVersion: rbac.authorization.k8s.io/v1  
kind: RoleBinding  
metadata:  
  name: ns1-role-binding  
  namespace: ns1  
subjects:  
  - kind: User  
    name: vamsi  
    apiGroup: rbac.authorization.k8s.io  
roleRef:  
  kind: Role  
  name: ns1-role  
  apiGroup: rbac.authorization.k8s.io
```
now apply the role binding:
```bash
# it must be applied at the Master node.
~$ kubectl apply -f ns1_role_binding.yaml 
rolebinding.rbac.authorization.k8s.io/ns1-role-binding created

$ kubectl get rolebindings -n ns1
NAME               ROLE            AGE
ns1-role-binding   Role/ns1-role   3m38s
```

Now let the user vamsi try to list the pods and deployments in the namespace ns1.

```bash
~$ kubectl get pods -n ns1
NAME                         READY   STATUS    RESTARTS   AGE
my-deploy-779ddd99b6-f77zb   1/1     Running   0          12m
my-deploy-779ddd99b6-kdw2s   1/1     Running   0          12m

~$ kubectl get deploy -n ns1
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
my-deploy   2/2     2            2           13m
```
But is it necessary to pass the namespace ns1 dynamically whenever we execute the kubectl command?
The Answe is no. we can set the namespace which we will be interacting at the configuration level.

```bash
# This must be executed in the Jump Server
~$ kubectl config set-context vamsi-context --cluster kubecluster --user vamsi --namespace ns1
Context "vamsi-context" modified.

~$ cat .kube/config 
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/vamsi/ca.crt
    server: https://10.128.0.2:6443
  name: kubecluster
contexts:
- context:
    cluster: kubecluster
    namespace: ns1
    user: vamsi
  name: vamsi-context
current-context: vamsi-context
kind: Config
preferences: {}
users:
- name: vamsi
  user:
    client-certificate: /home/vamsi/vamsi.crt
    client-key: /home/vamsi/vamsi.key
```
Now the namespace has been updated in the config file. let's see what happens if the namespace is not passed this time.
```bash
$ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
my-deploy-779ddd99b6-f77zb   1/1     Running   0          22m
my-deploy-779ddd99b6-kdw2s   1/1     Running   0          22m

$ kubectl get deploy
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
my-deploy   2/2     2            2           22m
```
###  Understanding the config file  
A config file will have the following information:
	1. clusters
	2. contexts
	3. current-context
	4. users

**Clusters**: *This field contains details of all the cluster information among the available cluster environments in the organisation.*
**Context**: *This field consists of information regarding the cluster , user and namespace for which the user has access to within that cluster.*  
**Current-Context**:  *The current-context tells us to which context the current user is pointing it to.*  
**Users**: *It contains the user details such as username , which client-certificate the user is using and which private key the user is using.*

Given below is how a Config file looks like.
```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/vamsi/ca.crt
    server: https://10.128.0.2:6443
  name: kubecluster
contexts:
- context:
    cluster: kubecluster
    namespace: ns1
    user: vamsi
  name: vamsi-context
current-context: vamsi-context
kind: Config
users:
- name: vamsi
  user:
    client-certificate: /home/vamsi/vamsi.crt
    client-key: /home/vamsi/vamsi.key
```