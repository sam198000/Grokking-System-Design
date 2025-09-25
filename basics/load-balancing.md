Load Balancing (LB)
====

Help scale horizontally across an ever-increasing number of servers.

## LB locations
- Between user and web server
- Between web servers and an internal platform layer (application servers, cache servers)
- Between internal platform layer and database

## Algorithms
- Least connection
- Least response time
- Least bandwidth
- Round robin
- Weighted round robin
- IP hash

## Implementation
- Smart clients
- Hardware load balancers
- Software load balancers


# Load Balancing (LB) Guide

A load balancer distributes network or application traffic across multiple servers, helping scale horizontally and improve reliability and availability.

---

## LB Locations

Load balancers can be deployed at several points in your system architecture:

1. **Between User and Web Server**
    - *Purpose*: Distribute incoming HTTP/S requests from clients (browsers/apps) to a pool of web servers.
    - *Examples*: A site like example.com uses an LB to split traffic among 10 identical web servers, keeping any server from being overloaded. If one server goes down, others handle the load.
    AWS Elastic Load Balancer (ELB/ALB), NGINX, HAProxy, F5 Networks (hardware), Google Cloud Load Balancer.
2. **Between Web Servers and Internal Platform Layer**
    - *Purpose*:  In multi-tier architectures, web servers may hand off requests to one of several application servers or cache nodes.
    Balances traffic between the web servers and application servers, cache servers, etc.
    - *Examples*: After SSL termination and routing by the web tier, requests requiring dynamic content are load balanced across backend app servers (e.g., Node.js or Spring Boot instances).
    Internalsoftware LBs (NGINX, Envoy), HAProxy, service mesh load balancing(Istio).
3. **Between Internal Platform Layer and Database**
    - *Purpose*: Routes queries to read replicas, manages connection pools for scaling databases.
      Especially in databases with read-replicas, load balancing ensures queries are routed to available and least-busy replicas.
    - *Examples*: MySQL read replicas: App server connections are balanced across multiple replicas, while writes always go to the primary.
      ProxySQL (MySQL), PgBouncer (PostgreSQL), Smart Client logic.

---

## Load Balancing Algorithms

Here are commonly used algorithms, with detailed explanations and examples:

### 1. Least Connection

- **Description:** New requests go to the server with the fewest active connections.
- **Best For:** Variable load per request, long-lived connections (e.g., chat).
- **Example:** Four backend servers; three have 10, 12, and 2 active connections—new request goes to the one with 2.
- **LBs Supporting:** NGINX, HAProxy.

**How it works:**

Routes each new request to the server with the fewest active connections.

**Use Cases:**

- Services with long-lived connections (e.g. chat, streaming).

**Example:**

Consider servers A, B, and C:

- Connections: A: 15, B: 10, C: 5
- Incoming request is routed to Server C.

**Supported By:** NGINX, HAProxy.

---

### 2. Least Response Time

- **Description:** Routes to the server with the fastest average response time.
- **Best For:** Ensures lowest latency for users; servers with slow responses get less traffic.
- **Example:** Web app where overloaded servers' latency increases—LB shifts load toward the servers still responding quickly.
- **LBs Supporting:** Application-aware LBs; advanced versions of HAProxy and NGINX

**How it works:**

Sends new requests to the server responding the quickest (lowest average response time).

**Use Cases:**

- APIs or services where latency impacts user experience.

**Example:**

- Server A: 120ms, Server B: 80ms, Server C: 200ms (latency averages)
- New requests assigned to Server B.

**How it's implemented:**

LB tracks server response time via health checks & metrics.

---

### 3. Least Bandwidth

- **Description:** Directs traffic to the server using the least bandwidth at that moment.
- **Best For:** High-bandwidth scenarios: file-download servers, video streaming.
- **Example:** If one server is serving a huge file, LB favors others until load normalizes.
- **LBs Supporting:** Specialized NGINX modules, commercial hardware LBs.

**How it works:**

Assigns new client to the server using the least bandwidth.

**Use Cases:**

- Streaming or download services with highly variable per-user data transfer requirements.

**Example:**

- Server A: 180 Mbps, Server B: 80 Mbps, Server C: 30 Mbps
- LB sends next download request to Server C.

**Supported By:** Specialized NGINX modules, some enterprise hardware LBs.

---

### 4. Round Robin

- **Description:** Simply cycles through the server list in order for each new request.
- **Best For:** Simple, stateless (every server handles equal-size tasks), homogeneous environments.
- **Example:** Three servers, requests distributed as 1-2-3-1-2-3...
- **LBs Supporting:** All major LBs (default option).

**How it works:**

Distributes requests in a fixed cyclic order, regardless of server load.

**Use Cases:**

- Stateless services, equally powerful servers.

**Example:**

With servers A, B, and C and incoming requests:

- 1st request: A, 2nd: B, 3rd: C, 4th: A, 5th: B, and so on.

**Supported By:** All major LBs, usually default.

---

### 5. Weighted Round Robin

- **Description:** Like round robin, but servers with higher weights get more requests relative to their weight.
- **Best For:** Heterogeneous environments (servers are not equal in power).
- **Example:** ServerA(weight=3), ServerB(weight=1) → 4 requests: A–A–A–B–... repeats.
- **LBs Supporting:** All major software/hardware LBs.

**How it works:**

Similar to round robin, but servers with higher weights receive more requests.

**Use Cases:**

- Heterogeneous hardware; one server can handle more traffic than others.

**Example:**

Server A (weight 3), B (weight 1), C (weight 2).

Request distribution:

- AAA B CC AAA B CC ...

**Supported By:** All major LBs (NGINX, HAProxy, etc.) via configuration.

