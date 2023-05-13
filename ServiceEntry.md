# Accessing External Services
Because all outbound traffic from an Istio-enabled pod is redirected to its sidecar proxy by default, accessibility of URLs outside of the cluster depends on the configuration of the proxy. By default, Istio configures the Envoy proxy to pass through requests for unknown services. Although this provides a convenient way to get started with Istio, configuring stricter control is usually preferable.

#### Before you begin
$ kubectl exec -it deployment/ratings-v1 bash

$ curl -sI http://ec2-3-125-207-155.eu-central-1.compute.amazonaws.com

$ export SOURCE_POD=$(kubectl get pod -l app=ratings -o jsonpath='{.items..metadata.name}')

#### Access External URL
Allow the Envoy proxy to pass requests through to services that are not configured inside the mesh.
$ kubectl exec "$SOURCE_POD" -c ratings -- curl -sI http://ec2-3-125-207-155.eu-central-1.compute.amazonaws.com | grep  "HTTP/"


## Controlled access to external services
To check the Istio mode specifically in the meshConfig.outboundTrafficPolicy, you can use the following command:

$ kubectl get configmap istio -n istio-system -o jsonpath='{.data.mesh}' | grep mode
If the output is outboundTrafficPolicy: REGISTRY_ONLY, it means Istio is running in "permissive" mode. If the output is outboundTrafficPolicy: ALLOW_ANY, it means Istio is running in "strict" mode.

#### Change to the blocking-by-default policy
$ kubectl get configmap istio -n istio-system -o yaml | sed 's/mode: ALLOW_ANY/mode: REGISTRY_ONLY/g' | kubectl replace -n istio-system -f -

$ kubectl get configmap istio -n istio-system -o yaml > istio-configmap.yaml

$ kubectl exec "$SOURCE_POD" -c ratings -- curl -sI http://ec2-3-125-207-155.eu-central-1.compute.amazonaws.com | grep  "HTTP/"
This returns --> HTTP/1.1 502 Bad Gateway

$ kubectl apply -f MyWebApp-ServiceEntry.yaml
$ kubectl exec "$SOURCE_POD" -c ratings -- curl -sI http://ec2-3-125-207-155.eu-central-1.compute.amazonaws.com | grep  "HTTP/"
Now, this returns --> HTTP/1.1 200 OK

#### Change back to the allow-any-default policy
$ kubectl get configmap istio -n istio-system -o yaml | sed 's/mode: REGISTRY_ONLY/mode: ALLOW_ANY/g' | kubectl replace -n istio-system -f -

$ kubectl get configmap istio -n istio-system -o yaml > istio-configmap.yaml



#### Completely bypass the Envoy or Direct access to external services
If you want to completely bypass Istio for a specific IP range, you can configure the Envoy sidecars to prevent them from intercepting external requests. To set up the bypass, change either the global.proxy.includeIPRanges or the global.proxy.excludeIPRanges configuration option and update the istio-sidecar-injector configuration map using the kubectl apply command. This can also be configured on a pod by setting corresponding annotations such as 'traffic.sidecar.istio.io/includeOutboundIPRanges'. After updating the istio-sidecar-injector configuration, it affects all future application pod deployments.


### Understanding what happened
In this task you looked at three ways to call external services from an Istio mesh:

1. Configuring Envoy to allow access to any external service.

2. Use a service entry to register an accessible external service inside the mesh. This is the recommended approach.

3. Configuring the Istio sidecar to exclude external IPs from its remapped IP table.

The first approach directs traffic through the Istio sidecar proxy, including calls to services that are unknown inside the mesh. When using this approach, you can’t monitor access to external services or take advantage of Istio’s traffic control features for them. To easily switch to the second approach for specific services, simply create service entries for those external services. This process allows you to initially access any external service and then later decide whether or not to control access, enable traffic monitoring, and use traffic control features as needed.

The second approach lets you use all of the same Istio service mesh features for calls to services inside or outside of the cluster. In this task, you learned how to monitor access to external services and set a timeout rule for calls to an external service. Using Istio ServiceEntry configurations, you can access any publicly accessible service from within your Istio cluster. This section shows you how to configure access to an external HTTP service.


The third approach bypasses the Istio sidecar proxy, giving your services direct access to any external server. However, configuring the proxy this way does require cluster-provider specific knowledge and configuration. Similar to the first approach, you also lose monitoring of access to external services and you can’t apply Istio features on traffic to external services.


Reference: https://istio.io/latest/docs/tasks/traffic-management/egress/egress-control 

