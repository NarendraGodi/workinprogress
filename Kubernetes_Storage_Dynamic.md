## Kubernetes Storage Dynamic:

What is Dynamic provisioning in Kubernetes?


Creating Dynamic Storage Provisioning in Google Kubernetes Engine Cluster:

Open https://console.cloud.google.com/ with your crentials.

1. From Navigation Menu ==> Go To Kubernetes Engine ==>  Enable "Kubernetes Engine API".
2. Once API is enabled ==> Click Cluster and Create.
3. Choose Standard Cluster ==> Provide Cluster name ==> Choose the version.
4. Under Node Pools ==> Select your node requirement like number of nodes.
5. Under Nodes ==> Select the node level requirement such as Memory , OS.
6. Click Create and wait for the Cluster to Populate.
7. Once the Cluster has been created, click on the 3 dots in the end and click on Connect. And copy the URL.
8. Open Cloud Shell and paste the URL. this will copy the config file, So that you can connect to the Kubernetes Cluster.


Storage Class : A Storage class will allow Kubernetes Administrators to specify the type of Storage services they offer in the Cluster.

A Storage class will help us define the type of Storage that is required for a Specific Applications. For example we have a high priority application which need high performance for those applications we create a Storage class which will assign a SSD type of Storage and for low priority applications we can create a Storage class to assign a HDD Storage.

```yaml
# Creation of Storage Class 
```
```yaml
# Creation of PVC
```
```yaml
# Creation of Deployment and provisioning the Volume.
```

