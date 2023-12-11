# Troubleshooting Service Principal / Permission issues

A Service Principal must have Contributor permissions to the Vnet (Virtual Network) and User Access Administrator to the Resource Group assigned.

```
subscriptionID =
resourceGroup =
ARO_VNetName =
```

# Bring your Own SP Scoped

Based on: https://learn.microsoft.com/en-us/azure/openshift/howto-create-service-principal?pivots=aro-azurecli#create-and-use-a-service-principal

- How to check role applied to VNET

```
az role assignment list --scope /subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Network/virtualNetworks/<vnet-name> -o table

/// output
Principal                             Role                 Scope
------------------------------------  -------------------  -----------------------------------------------------------------------------------------------------------------------------------------
12345678-1234-1234-1234-123456789012  Network Contributor  /subscriptions/98765432-4321-4321-4321-987654321098/resourceGroups/hgomes-rg/providers/Microsoft.Network/virtualNetworks/myaro-vnet
```

- How to check role applied to Resource Group
```
az role assignment list --scope /subscriptions/<subscription-id>/resourceGroups/<resource-group-name> -o table
```

- Check service principal roles
```
az role assignment list --all -o table
```

- Check Role assignment in a subscription
```
az role assignment list --scope /subscriptions/<sub-id> --include-inherited -o table
```


# User Scoped

- Check my own user applications/SPs
```
az ad app list --show-mine -o table
```

- Check Role assignments for a user
```
az role assignment list --assignee <email> --include-groups -o table

//output
Principal               Role                       Scope
----------------------  -------------------------  ---------------------------------------------------
my-group                Contributor                /subscriptions/<sub-id>
my-group                User Access Administrator  /subscriptions/<sub-id>
```

# Cluster scoped

- Show the Service Principal associated to the ARO cluster
```
 az aro show -g $RESOURCEGROUP -n $CLUSTER --query servicePrincipalProfile.clientId -o tsv                                                                                                    
```

# Others

- RFE to reduce the permissions needed: https://issues.redhat.com/browse/RFE-1051 [No ETA yet]