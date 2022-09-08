#### What is Kubernetes Service ?
Service is one of the kubernetes object and is used for resolving the connectivity issues.  

* _What are the connectivity issues ?_  
  * _If there is an application is running on a pod and is being accessed by its IP and when the pod is deleted and recreated the ip will change.
  IP Address of a pod is not persistent._  
  * And also Service help in accessing the application outside the cluster exposed to the external world.

**Note == > Pod VS Service : Pod is a process however service is not a process.**

 * _What type of IP Address does a Service has?  
Answer: Static._  


* _Can we run two containers in a same pod , which are using same port numbers?  
Answer: No. Just like how we cannot run two applications using the same port number. The same applies here._  


* _How does the service know the latest IP of a Pod when a pod is destroyed and recreated?  
Answer: Kube proxy will update the registry in the service with the latest IP of the Pod._  


* _What is Kube Proxy ?  
Answer: kube-proxy is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept. kube-proxy maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside your cluster_


* _Does Service send the traffic to unhealthy nodes ?  
Answer: No_  


*  _How does the service know which details of the Pods need to be updated in its Registry?  
Answer: Service will be able to do that based on the labels defined in the Deployment/Pod_  


* _How many types of Services are there?_  
  * NodePort ==> A NodePort is an open port on every node of your cluster.  
  * Cluster IP ==> The ClusterIP provides a load-balanced IP address. One or more pods that match a label selector can forward traffic to the IP address.   
  * Load Balancer ==> A load Balancer sends the traffic to the pods in the healthiest node within the cluster.
  * headless service ==> 


* _If there are 3 nginx pods in the deployment, and you have created the service for them and the service will have all the details of the IPs.
If one of the pods is deleted the new pod will have a new IP. How will the service know?  
Answer: The Kube proxy will update by IP of the new pod in the deployment in the Service's registry(End Points)._


* **Service will use Random algorithm, and it has Session affinity as "yes" by default.**  


* _What will service do when it has 3 pods running in 3 nodes and has a nodePort as 8080?  
Answer: It will open the nodePort 8080 on all the 3 nodes._  


* _There are 3 nodes n1,n2,n3 and only one pod is running in n1 with nodePort 9090. Can we access the pod from n3?  
Answer: Yes. Even though there is no Pod in n3 the service can still route the traffic to the pod running in n1._  


* _Will the existing service with the nodePort 8090 will be opened when a new node is added to the cluster?  
Answer: Yes._  
