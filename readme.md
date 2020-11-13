## Challenge 3
Deploying on Azure: Build a Jenkins CI/CD pipeline similar to previous challenges, but on Microsoft Azure.

# Infrastructure 
You will need to have a Azure CLI already configurated to use these commands. Simplly refer to the Azure documents for setting up and logging in (az login)

To set up the Azure infrastructure, we will need to create resource groyps

```
az group create --name devOpsGroup3 --location westus
```
Once this is completed, you can create a cluster by using 
```
az aks create --resource-group devOpsGroup3 --name Challenge3 --node-count 2 --node-vm-size Standard_B2s
```
Now you can load your credentials into you .kube config by running this command
```
az aks get-credentials --resource-group devOpsGroup3 --name Challenge3
```

This will take a few minutes to complete.

# Jenkins Image
You may want to build and push your own Jenkins image, to do so
```
docker build -t yourdockeruser/yourjenkinsimage .
docker push yourdockeruser/yourjenkinsimage
```
Note, the username and password to Jenkins login is 'admin'

# Kubernetes Deployment
This step will help with setting up 'namespace', 'volume', 'deployment', 'load-balancer', 'service accounts', 'cluster rolebindings' and many more configuration with one command.

Make sure to update the name of your docker image in the yaml file, under deployment before procceding.

```
kubectl apply -f kube/
```

# View Important Data

Depending on aws, it should take a while to setup all services. You can view your pods by
```
kubectl get pods -n devops
```

Once it is completed, use commands to generate the token for service account and retrive the token by decoding. You will need to save it in the Jenkins Credentials
```
kubectl -n devops get serviceaccount jenkins-robot -o go-template --template='{{range .secrets}}{{.name}}{{"\n"}}{{end}}'
kubectl -n devops get secrets SERVICEACCOUNT -o go-template --template '{{index .data "token"}}' | base64 -d
```

To get the Token for accessing dashboard, use
```
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

If you want to check your context, you can simply go to your user\ .kube\config, to get your Kubernetes master  server url, simply use

```
kubectl cluster-info
```


# Access via Browser
Now it is all setup (again, keep in mind that this can take a few minutes), you can copy the external url and start building your project! Your URL can also be accessed via:
```
kubectl get services -n devops
```
Make sure to add the port number so that you can access the jenkins server.

Now your Jenkins configuration should be up and running. Congrats!

To access your k8s dashboard, use
```
kubectl proxy
```
Then insert your proxy followed by: 'localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/.'

Since you are also running this on Azure, you can skip the step and just use
```
az aks browse -g devOpsGroup3 -n challenge3
```

# Deleting
You may want to delete deployment and services first based on your perferences. But if you want to shut down the whole infrausture by deleting the whole resource group, use 
```
az group delete --name devOpsGroup3
```