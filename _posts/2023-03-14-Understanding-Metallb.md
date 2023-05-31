---
title: 'Understanding Metallb'
date: 2023-03-14
permalink: /posts/2013/08/understanding-metallb/
tags:
  - Kubernetes
  - Load-Balancer
  - Metallb
---

This post aims to provide basis to understand metallb

Introduction to Load Balancing 
======
Kubernetes is an open-source container orchestration platform that allows you to deploy, manage, and scale containerized applications. When running multiple instances of an application, Kubernetes distributes the workload across different nodes in the cluster. However, as the number of replicas increases, efficiently managing the incoming network traffic becomes challenging.

This is where load balancing comes into the picture. Load balancing is a technique that evenly distributes network traffic across multiple backend services or instances, ensuring efficient resource utilization and preventing any single component from becoming a bottleneck.

In a Kubernetes environment, load balancing is essential for several reasons:

1. Scalability: Load balancing ensures that traffic is evenly distributed among these replicas, enabling efficient utilization of resources and accommodating increased demand.

2. High Availability: By distributing traffic across multiple instances, load balancing helps improve the availability of applications. If one replica becomes unavailable or experiences issues, the load balancer automatically redirects traffic to healthy replicas, ensuring continuous service availability.

3. Traffic Management: Load balancers in Kubernetes provide advanced traffic management capabilities. You can define routing rules, prioritize traffic, perform SSL termination, and manage traffic flow to different versions of your application, facilitating seamless deployments and updates.

In the Kubernetes ecosystem, there are various load balancing options available, including the built-in options and external solutions. One popular external solution is MetalLB, which is a software load balancer implementation specifically designed for Kubernetes clusters.

MetalLB
======
MetalLB operates at the network layer and works by advertising a set of virtual IP addresses from a pool to external networks. When an external client sends a request to one of these virtual IP addresses, MetalLB selects a backend service within the cluster and forwards the traffic to it.

Here's a high-level overview of how MetalLB works:

**Configuration**: MetalLB requires a configuration to define the behavior of the load balancer. This configuration specifies the IP address range, protocols (Layer 2 or Layer 3), and the desired behavior for load balancing. It can be configured using either ConfigMaps or custom resources. It is compatible with a wide range of network environments and infrastructure.

**IP Address Allocation**: MetalLB allocates IP addresses from the configured range to services within the cluster. It can operate in two modes: Layer 2 mode and BGP mode.
  - Layer 2 Mode: In this mode, MetalLB responds to ARP requests for the virtual IP addresses it manages. It uses Address Resolution Protocol (ARP) to announce the presence of the IP address to the local network, allowing external clients to reach the services. Layer 2 mode in MetalLB is relatively easy to configure and deploy. But it relies on ARP, which can have performance issues in large-scale deployments or environments with high traffic loads. MetalLB requires the virtual IP addresses to be within the same subnet as the Kubernetes nodes. In some cases, Layer 2 mode might require specific network configurations to ensure proper functioning. For example, it may require enabling proxy ARP on the local network switches or configuring network policies to allow ARP traffic

   - BGP Mode: In this mode, MetalLB speaks the Border Gateway Protocol (BGP) to advertise the virtual IP addresses to external routers. This mode is suitable for more advanced networking setups and requires integration with a BGP-speaking router. BGP mode involves more complex setup and configuration compared to Layer 2 mode. It relies on external BGP-speaking routers or network infrastructure to function properly. It may have provider-specific requirements and limitations.

**Load Balancing**: Once the IP addresses are allocated, MetalLB continuously monitors the health of the backend services within the cluster. It uses Kubernetes' service endpoints to determine the available endpoints for each service. MetalLB then uses load balancing algorithms, such as round-robin or least connections, to distribute the incoming traffic across these endpoints.

**Service Discovery**: MetalLB integrates with the Kubernetes service discovery mechanisms. When a service is created or updated, MetalLB automatically assigns an external IP address to it. This enables external clients to reach the service without any manual configuration.

**High Availability**: MetalLB ensures high availability by constantly monitoring the health of backend services. If a backend service becomes unavailable, MetalLB stops directing traffic to it and redirects requests to other healthy instances. This helps in maintaining the availability and reliability of the applications.

Concrete example
======

**Step 1**: Install and Configure MetalLB:
- Install MetalLB in your Kubernetes cluster by following the [official installation instructions](https://metallb.universe.tf/installation/) provided by the MetalLB project.
- Configure MetalLB in Layer 2 mode by creating a custom resource with the desired IP address range for load balancing. For example, you can specify a range of IP addresses from your network subnet that will be used by MetalLB to allocate virtual IP addresses.
<pre>
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: my-app-adv
  namespace: metallb-system
</pre>
<pre>
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: my-app-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.65.69-192.168.65.70
</pre>

**Step 2**: Create Backend Services:
- Define the backend services in your Kubernetes cluster that you want to expose externally. These services can be a set of pods or deployments running your application.

**Step 3**: Create a LoadBalancer Service:
- Create a Kubernetes Service resource of type LoadBalancer to expose your backend services externally. This is where MetalLB comes into play.
- Specify the desired ports, protocols, and selectors in the Service definition to match your backend services.
- MetalLB will allocate a virtual IP address from the configured IP address range and assign it to the LoadBalancer Service.
<pre>
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: my-app
</pre>

**Step 4**: Access the LoadBalancer Service:
- MetalLB, in Layer 2 mode, responds to ARP requests for the virtual IP address it allocated for the LoadBalancer Service.
- Once MetalLB receives the ARP request, it maps the virtual IP address to the backend services and forwards the traffic accordingly.
- External clients can now access the LoadBalancer Service using the allocated virtual IP address, and MetalLB will distribute the traffic to the backend services. In our case on 192.168.65.69:80

Resources
======
[https://metallb.universe.tf/concepts/](https://metallb.universe.tf/concepts/)
