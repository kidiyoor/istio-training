## Istio Training

### Without istio

Deploy first service
```
kubectl apply -f training/services/service-productpage/kitab-productpage-service.yaml
```

Deploy second service
```
kubectl apply -f training/services/service-details/kitab-details-service.yaml
```

productpage -> details

productpage code : `https://github.com/istio/istio/blob/master/samples/bookinfo/src/productpage/productpage.py`

details code : `https://github.com/istio/istio/blob/master/samples/bookinfo/src/details/details.rb`

Discussion

How did kubernetes help ?
How is istio different from kubernetes ?
Where does k8s responsibility end ? Should k8s extend its features to do all that istio does ?

Demerits

How to enable tracing ?
How to enable monitoring to my services ?
How to secure the communication ?
How to dynamically apply l7 policy / l3 routing for service to service communication ?

### Debugging

```
kubectl get svc
kubectl get deployments
kubectl get pods
kubectl describe pods <pods_id>
kubectl exec -it <pods_id> bash
```
`-n` for namespace

### Install Istio

To give admin permission to current user
```
kubectl create clusterrolebinding cluster-admin-binding \
    --clusterrole=cluster-admin \
    --user=$(gcloud config get-value core/account)
```
Install Istio
```
kubectl apply -f install/kubernetes/istio-demo.yaml
```

What it means by installing istio on my k8s cluster ?

```
kubectl get svc -n istio-system
kubectl get pods -n istio-system
kubectl get cm -n istio-system
kubectl get secrets -n istio-system
kubectl get secrets
```

Enable auto-injection on my default namespace
```
kubectl label namespace default istio-injection=enabled
```


### Re-Deploy the app with Istio
Clean up
```
kubectl delete -f training/services/service-productpage/kitab-productpage-service.yaml
kubectl delete -f training/services/service-details/kitab-details-service.yaml
```

Deploy first service
```
kubectl apply -f training/services/service-productpage/kitab-productpage-service.yaml
```

Deploy second service
```
kubectl apply -f training/services/service-details/kitab-details-service.yaml
```

Check if sidecar is injected
```
kubectl describe pods <pods_id>
```

### Debugging Istio
```
kubectl get pods -n istio-system
kubectl exec -it -c istio-proxy <pod_id> bash
curl localhost:15000/logging?
curl localhost:15000/listeners
curl localhost:15000/routes
kubectl logs -c istio-proxy <pod_id>
kubectl edit cm istio -n istio-system
```


### Pilot Config vs Mixer Config

By now you should have understood that by introducing istio in your architecture
you were able to capture all the communication between all your services so that
you can apply some kind of rules on these communication

If you look closer on what these rules might look like -

Assumptions and Notes: We deal at l4 and above. Which means Istio do not see/modify content of individual network
packets. Enabling l3 network connectivity from one VM to another (or POD to POD) is provided by the platform. Now that we established basic networking - can Istio see/modify anything that is l4 and beyond ? No! Istio proxy ie. envoy does not capture UDP as yet. Which means Istio cannot apply rules on DNS traffic or UDP traffic.

1. security - encryption (Using 'session layer' of OSI stack ie TLS)
2. identify the protocol and capture standards provided by these protocols and
   make desicions eg. Port, HTTP headers, TCP source IP, TCP destination IP
3. identify the protocol and capture non-standard part of the protocol and make desicions eg. HTTP
   payload
4. record everything
5. ??

Currently Istio does 1. 2. and 4. and not 3.
There are lot of reasons for not doing 3. Here is few - https://github.com/istio/istio/issues/4460#issuecomment-398209912

Pilot is responsible for configuring routing related rules for service-service communication. eg. re-routing, canary, fault injection, retry, circuit breaker
Mixer is responsible for configuring policy enforcment for dynamic run time traffic. eg. authentication, apikey verification, analytics

Sample routing configuration

Talk about Virtualservice and Destinationrule

TODO

Sample mixer configuration

Talk about handler, instance, rule

TODO

### Security

Talk about server side, client side mTLS configuration

TODO

Show certs

TODO
