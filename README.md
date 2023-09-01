# Enable Pod Identity With Azure Keyvault

## Pre-Requisites:
- Ensure availibility of Helm3 client on your machine or Install the helm3 

## Steps To Deploy
#### . Install csi Secrets by Helm
```bash
helm repo add csi-secrets-store-provider-azure https://raw.githubusercontent.com/Azure/secrets-store-csi-driver-provider-azure/master/charts
helm install csi-secrets-store-provider-azure/csi-secrets-store-provider-azure --generate-name
```
#### . Assign your service principal Role
```bash
export AZURE_CLIENT_ID=""
export KEYVAULT_NAME=""
export SUBID=""
export KEYVAULT_RESOURCE_GROUP=""

az role assignment create --role Reader --assignee $AZURE_CLIENT_ID --scope /subscriptions/$SUBID/resourcegroups/$KEYVAULT_RESOURCE_GROUP/providers/Microsoft.KeyVault/vaults/$KEYVAULT_NAME
az keyvault set-policy -n $KEYVAULT_NAME --secret-permissions get --spn $AZURE_CLIENT_ID
az keyvault set-policy -n $KEYVAULT_NAME --key-permissions get --spn $AZURE_CLIENT_ID
```
#### . Create Kubernetes Secret
```bash
kubectl create secret generic secrets-store-creds --from-literal clientid=$AZURE_CLIENT_ID --from-literal clientsecret=<replace to your AZURE_CLIENT_SECRET>
```

#### . Deploy the custom SecretProviderClass
```bash
kubectl apply -f SecretProviderClass.yaml
```

#### . Deploy A Test pod 
```bash
kubectl apply -f nginx-pod-inline-volume-service-principal.yaml
```

#### . Validate Secret Mounting
```bash
kubectl exec -it nginx-secrets-store-inline ls /mnt/secrets-store/
kubectl exec -it nginx-secrets-store-inline env
```
You should be able see the secret are mounted to the pod and you should be able to see the secret in the pod environment variables.







