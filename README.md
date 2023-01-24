# JupyterHub on Azure Kubernetes Service (AKS) with Azure Active Directory Authentication | aks-jhub

JupyterHub is a powerful tool for creating and managing interactive Jupyter notebook environments for multiple users. When combined with the scalability and flexibility of the Azure Kubernetes Service (AKS), it becomes an ideal platform for data science and machine learning workloads. In this guide, we will walk through the process of deploying JupyterHub on AKS with Azure Active Directory (AAD) authentication.

## Prerequisites
- An Azure account
- A resource group in Azure
- The Azure CLI installed on your local machine
- The kubectl command-line tool installed on your local machine
- An AAD tenant with at least one user account
- A JupyterHub image (you can use one from the JupyterHub Helm Chart)

## Deployment Steps
- Create a new AKS cluster. You can do this using the Azure CLI with the following command:

    ```az aks create --resource-group nsaks-rg --name nsaks --node-count 2 --generate-ssh-keys```

    You can also use the Azure Portal to create a new AKS cluster.

- Connect to your AKS cluster using the following command:

    ```az aks get-credentials --resource-group nsaks-rg --name nsaks```

- Create a namespace for JupyterHub:

    ```kubectl create namespace jhub```

- Install the JupyterHub Helm chart and Create a new deployment for JupyterHub using the JupyterHub Helm Chart:

    ```helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/ helm install jupyterhub jupyterhub/jupyterhub --version=3.0.0 --namespace jhub```

- Create a new Service Principal in Azure for JupyterHub:
    ```az ad sp create-for-rbac --name jupyterhub-sp --role Contributor --scopes /subscriptions/<subscription-id>/resourceGroups/<resource-group> ```

- Create a new secret in Kubernetes for the Service Principal:
    ```kubectl create secret generic jupyterhub-sp-secret --from-literal=client-id=<client-id> --from-literal=client-secret=<client-secret> --namespace jhub```

- Update the JupyterHub Helm chart to use the new secret:
    ```helm upgrade jupyterhub jupyterhub/jupyterhub --version=0.9.1 --namespace jhub --set singleuser.extraEnv[0].name=AZURE_CLIENT_ID --set singleuser.extraEnv[0].valueFrom.secretKeyRef.name=jupyterhub-sp-secret --set singleuser.extraEnv[0].valueFrom.secretKeyRef.key=client-id --set singleuser.extraEnv[1].name=AZURE_CLIENT_SECRET --set singleuser.extraEnv[1].valueFrom.secretKeyRef.name=jupyterhub-sp-secret --set singleuser.extraEnv[1].valueFrom.secretKeyRef.key=client-secret```

- Create a new AAD group for JupyterHub users:
    ```az ad group create --display-name "JupyterHub Users" --mail-nickname "jupyterhub-users"```

- Add users to the AAD group:
    ```az ad group member add --group "JupyterHub Users" --member-id <user-object-id>```

- Update the JupyterHub Helm chart to use the AAD group:
    ```helm upgrade jupyterhub jupyterhub/jupyterhub --version=3.0.0 --namespace jhub --set hub.extraEnv[0].name=AAD_GROUP_ID --set hub.extraEnv[0].value=<group-object-id>```

- Expose the JupyterHub service with a LoadBalancer:
    ```kubectl expose deployment jupyterhub-hub --type=LoadBalancer --name=jupyterhub --namespace jhub```

- Get the external IP of the LoadBalancer:
    ```kubectl get service jupyterhub --namespace jhub```

- Access JupyterHub at the external IP in a web browser, and log in with your AAD credentials.

- Create new user profiles with the jupyterhub-admin command-line tool:
    ```kubectl exec -it jupyterhub-hub-0 --namespace jhub -- jupyterhub-admin create-user <username>```

- (Optional) To configure custom resource limits for the single-user notebook pods, you can use the singleuser.resources field in the Helm chart:
    ```helm upgrade jupyterhub jupyterhub/jupyterhub --version=3.0.0 --namespace jhub --set singleuser.resources.requests.cpu=1 --set singleuser.resources.requests.memory=2Gi --set singleuser.resources.limits.cpu=2 --set singleuser.resources.limits.memory=1Gi```

By following these steps, you should now have a fully functional JupyterHub deployment on AKS with AAD authentication for your users. Keep in mind that you should run the above steps periodically to update the JupyterHub deployment and keep it secure.


<!--


- Create a new service for JupyterHub using the following command:

    ```kubectl expose deployment jupyterhub --type LoadBalancer --name jupyterhub-lb --namespace jhub```

- Get the IP address of your JupyterHub service using the following command:

    ```kubectl get service jupyterhub-lb --namespace jhub```

- Use the IP address to access JupyterHub in your web browser.

## Conclusion
By following the steps above, you should now have JupyterHub running on AKS. You can use the service to create and manage notebooks, and invite other users to collaborate on your projects.

Be sure to clean up your resources when you are done to avoid unnecessary charges. You can delete the AKS cluster and the associated resources by running the following command:

```az group delete --name nsaks-rg```

### Note:

- For more customization options, you can refer to the JupyterHub Helm Chart documentation
- Make sure you have enough resources available in your subscription
- Also refer to Azure AKS documentation for more information on managing AKS clusters.


JupyterHub on Azure Kubernetes Service with Azure Active Directory Authentication
JupyterHub is a powerful tool for creating and managing interactive Jupyter notebook environments for multiple users. When combined with the scalability and flexibility of the Azure Kubernetes Service (AKS), it becomes an ideal platform for data science and machine learning workloads. In this guide, we will walk through the process of deploying JupyterHub on AKS with Azure Active Directory (AAD) authentication.

Prerequisites
Before beginning, you will need the following:

An Azure subscription
A resource group in Azure
An AKS cluster
An AAD tenant with at least one user account
The Azure CLI installed and configured on your local machine
Deploying JupyterHub
Create a new namespace in your AKS cluster for JupyterHub:
Copy code
kubectl create namespace jupyterhub
Install the JupyterHub Helm chart:
Copy code
helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
helm install jupyterhub jupyterhub/jupyterhub --version=0.9.1 --namespace jupyterhub
Create a new Service Principal in Azure for JupyterHub:
Copy code
az ad sp create-for-rbac --name jupyterhub-sp --role Contributor --scopes /subscriptions/<subscription-id>/resourceGroups/<resource-group>
Create a new secret in Kubernetes for the Service Principal:
Copy code
kubectl create secret generic jupyterhub-sp-secret --from-literal=client-id=<client-id> --from-literal=client-secret=<client-secret> --namespace jupyterhub
Update the JupyterHub Helm chart to use the new secret:
Copy code
helm upgrade jupyterhub jupyterhub/jupyterhub --version=0.9.1 --namespace jupyterhub --set singleuser.extraEnv[0].name=AZURE_CLIENT_ID --set singleuser.extraEnv[0].valueFrom.secretKeyRef.name=jupyterhub-sp-secret --set singleuser.extraEnv[0].valueFrom.secretKeyRef.key=client-id --set singleuser.extraEnv[1].name=AZURE_CLIENT_SECRET --set singleuser.extraEnv[1].valueFrom.secretKeyRef.name=jupyterhub-sp-secret --set singleuser.extraEnv[1].valueFrom.secretKeyRef.key=client-secret
Create a new AAD group for JupyterHub users:
Copy code
az ad group create --display-name "JupyterHub Users" --mail-nickname "jupyterhub-users"
Add users to the AAD group:
Copy code
az ad group member add --group "JupyterHub Users" --member-id <user-object-id>
Update the JupyterHub Helm chart to use the A


-->