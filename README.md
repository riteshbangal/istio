# Getting Started with Istio
Reference: https://istio.io/latest/docs/setup/getting-started

### Install 'istioctl'
 $ curl -L https://istio.io/downloadIstio | sh -
 
 $ export PATH="$PATH:/Users/riteshbangal/Geeks/workshop/istio/istio-1.17.2/bin"
 
 $ istioctl version
 
 $ istioctl x precheck

### Install 'istio'
$ istioctl install --set profile=demo -y

$ kubectl get pod -n istio-system

$ kubectl get service -n istio-system

### Inject 'istio'
$ kubectl create namespace istio-poc

$ kubectl label namespace istio-poc istio-injection=enabled

$ kubectl describe namespace istio-poc | grep istio-injection

### Deploy the sample application
$ kubectl apply -f bookinfo.yaml

$ kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"

### Open the application to outside traffic
$ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml

$ istioctl analyze

### Determining the ingress IP and ports
$ kubectl get svc istio-ingressgateway -n istio-system

$ export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')

$ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')

$ export INGRESS_HOST=127.0.0.1

$ export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT

$ echo "$GATEWAY_URL"

$ echo "http://$GATEWAY_URL/productpage"


# Access External URL
$ kubectl exec -it deployment/ratings-v1 bash

$ curl -sI http://ec2-3-125-207-155.eu-central-1.compute.amazonaws.com


$ export SOURCE_POD=$(kubectl get pod -l app=ratings -o jsonpath='{.items..metadata.name}')

$ kubectl exec "$SOURCE_POD" -c ratings -- curl -sI http://ec2-3-125-207-155.eu-central-1.compute.amazonaws.com | grep  "HTTP/"

$ kubectl exec "$SOURCE_POD" -c ratings -- curl -sIS https://www.google.com | grep "HTTP/"

$ kubectl exec "$SOURCE_POD" -c ratings -- curl -sIk https://edition.cnn.com | grep "HTTP/"


To check the Istio mode specifically in the meshConfig.outboundTrafficPolicy, you can use the following command:

$ kubectl get configmap istio -n istio-system -o jsonpath='{.data.mesh}' | grep mode
If the output is outboundTrafficPolicy: REGISTRY_ONLY, it means Istio is running in "permissive" mode. If the output is outboundTrafficPolicy: ALLOW_ANY, it means Istio is running in "strict" mode.

$ kubectl get configmap istio -n istio-system -o yaml | sed 's/mode: ALLOW_ANY/mode: REGISTRY_ONLY/g' | kubectl replace -n istio-system -f -

$ kubectl get configmap istio -n istio-system -o yaml > istio-configmap.yaml


$ kubectl get configmap istio -n istio-system -o yaml | sed 's/mode: REGISTRY_ONLY/mode: ALLOW_ANY/g' | kubectl replace -n istio-system -f -

$ kubectl get configmap istio -n istio-system -o yaml > istio-configmap.yaml



