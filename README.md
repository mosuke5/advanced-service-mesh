# Advanced Service Mesh Homework

## How to setup

### Deploy ServiceMesh control plane(for istio admin)
Before installoing ServiceMesh control plane, you need to install some following operators.

- Jaeger operator
- ElasticSearch operator
- ServiceMesh operator
- Kiali operator

```
$ oc login -u admin -p xxxx
$ oc apply -f manifests/operators.yaml
subscription.operators.coreos.com/elasticsearch-operator created
subscription.operators.coreos.com/jaeger-product created
subscription.operators.coreos.com/kiali-ossm created
subscription.operators.coreos.com/servicemeshoperator created

$ oc get subscription -n openshift-operators
NAME                     PACKAGE                  SOURCE             CHANNEL
elasticsearch-operator   elasticsearch-operator   redhat-operators   4.4
jaeger-product           jaeger-product           redhat-operators   stable
kiali-ossm               kiali-ossm               redhat-operators   stable
servicemeshoperator      servicemeshoperator      redhat-operators   stable
```

Then, you can install ServiceMesh control plane.

```
$ oc new-project bookretail-istio-system
$ oc apply -f manifests/istio-control-plane.yaml -n bookretail-istio-system
$ oc policy add-role-to-user edit user1 -n bookretail-istio-system
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

### Deploy bookinfo application(for application users)
Deploy bookinfo application. In this time, bookinfo application does not include service mesh conpornents like envoy.

```
$ oc login -u user1 -p xxxx
$ oc new-project bookinfo
$ oc apply -f https://raw.githubusercontent.com/istio/istio/1.4.0/samples/bookinfo/platform/kube/bookinfo.yaml -n bookinfo
$ oc get pod -n bookinfo
NAME                              READY   STATUS    RESTARTS   AGE
details-v1-78d78fbddf-n4vfk       1/1     Running   0          42s
productpage-v1-596598f447-4sc48   1/1     Running   0          40s
ratings-v1-6c9dbf6b45-jl8p8       1/1     Running   0          42s
reviews-v1-7bb8ffd9b6-h8gfp       1/1     Running   0          41s
reviews-v2-d7d75fff8-z896x        1/1     Running   0          41s
reviews-v3-68964bc4c8-hh9gw       1/1     Running   0          41s
```

### Add application to a member roll and 
You can add `bookinfo` project to service mesh member roll and inject annotations `sidecar.istio.io/inject: "true"` to all bookinfo services by using an ansible playbook.
After executing this playbook, you can see pod number of each bookinfo applications increased to 2.

```
$ ansible-playbook service-mesh-member-roll.yaml
...
PLAY RECAP ***************************************************************************************************
localhost   : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

$ oc get ServiceMeshMemberRoll default -o yaml -n bookretail-istio-system | grep -A 1 configuredMembers
  configuredMembers:
  - bookinfo

$ oc get pod -n bookinfo
NAME                             READY   STATUS    RESTARTS   AGE
details-v1-6657b8bdf-vgppk       2/2     Running   0          2m35s
productpage-v1-597b74b4c-ndn6j   2/2     Running   0          2m29s
ratings-v1-66cddbfb8f-4mhk5      2/2     Running   0          2m24s
reviews-v1-6788566f98-x4zpl      2/2     Running   0          2m19s
reviews-v2-7c4bffdcc4-4r7s6      2/2     Running   0          2m13s
reviews-v3-69b6d8786-jqqfb       2/2     Running   0          2m7s
```

### Configure mTLS and expose your application
And you can configutre mTLS and expose bookinfo application to internet through istio ingress gateway.

```
$ ansible-playbook service-mesh-mtls.yaml
...
PLAY RECAP ****************************************************************************************************
localhost   : ok=7    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

$ curl -I -k https://productpage.bookinfo.apps.<your-base-domain>
HTTP/2 200
content-type: text/html; charset=utf-8
content-length: 1683
server: istio-envoy
date: Sun, 14 Jun 2020 11:27:44 GMT
x-envoy-upstream-service-time: 2

$ curl -I -k https://productpage.bookinfo.apps.<your-base-domain>/productpage\?u=normal
HTTP/2 200
content-type: text/html; charset=utf-8
content-length: 5183
server: istio-envoy
date: Sun, 14 Jun 2020 13:33:08 GMT
x-envoy-upstream-service-time: 681
```

## Diagram
Following is a diagram of this network.

![overview](/images/overview.png)

![kiali-sample](/images/kiali-sample.png)
