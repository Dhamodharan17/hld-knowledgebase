# Index of system design topics
---

> Summaries of various system design topics, including pros and cons. **Everything is a trade-off.**
>
> Each section contains links to more in-depth resources.

<!-- Table of Contents Section -->
* [Load balancer](#load-balancer)
  * [Layer 4 load balancing](#layer-4-load-balancing)
  * [Layer 7 load balancing](#layer-7-load-balancing)

---

## Load balancer

Load balancing distributes incoming network traffic across multiple servers to ensure high availability, fault tolerance, and responsiveness.

### Layer 4 load balancing

Layer 4 (L4) load balancing operates at the **Transport Layer** (TCP/UDP) of the OSI model. It routes traffic based on network-level metrics like IP address and port number without inspecting the actual payload or application content.

* **How it works:** Reads the IP packet header and TCP/UDP header to forward packet traffic directly to the backend servers.
* **Pros:** Fast and lightweight with lower CPU overhead because it does not decrypt or process HTTP payloads.
* **Cons:** Cannot make decisions based on URL paths, cookies, or HTTP headers.

### Layer 7 load balancing

Layer 7 (L7) load balancing operates at the **Application Layer** (HTTP/HTTPS) of the OSI model. It terminates the connection, inspects the HTTP request payload, and makes intelligent routing decisions based on request content.

* **How it works:** Reads headers, path parameters (e.g., `/api/v1/users`), request methods, and cookies to route requests to specific service instances.
* **Pros:** Enables microservice path routing, sticky sessions, SSL termination, and advanced traffic shaping.
* **Cons:** Higher CPU overhead due to TLS decryption and HTTP payload parsing.
