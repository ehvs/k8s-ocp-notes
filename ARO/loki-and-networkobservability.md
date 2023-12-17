0. Create a New Storage Account, Storage Class as per : https://learn.microsoft.com/en-us/azure/openshift/howto-create-a-storageclass#before-you-begin 

1. Create a new Blob service.
In the Azure Portal > Storage Account > select the newly created > Side right bar "Data storage -> Containers" > Click "+ Container" > Add a "Name" and Create.

Eg. name: container-loki

2. Create a new machineset with 2 replicas, using large vmSizes, to be able to allocate the Loki instance.
Copy an existing machineset, and replace the values of "name" and label "machine.openshift.io/cluster-api-machineset", use a new "vmSize" too. In this example, I'm using 'Standard_E16s_v5', and added a label "node-role.kubernetes.io/worker: "loki"" in case I want to add nodeSelectors in the LokiStack components.

```
apiVersion: machine.openshift.io/v1beta1
kind: MachineSet
metadata:
  name: new-worker-westeurope1
  namespace: openshift-machine-api
  labels:
    machine.openshift.io/cluster-api-cluster: hgomes-cluster-l4jlr
    machine.openshift.io/cluster-api-machine-role: worker
    machine.openshift.io/cluster-api-machine-type: worker
spec:
  replicas: 2
  selector:
    matchLabels:
      machine.openshift.io/cluster-api-cluster: hgomes-cluster-l4jlr
      machine.openshift.io/cluster-api-machineset: new-worker-westeurope1
  template:
    metadata:
      labels:
        machine.openshift.io/cluster-api-cluster: hgomes-cluster-l4jlr
        machine.openshift.io/cluster-api-machine-role: worker
        machine.openshift.io/cluster-api-machine-type: worker
        machine.openshift.io/cluster-api-machineset: new-worker-westeurope1
        node-role.kubernetes.io/worker: "loki"
    spec:
      lifecycleHooks: {}
      metadata: {}
      providerSpec:
        value:
          osDisk:
            diskSettings: {}
            diskSizeGB: 128
            managedDisk:
              storageAccountType: Premium_LRS
            osType: Linux
          networkResourceGroup: hevs-rg
          publicLoadBalancer: hgomes-cluster-l4jlr
          userDataSecret:
            name: worker-user-data
          vnet: aro-vnet
          credentialsSecret:
            name: azure-cloud-credentials
            namespace: openshift-machine-api
          diagnostics: {}
          zone: '1'
          publicIP: false
          resourceGroup: aro-kxd5yl2g
          kind: AzureMachineProviderSpec
          location: westeurope
          vmSize: Standard_E16s_v5
          image:
            offer: aro4
            publisher: azureopenshift
            resourceID: ''
            sku: aro_412
            version: 412.86.20230503
          acceleratedNetworking: true
          subnet: worker-subnet
          apiVersion: machine.openshift.io/v1beta1
```

3. Create a new project.

```
oc new-project netobserv
oc project netobserv
```

4. Create a secret. Pull the Account key from Azure Portal > Storage Account > $YourNewSA > Security + Networking > Access keys
```
oc create secret generic logging-loki-azure --from-literal=container="<add-blob-container-name>" --from-literal=environment="AzureGlobal" --from-literal=account_name="<add-storageaccount-name-here>" --from-literal=account_key="<addhere>"
```

- Note: Environment is usually `AzureGlobal`

5. Template to:
- Create the LokiStack. Use the newly created "storageClass" (in this example 'azure-file'), and the secret name from previous step.
- Create  the Cluster Roles as per : https://docs.openshift.com/container-platform/4.12/network_observability/installing-operators.html#network-observability-auth-mutli-tenancy_network_observability

```
---
apiVersion: loki.grafana.com/v1
kind: LokiStack
metadata:
  name: loki
  namespace: observe   
spec:
  size: 1x.extra-small
  storage:
    schemas:
    - version: v12
      effectiveDate: '2022-06-01'
    secret:
      name: logging-loki-azure
      type: azure
  storageClassName: azure-file  
  tenants:
    mode: openshift-network
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: netobserv-reader    
rules:
- apiGroups:
  - 'loki.grafana.com'
  resources:
  - network
  resourceNames:
  - logs
  verbs:
  - 'get'
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: netobserv-writer-flp
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: netobserv-writer
subjects:
- kind: ServiceAccount
  name: flowlogs-pipeline    
  namespace: netobserv
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: netobserv-writer-flp
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: netobserv-writer
subjects:
- kind: ServiceAccount
  name: flowlogs-pipeline    
  namespace: netobserv
```

6. Wait stabilize the Loki pods.
7 Create the Flow Collector. Replace URL - statusURL - namespace, with the proper project name if needed.
```
apiVersion: flows.netobserv.io/v1alpha1
kind: FlowCollector
metadata:
  name: cluster
spec:
  agent:
    ebpf:
      sampling: 0 #Capture all traffic in the network by turning off sampling.
    type: EBPF
  deploymentModel: DIRECT
  consolePlugin:
    register: true
  loki:
    tenantID: network
    url: 'https://loki-gateway-http.netobserv.svc:8080/api/logs/v1/network'
    authToken: FORWARD
    tls:
      caCert:
        certFile: service-ca.crt
        name: loki-gateway-ca-bundle
        type: configmap
      enable: true
      insecureSkipVerify: true
    statusUrl: 'https://loki-query-frontend-http.netobserv.svc:3100/'
  namespace: netobserv
```
