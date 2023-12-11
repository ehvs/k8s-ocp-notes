# Private Ingress Controller (IC)

Network diagram: https://learn.microsoft.com/en-us/azure/openshift/concepts-networking

## How Private Ingress Controller works in Azure?

When a CR Ingress Controller is created, it will add a backend pool in the **Internal** Load Balancer for Azure for both masters and workers.
This is because the internal LB needs to be able to route the traffic for ingress to where the router pods run, which are the worker nodes.

Having an ingress controller - like the default one - will create a LB service with annotations which will cause the provisionning of Frontend IP configurations / LB rules etc on the internal lb and for the Private Ingress Controller, also means: having the workers in the Internal LB backend pool.

## What is Private Ingress in ARO?
Private ingress means that the ingress is accessible from the internal load balancer (and which has no public IP)

# Attention

Public API and private ingress will mean that you will need a bastion host or equivalent way to connect to the cluster (api is public but ingress, and so the authentication endpoints as well as console will be private)

### To create a cluster with ONLY Private Ingress, the following flags make the trick.
```
--ingress-visibility Privte --domain foo.example.com
```
- Microsoft documentation for Private cluster: https://learn.microsoft.com/en-us/azure/openshift/howto-create-private-cluster-4x#create-the-cluster

### To create an ADDITIONAL Ingress Controller
Simply create another Ingress Controller Custom Resource:
https://docs.openshift.com/container-platform/4.12/networking/ingress-operator.html#nw-ingress-setting-internal-lb_configuring-ingress


