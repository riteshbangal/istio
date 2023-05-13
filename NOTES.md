# Virtual services

#### What is virtual services? Why use virtual services? How it works?
- Without virtual services, Envoy distributes traffic using round-robin load balancing between all service instances.
- With a virtual service, you can specify traffic behavior for one or more hostnames. You use routing rules in the virtual service that tell Envoy how to send the virtual service’s traffic to appropriate destinations. Route destinations can be different versions of the same service or entirely different services.
- In some cases you also need to configure destination rules to use these features, as these are where you specify your service subsets. Specifying service subsets and other destination-specific policies in a separate object lets you reuse these cleanly between virtual services.
- The spec.hosts field lists the virtual service’s hosts - in other words, the user-addressable destination or destinations that these routing rules apply to. This is the address or addresses the client uses when sending requests to the service.
- The virtual service hostname can be an IP address, a DNS name, or, depending on the platform, a short name (such as a Kubernetes service short name) that resolves, implicitly or explicitly, to a fully qualified domain name (FQDN). You can also use wildcard ("*") prefixes, letting you create a single set of routing rules for all matching services. Virtual service hosts don’t actually have to be part of the Istio service registry, they are simply virtual destinations. This lets you model traffic for virtual hosts that don’t have routable entries inside the mesh.
- The http section contains the virtual service’s routing rules
- The route section’s destination field specifies the actual destination for traffic that matches this condition.
- Unlike the virtual service’s host(s), the destination’s host must be a real destination that exists in Istio’s service registry or Envoy won’t know where to send traffic to it. This can be a mesh service with proxies or a non-mesh service added using a service entry.
- Using short names like this only works if the destination hosts and the virtual service are actually in the same Kubernetes namespace. Because using the Kubernetes short name can result in misconfigurations, we recommend that you specify fully qualified host names in production environments.
- Routing rules are evaluated in sequential order from top to bottom, with the first rule in the virtual service definition being given highest priority.

# Destination rules
- DestinationRule defines policies that apply to traffic intended for a service after routing has occurred.
- Along with virtual services, destination rules are a key part of Istio’s traffic routing functionality. You can think of virtual services as how you route your traffic to a given destination, and then you use destination rules to configure what happens to traffic for that destination. Destination rules are applied after virtual service routing rules are evaluated, so they apply to the traffic’s “real” destination.
- In particular, you use destination rules to specify named service subsets, such as grouping all a given service’s instances by version.
- By default, Istio uses a round-robin load balancing policy, where each service instance in the instance pool gets a request in turn. Istio also supports the following models, which you can specify in destination rules for requests to a particular service or service subset.

#### Is it possible to have a DestinationRule object without a VirtualService object in Istio?
- Yes, it is possible to have a DestinationRule object without a VirtualService object in Istio. The DestinationRule and VirtualService objects are separate resources in Istio, and while they are often used together to configure traffic routing and behavior, they can also be used independently based on your specific requirements.

- The DestinationRule resource allows you to define traffic policies, load balancing settings, connection pool settings, and other configuration options for a specific service or subset of services. It provides fine-grained control over the behavior of traffic sent to those services.

- The VirtualService resource, on the other hand, is responsible for defining routing rules and traffic behavior. It allows you to control how incoming traffic is routed to different destination services based on matching criteria such as request headers, paths, or source IPs.

- For example, you might use a DestinationRule to specify the load balancing algorithm, connection pool settings, and TLS configuration for a service, without requiring any specific routing rules. In such cases, you can omit the VirtualService object.

