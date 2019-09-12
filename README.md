# Template for creating AKS cluster with App GW integration.
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Frp2343%2Faks-appgw%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

With the instructions in the previous section we created and configured a new Azure Kubernetes Service (AKS) cluster. We are now ready #to deploy to our new Kubernetes infrastructure. The instructions below will guide us through the proccess of installing the following 2 components on our new AKS:

Azure Active Directory Pod Identity - Provides token-based access to the Azure Resource Manager (ARM) via user-assigned identity.

Application Gateway Ingress Controller - This is the controller which monitors ingress-related events and actively keeps your Azure Application Gateway installation in sync with the changes within the AKS cluster.

Steps:
1. Configure kubectl to connect to Azure Kubernetes Cluster.

2. Add aad pod identity service to the cluster using the following command. This service will be used by the ingress controller. You can refer aad-pod-identity for more information.
RBAC enabled AKS cluster
kubectl create -f https://raw.githubusercontent.com/Azure/aad-pod-identity/master/deploy/infra/deployment-rbac.yaml

3. Install Helm and run the following to add application-gateway-kubernetes-ingress helm package:
RBAC enabled AKS cluster
kubectl create serviceaccount --namespace kube-system tiller-sa
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller-sa
helm init --tiller-namespace kube-system --service-account tiller-sa
helm repo add application-gateway-kubernetes-ingress https://appgwingress.blob.core.windows.net/ingress-azure-helm-package/
helm repo update

4. Edit helm-config.yaml and fill in the values for appgw and armAuth  
NOTE: The <identity-resource-id> and <identity-client-id> are the properties of the Azure AD Identity you setup in the previous section. You can retrieve this information by running the following command:

az identity show -g <resourcegroup> -n <identity-name>
    
5. Deploy guestbook application

a. Download guestbook-all-in-one.yaml

b. Deploy guestbook-all-in-one.yaml into your AKS cluster by running
kubectl apply -f guestbook-all-in-one.yaml

c. Expose service over HTTP :
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: guestbook
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: frontend
          servicePort: 80
          
          
This ingress will expose the frontend service of the guestbook-all-in-one deployment as a default backend of the Application Gateway.
Save the above ingress resource as ing-guestbook.yaml.

d. Deploy ing-guestbook.yaml by running:

kubectl apply -f ing-guestbook.yaml

Now the guestbook application should be available. You can check this by visiting the public address of the Application Gateway.


