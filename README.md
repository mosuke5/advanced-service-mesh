# Advanced Service Mesh Homework

## How to setup

### Deploy ServiceMesh control plane
Before installoing ServiceMesh control plane, you need to install some following operators.

- Jaeger operator
- ElasticSearch operator
- ServiceMesh operator

Then, you can install ServiceMesh control plane.

```
$ oc new-app bookretail-istio-system
$ oc apply manifests/istio-control-plane.yaml -n bookretail-istio-system
$ oc get pod -n bookretail-istio-system
NAME                                      READY   STATUS    RESTARTS   AGE
grafana-7df65c57d5-8tzqn                  2/2     Running   12         5d19h
istio-citadel-54fc4655b4-hb28n            1/1     Running   6          5d19h
istio-egressgateway-6bb97d479d-smwck      1/1     Running   2          29h
istio-galley-797746b78d-b5vxv             1/1     Running   6          5d19h
istio-ingressgateway-66fcccb49d-c4s9d     1/1     Running   0          140m
istio-pilot-567c947974-xjp8l              2/2     Running   4          29h
istio-policy-5fcfd5f79c-7rk44             2/2     Running   5          29h
istio-sidecar-injector-75577f4fdb-gz9vk   1/1     Running   6          5d19h
istio-telemetry-dbc897cbb-v7sv9           2/2     Running   5          29h
jaeger-778bcb6bf4-t8n6t                   2/2     Running   12         5d19h
kiali-78b6f44646-pqzq8                    1/1     Running   1          17h
prometheus-884ff6bf9-5mbh7                2/2     Running   12         5d19h
```

### Deploy bookinfo application

```
$ oc new-project bookinfo
$ oc apply -f https://raw.githubusercontent.com/istio/istio/1.4.0/samples/bookinfo/platform/kube/bookinfo.yaml -n bookinfo
```

### Add application to a member roll and 
You can add `bookinfo` project to service mesh member roll and inject annotations `sidecar.istio.io/inject: "true"` to all bookinfo services by using an ansible playbook.

```
$ ansible-playbook service-mesh-member-roll.yaml
...
PLAY RECAP ***************************************************************************************************
localhost   : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### Configure mTLS and expose your application
And you can configutre mTLS and expose bookinfo application to internet through istio ingress gateway.

```
$ ansible-playbook service-mesh-mtls.yaml
...
PLAY RECAP ****************************************************************************************************
localhost   : ok=7    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## Diagram
Following is a diagram of this network.

![overview](/images/overview.png)

![kiali-sample](/images/kiali-sample.png)