---

### 6. IP Hash

- **Description:** Hashes the client’s IP address and routes to a particular server based on the hash.
- **Best For:** Stickiness; ensures the same user usually goes to the same backend server ("session stickiness").
- **Example:** For shopping cart sessions when not using centralized session stores.
- **LBs Supporting:** NGINX, HAProxy.

**How it works:**

Applies a hash function to a client’s IP address and routes the request to a server based on the result. Offers session stickiness.

**Use Cases:**

- Applications needing user session persistence without centralized session store.

**Example:**

- User with IP 192.168.88.11 is always routed to server B, determined by hash(192.168.88.11) % number_of_servers.

**Supported By:** NGINX, HAProxy.

---

## Implementation Types

### 1. Smart Clients

- **Description:** Client-side logic determines which server to send requests to, based on a dynamic server list from a service registry.
  
- **Best For:** Microservices, when you want to avoid a single point of failure.
  
- **Example:** Netflix Ribbon library (used with Eureka) for microservices. Each service node queries the registry and chooses backend servers (using round robin, random, weighted, etc.). [each app instance gets the list of servers from Eureka and balances load locally]
  
- **Pros & Cons:** No extra network hop; but pushes logic complexity to clients.


---


### 2. Hardware Load Balancers

- **Description:** Dedicated devices (appliances) for high-performance load balancing, often with built-in security and acceleration.
     Physical devices with specialized chips for routing and accelerating traffic; often include SSL offloading, DoS protection.
     
- **Examples/Vendors:** F5 BIG-IP, Citrix ADC, A10 Networks.
    
- **Use Case:** Financial or enterprise data centers needing maximum reliability/performance/latency/security needs.

- **Example:** A large bank might front its core banking web portals with F5 devices, routing all incoming HTTPS connections to internal server pools.


---


### 3. Software Load Balancers

- **Description:**  Runs on standard servers or cloud VMs; highly configurable and scalable.
    
- **Popular Options:** NGINX, HAProxy, Envoy, Traefik, Kubernetes Ingress Controllers.
    
- **Use Case:** Most web apps, microservices, API gateways, and cloud-native environments.

- **Example:** A startup deploys HAProxy on a VM to distribute traffic to its Dockerized web app services. Can be scaled or automated with Kubernetes ingress controllers (which are, themselves, software LBs).

---

## **Summary Table**

| **Scope/Location** | **Examples** | **Use Case** |
| --- | --- | --- |
| User <-> Web Server | AWS ALB, NGINX, Cloud LBs, F5 | Frontend website & mobile API entrypoint |
| Web <-> App Layer | Internal NGINX, HAProxy, Envoy, Istio | Microservices, business logic routing |
| App Layer <-> Database | ProxySQL, PgBouncer, custom client logic | Read replica sharding, master-slave writes |

| **Algorithm** | **Use Case Example** |
| --- | --- |
| Least Connection | Chat app backends with many long sessions |
| Least Response Time | User-facing web apps with variable latency |
| Least Bandwidth | File or video servers |
| Round Robin | Simple web frontends |
| Weighted Round Robin | Mixture of servers w/ different capacities |
| IP Hash | Session stickiness needed |

| **Implementation Type** | **Example Scenario** |
| --- | --- |
| Smart Clients | Service-to-service inside a microservices architecture |
| Hardware LB | Data center ingress for banking/finance |
| Software LB | Kubernetes Ingress, DIY cloud frontends, general websites |

---

## Real-world Example

**Scenario:**

*A growing e-commerce platform wants to scale reliably, optimize user experience, and ensure high availability.*

**Design:**

1. **User → Web LB (ALB in AWS):**
    
    Handles HTTP/S requests from users; uses Round Robin or Least Connection to distribute load among several NGINX web servers.
    
2. **Web Servers → App Layer LB (HAProxy):**
    
    Distributes traffic among Spring Boot application servers using Least Response Time, ensuring quick checkout, search responses.
    
3. **App Layer → Database LB (ProxySQL):**
    
    Handles routing to MySQL read replicas for GET/read queries, while writes go to the primary database.
    
4. **If session persistence is needed:**
    
    Web LB uses an IP Hash algorithm so the same user gets routed to the same web server for sticky sessions.

## **(Scenario)**

Suppose you launch an **e-commerce web app**:

- **Clients:** Browsers/mobile apps connect to **`shop.example.com`**
- **LB1 (User ↔ Web):** AWS Application Load Balancer uses round robin to distribute traffic among NGINX instances
- **LB2 (Web ↔ App):** Internal HAProxy load balancer uses least connection for Java microservices responsible for order processing (since order processing is complex and time-varying)
- **LB3 (App ↔ DB):** PgBouncer (with custom smart client routing) distributes read queries across three read replicas, while all writes go to the primary.

If the cart service stores session locally and you want users to keep getting the same server, NGINX uses IP Hashing at the first LB layer.
    
# 

- **Location:** Place LBs where scaling or failover is needed (often at user entry, between internal tiers, and for database sharding)
- **Algorithm:** Pick what suits your workload pattern (stateless → round robin, uneven loads → least connection, need stickiness → IP Hash)
- **Implementation:** Choose software LBs for flexibility/cloud; hardware for max performance/security; smart clients for microservices.
  
---

## References

- [NGINX Load Balancing Docs](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)
- [HAProxy Algorithms](https://www.haproxy.com/documentation/hapee/latest/load-balancing/load-balancing-algorithms/)
- [AWS Load Balancer Types](https://aws.amazon.com/elasticloadbalancing/)
- [Load Balancing Wikipedia](https://en.wikipedia.org/wiki/Load_balancing_(computing))

---
