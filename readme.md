## Challenge 3
Deploying on Azure: Build a Jenkins CI/CD pipeline similar to previous challenges, but on Microsoft Azure.

# Infrastructure 
You will need to have a Azure CLI already configurated to use these commands. Simplly refer to the Azure documents for setting up and logging in (az login)

To set up the Azure infrastructure, we will need to create resource groyps

```
az group create --name devOpsGroup --location eastus
```
Once this is completed, you can create a cluster by using 
```
az aks create --resource-group devOpsGroup --name devOpsCluster --node-count 1
```
Now you can load your credentials into you .kube config by running this command
```
az aks get-credentials --resource-group devOpsGroup --name devOpsCluster

This will take a few minutes to complete.

# Jenkins Image
You may want to build and push your own Jenkins image, to do so
```
docker build -t yourdockeruser/yourjenkinsimage .
docker push yourdockeruser/yourjenkinsimage
```

# Kubernetes Deployment
This step will help with setting up 'namespace', 'volume', 'deployment', 'load-balancer', 'service accounts', 'cluster rolebindings' and many more configuration with one command.

Make sure to update the name of your docker image in the yaml file, under deployment before procceding.

```
kubectl apply -f jenkinsk8s.yaml
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

# Deleting
You may want to delete deployment and services first based on your perferences. But if you want to shut down the whole infrausture by deleting the whole resource group, use 
```
az group delete --name devOpsGroup
```