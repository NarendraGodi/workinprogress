
* _What are the Main fields in Service Yaml file?  
Answer:_  
      1. apiversion  
      2. kind  
      3. metadata  
      4. spec  

**apiVersion:** it contains the k8s api used for creating the service.  
**kind:** it tells what type of kubernetes service it is.  
**metadata:** it contains the name of the service and labels.  
**spec:** it contains the selector to identify the deployment, ports, type of port and protocol.  

* _How to create a deployment from command line ?_
```bash
godi@master:~/service$ kubectl create deploy nginx-deploy --image nginx --dry-run=client -o yaml > deploy_example.yaml  
godi@master:~/service$ kubectl apply -f deploy_example.yaml 
```
**Yaml for the Service**
```yaml
apiVersion: v1
kind: Service
metadata: 
   name: nginx-service
spec:
  ports:
  - port: 80
    targetPort: 80
  type: NodePort
  selector:
    app: nginx-deploy
```
- Here in the list of ports:
  - port ==> is the port of the service
  - targetPort ==> is the port of the container
  - NodePort ==> is the port of the node.

* _When no NodePort is given how to get the NodePort ?  
Answer: By using kubectl get svc_
``` bash 
godi@master:~/service$ kubectl get svc
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP        16d
nginx-service   NodePort    10.101.26.120   <none>        80:31232/TCP   19m
```
* _What is Cluster IP?  
Answer: Cluster IP is used to access the application within the Cluster_

* _How does Applications interact or communicate within the cluster?
Answer : They will be interacting or communicating through cluster IP_


*  _When no type is defined for a service what is the default one?  
Answer: By Default kubernetes service will take CLusterIp as its type_
* _We already have deployment in place, and we need to create a service to expose the deployment how to do it using kubectl?  
Answer:_
```bash
godi@master:~/service$ kubectl expose deploy nginx-deploy --port 80 --target-port 80 --type NodePort
service/nginx-deploy exposed
```
* _How to edit Service from NodePort to ClusterIP ?  
Answer:_
```bash
godi@master:~/service$ kubectl edit svc nginx-deploy 
```

**Service type: Load Balancer:** When the Service type is chosen as "LoadBalancer" a layer 4 load balancer will be created by the cloud provider if the cluster is a cloud provided one and if the Service is created in a cluster created by Kubeadm then it will go into pending.  




