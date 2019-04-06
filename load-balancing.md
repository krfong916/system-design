# Load Balancing
## What it is
- Strategy to spread traffic across cluster of servers to improve availability and responsiveness for applications, websites or dbs.
- Distributes traffic across multiple backend servers, reducing individual server load. If server is not responding or has elevated error rate, LB will stop sending traffic to that server.
- This prevents any one server becoming a single point of failure, thus improving responsiveness and availability.
- Load Balancers help with horizontal scaling and redundancy. If a node fails, then traffic can be routed to another node.
- In summary:
  - prevents requests going to unhealthy servers
  - prevents overloaded resources
  - helps eliminate single points of failure

## Types and Algorithms
How a load balancer chooses a server:
- Load balancer uses a pre-config. algorithm to select a healthy server (see below)
- Health checks - lb sends ping to backend server to make sure server is listening. If not latent or no response, the lb may remove the server from pool. Traffic is not forwarded to that server until it responds to health check ping within normal bounds

### Algorithms
- Least connection: route request to server with fewest active connections. Useful for evenly distributing b/t servers
- Least response time: traffic to server w/fewest active connections and lowest avg. response timd
- Least bandwidth: selects server w/least amt. of traffic measured mgbs
- Round robin: equal distribution
- Weight round robin: equitable distribution. A 'weight' determines how many requests are sent that server's way, compared to others in the pool. Nodes are sent requests proportionate to their weight.
  - Ex: resource weight.
    - server-1 capacity = 4
    - server-2 capacity = 12
    - server-3 capacity = 1
  - for every 12 requests sent to server-2, 4 will be sent to server-1, and 1 to server-3. Results in a less equal, more even load distribution [4]
- Layer 4: these look at info at the transport layer to determine how to route request - destination IP address, source, ports in the header, but not the data itself.
    - requires less time and computing resources thatn layer 7, but performance impact is minimal in modern commodity hardware
- Layer 7: these load balancers look at the application layer to determine how to route. This can be session, header, message, and cookies. It terminates network traffic, reads message, makes a balancing-decision, and opens a connection to the selected server. e.g. direct video to servers that host videos while directing more sensitive user billing traffic to security-hardened servers

### Redundant Load Balancers
- Having only one load balancer can be a single point of failure. Adding additional lb to form a cluster can improve availability and responsiveness. Each lb monitors the health of servers and can serve traffic/detect failures. When one lb fails, others can take over.

Strategy to spread the load (e.g. HTTP requests) over a fleet of instances
Typical lb config. - some flavor of NGINX or AWS ELB in front of nodes (EC2 instances, docker containers...) [1]

A. Server-side load-balancing  (a.k.a. proxy load-balancing) [1]
- Load-balancer in front of fleet of servers. Clients connect to servers by going through the load-balancer
- Load-balancer acts as a proxy - isolates clients from backend nodes
- Pros:
    - simple client config. only need to know the load-balancer address
    - All traffic goes through the load-balancer
    - Clients not aware of backend servers
- Cons:
    - Single point of failure. If load-balancer fails then no connection to the servers to serve requests
    - Load-balancer may bottle-neck tons of traffic concentrated
    - Scaling is limited by load-balancer capabilities
    - Increased latency b/c of load-balancer proxy extra hops (?)
- Use cases:
    - Typical web-arch - it isolates clients from backend infrastructure
    - 2 types of proxy-load-balancers
        - network load-balancer
        - application load-balancer

B. Client-side load-balancing [2]
- This design assumes absolutely no load-balancers between servers and client. Direct connection between the client and server, therefore we assume the connection is trusted. The client can decide to use a round-robin or hashing mechanism, or even have the server report their actual load to clients in order to choose which server to fulfill their requests.
- Clients are more complex, they need to know how to implement the lb strategy themselves
- Pros:
    + Improved latency - direct connection from client to backend server (no proxy hops)
    + Improved scalability (just add servers)
- Cons:
    + Pay price of additional complexity on the client
    + Clients must be trusted
    + A/B testing is difficult to do (i.e. redirecting only portion of traffic to specific instances)
- Use cases:
    + ?

C. External load-balancing [3]
- Cluster state is maintained by external load balancer (Zookeeper) and clients ask load-balancer which server to connect to.
- Results of the load balancer must be cached allowing clients to connect to servers even when load-balancer is not available
- Pros:
    + Improved latency: direct connection from client to backend server (no proxy hops)
    + Improved scalability: depends only on num. servers (no bottleneck)
    + Reduced complexity on clients
- Use Cases:
    + microservice communications where you control both clients and servers
    + Also known as service discovery. If on aws, probably implement this using ELB
- How to handle failure?

## Logistics
Load balancers can be implemented with hardware (expensive) or software, like HAProxy.

## Where they can be placed
Load Balancer at each layer of the system
- b/t user and web server
- b/t web servers and internal platform layer, like application servers or cache servers
- b/t internal platform layer and dbs

## Review of benefits
- user experience faster, won't wait for struggling server to finish. Requests are passed to server with available resources
- Service providers exp. less downtime, higher throughput
- Full server failure doesn't end in poor user exp., load balancer routes request to healthy server
- Smart lbs use predicitve analytics to determine traffic bottlenecks and gives organization actionable insight at peak traffic times
- Several devices perform pieces of the work, no single pt. failure

Horizontal scaling - introduces complexity of cloning servers
Difference between Reverse proxy (web server) and load balancer


Sources:
1. [Load-Balancing Strategies: Beyond the Lines][7f662452]

  [7f662452]: http://www.beyondthelines.net/computing/load-balancing-strategies/ "Load-Balancing Strategies: Beyond the Lines"

2. [Load Balancing: System Design Primer][79025ff7]

  [79025ff7]: https://github.com/donnemartin/system-design-primer#load-balancer "Load Balancing: System Design Primer"

3. [Load Balancing: Educative.io][3af41ec0]

  [3af41ec0]: https://www.educative.io/collection/page/5668639101419520/5649050225344512/5747976207073280 "Load Balancing: Educative.io"

4. [Weight Round Robin][cceaa169]

  [cceaa169]: http://g33kinfo.com/info/archives/2657 "Weight Round Robin"
