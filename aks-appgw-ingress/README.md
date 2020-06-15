Deploying App Gateway Ingress Controller using ARM (Greenfield)
==============

Prerequisities
--------------

* [Install Azure CLI Preview Extension](https://github.com/Azure/azure-cli-extensions/tree/master/src/aks-preview)
* [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)

### Register AKS Ingress Application Gateway Add-on
```sh
az feature register --name AKS-IngressApplicationGatewayAddon --namespace Microsoft.Containerservice
```

### Check it's registered
It might take a few minutes for the status to show Registered. You can check on the registration status using the az feature list command:

```sh
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/AKS-IngressApplicationGatewayAddon')].{Name:name,State:properties.state}"
```
### Refesh the provider
```sh
az provider register --namespace Microsoft.ContainerService
```

### Install/Update aks-preview extension
```sh
az extension add --name aks-preview
az extension list
```
```sh
az extension update --name aks-preview
az extension list
```

### Create an resource group
```sh
az group create --name rg-example --location australiaeast
```

Deploy ARM Template
-------------------
```sh
az deployment group create -g rg-example --template-file template.json
```

Deploy a sample application using App Gateway Ingress Controller (AGIC)
-----------------------------------------------------------------------
```sh
az aks get-credentials -n testcluster -g rg-example
```
```sh
kubectl apply -f https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/docs/examples/aspnetapp.yaml
```

Check the application is reachable
----------------------------------
Application Gateway is now setup to service traffic on the AKS cluster, let's verify that is works. You'll need the IP address of the Ingress.
```sh
kubectl get ingress
```
Check that the sample application you created is up and running by either visiting the IP address of the Application Gateway that you got from running the above command or check with curl.
```sh
curl -Iv <ip address>
```

Diagnosing Issues
-----------------
* Experience any issues with executing the ARM template. Use --debug when running the command to log verbose output.


TODO
----
If you want to use an existing AppGw, or an existing vnet, or do advanced config on the vnet, then extra steps apply (mainly RBAC assignment for the MSI or Identity the AppGw ingress management uses).
For future reference when we look back at this in 6 months time:

Contributor on AppGwId [CONST_INGRESS_APPGW_APPLICATION_GATEWAY_ID],
Network Contributor on SubnetID [CONST_INGRESS_APPGW_SUBNET_ID]
OR Contributor on VNetId [result.agent_pool_profiles[0].vnet_subnet_id]

Resources
---------

* [App Gateway Ingress Controller](https://azure.github.io/application-gateway-kubernetes-ingress/)
* [Ingress Controller Add-on for AKS (Greenfield)](https://docs.microsoft.com/en-us/azure/application-gateway/tutorial-ingress-controller-add-on-new)
* [Ingress controller Add-on for AKS (Brownfield)](https://docs.microsoft.com/en-us/azure/application-gateway/tutorial-ingress-controller-add-on-existing)
* [Ingress Applicaiton Gateway configuration keys](https://github.com/Azure/azure-cli-extensions/blob/dcdb8b865cb95dbe0522210d749e2db528d0b89c/src/aks-preview/azext_aks_preview/_consts.py#L16)

