# General info about checking services

There are five types of Services [1]:

    ClusterIP (default): Internal clients send requests to a stable internal IP address.

    NodePort: Clients send requests to the IP address of a node on one or more nodePort values that are specified by the Service.

    LoadBalancer: Clients send requests to the IP address of a network load balancer.

    ExternalName: Internal clients use the DNS name of a Service as an alias for an external DNS name.

    Headless: You can use a headless service when you want a Pod grouping, but don't need a stable IP address.

- The NodePort type is an extension of the ClusterIP type. So a Service of type NodePort has a cluster IP address.

- The LoadBalancer type is an extension of the NodePort type. So a Service of type LoadBalancer has a cluster IP address and one or more nodePort values.

## Checking logs for services

- For adding HAProxy rules and binding
ns/openshift-ovn-kubernetes (oc logs <podname> --all-containers=true)
ns/openshift-sdn

- For service creation/deletion
ns/openshift-kube-controller-manager

## Type Loadbalancer
- Check if is accessible externally
```
curl <external-IP>:<targetPort>
```

- Check if IPTABLES added the rules
```
# iptables-save | grep <CLUSTER-IP-from-svc>
-A OVN-KUBE-NODEPORT -p tcp -m addrtype --dst-type LOCAL -m tcp --dport 31606 -j DNAT --to-destination 172.x.x.x:8080
-A OVN-KUBE-EXTERNALIP -d <External-IP>/32 -p tcp -m tcp --dport 8080 -j DNAT --to-destination 172.x.x.x:8080
```

- Check if pod is reachable from inside the node
```
curl <podIP>:<port>
```

## Type ClusterIP

Reachable **only** from inside K8S network
- Check if pod is reachable
```
// inside pod
curl <svc-cluster-ip>:<port>
curl <pod-ip>:<port>
curl <svc-name.namespace.svc.cluster.local>:<port>
```

### References
[1] https://cloud.google.com/kubernetes-engine/docs/concepts/service
