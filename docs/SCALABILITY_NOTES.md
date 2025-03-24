# Scalability in System Design - Comprehensive Notes

## 1. Introduction to Scalability

- **Definition**: System's ability to handle growing load while maintaining performance
- **Importance**: Critical for accommodating user growth, data growth, and traffic spikes
- **Key Properties**: Performance, availability, reliability under varying loads
- **Business Impact**: Directly affects user experience, operational costs, and business growth potential

## 2. Scaling Dimensions

### Vertical Scaling (Scale Up)
- Adding more power to existing machines
- CPU upgrades (more cores, higher clock speeds)
- RAM increases (16GB → 64GB → 128GB+)
- Storage enhancements (HDD → SSD, more capacity)
- Pros: Simple implementation, no distribution complexity
- Cons: Hardware limits, single point of failure, costly, requires downtime

### Horizontal Scaling (Scale Out)
- Adding more machines to the system
- Distributing load across multiple identical servers
- Pros: Virtually unlimited scaling, better fault tolerance, cost-effective
- Cons: Higher complexity, data consistency challenges, needs load balancing
- Common pattern: Start vertical, then transition to horizontal

## 3. Key Scalability Metrics

- **Throughput**: Requests/sec the system can handle
- **Response Time**: Total time to process a request
- **Latency**: Delay before processing begins
- **Resource Utilization**: CPU, memory, disk I/O, network usage percentages
- **Amdahl's Law**: Theoretical speedup from adding processors
- **Universal Scalability Law**: Models how contention and coherency affect scaling
- **Cost per Request**: Economic efficiency of scaling solutions

## 4. Load Balancing Essentials

- **Purpose**: Distribute traffic across multiple servers evenly
- **Types**:
  * Hardware (F5, Citrix) - High performance, expensive
  * Software (NGINX, HAProxy) - Flexible, cost-effective
  * DNS-based - Simple but limited control
  * Cloud services (AWS ELB, GCP LB) - Managed, integrated
- **Algorithms**:
  * Round Robin - Simple rotation between servers
  * Least Connections - Send to least busy server
  * IP Hash - Consistent server selection based on client IP
  * Weighted methods - Account for server capacity differences
- **Features**:
  * Health checks - Auto-remove failed servers
  * SSL termination - Offload encryption handling
  * Session persistence - Keep user on same server
  * Content-based routing - Route by request type

## 5. Database Scaling Strategies

### Replication
- **Master-Slave**: Write to master, read from slaves
- **Master-Master**: Write to any master, replication between them
- **Read Replicas**: Dedicated servers for read operations
- **Benefits**: Read scalability, fault tolerance, geographic distribution

### Partitioning
- **Vertical Partitioning**: Split by feature (users DB, products DB)
- **Horizontal Partitioning (Sharding)**: Split by data segment (users A-M, N-Z)
- **Functional Partitioning**: Split by operation type (reporting DB, transactional DB)

### Database Types for Scale
- **Relational (SQL)**: Strong consistency, ACID properties, complex queries
- **Document Stores**: Schema flexibility, scalable reads/writes (MongoDB)
- **Key-Value Stores**: Simple, extremely fast, high throughput (Redis, DynamoDB)
- **Column-Family**: Wide-column storage, high write throughput (Cassandra)
- **Graph Databases**: Relationship-focused scaling (Neo4j)
- **NewSQL**: Attempting SQL features with NoSQL scalability (CockroachDB)

## 6. Caching Deep Dive

- **Definition**: Store frequently accessed data in fast-access storage
- **Cache Levels**:
  * Client-side: Browser caching, local storage
  * CDN: Edge location content caching
  * Application: In-memory data caching
  * Database: Query results, buffer pools
  * Distributed: Shared cache across services

- **Caching Strategies**:
  * Cache-aside (lazy loading): App checks cache first, updates cache after DB read
  * Write-through: Write to cache and DB simultaneously
  * Write-behind: Write to cache, asynchronously update DB
  * Refresh-ahead: Predict and refresh cache before expiration

- **Cache Invalidation**:
  * TTL (Time-to-Live): Automatic expiration
  * Write invalidation: Invalidate on data changes
  * Event-based: External triggers for invalidation

- **Cache Technologies**:
  * Redis: In-memory data structure store, persistence
  * Memcached: Distributed memory caching
  * Varnish: HTTP accelerator
  * Ehcache: Java-based caching

## 7. Content Delivery Networks

- **Function**: Distribute content across global edge locations
- **Benefits**:
  * Dramatic latency reduction for users
  * Origin server load reduction
  * Traffic spike absorption
  * DDoS attack mitigation
  * Global presence with local performance

- **Content Types**:
  * Static assets: Images, CSS, JavaScript files
  * Video content: Streaming media
  * Dynamic content: API responses, personalized content
  * Web applications: Full application acceleration

- **CDN Operation**:
  * Request routing: Direct users to optimal edge location
  * Content caching: Store copies at edge
  * Origin shielding: Protect backend from excess traffic
  * Dynamic acceleration: Optimize connection paths

## 8. Asynchronous Processing

- **Concept**: Decouple time-consuming operations from request handling
- **Message Queue Architecture**:
  * Producers: Create messages/tasks
  * Queues: Store messages reliably
  * Consumers: Process messages

- **Technologies**:
  * RabbitMQ: Advanced routing capabilities
  * Kafka: High-throughput distributed streaming
  * SQS/SNS: AWS managed messaging services
  * Google Pub/Sub: Serverless messaging

- **Patterns**:
  * Work queues: Distribute tasks among workers
  * Publisher/subscriber: Send to multiple consumers
  * Request/reply: Asynchronous RPC
  * Competing consumers: Scale consumers horizontally

- **Benefits**:
  * Improved request handling capacity
  * Graceful handling of traffic spikes
  * Better fault isolation
  * Decoupled system components

## 9. Microservices and Scalability

- **Architecture**: Break applications into small, independent services
- **Scaling Advantages**:
  * Individual service scaling based on demand
  * Technology diversity for optimal solutions
  * Independent deployment cycles
  * Isolated failure domains
  * Team scalability

- **Implementation Considerations**:
  * Service discovery
  * API gateways
  * Circuit breakers
  * Distributed tracing
  * Containerization (Docker)
  * Orchestration (Kubernetes)

- **Challenges**:
  * Increased operational complexity
  * Distributed transactions
  * Network latency
  * Consistent monitoring
  * Data consistency between services

## 10. Auto-Scaling Mechanisms

- **Definition**: Automatically adjust resources based on demand
- **Approaches**:
  * Reactive: Scale based on current metrics
  * Predictive: Scale based on forecasted demand
  * Scheduled: Scale based on known patterns (e.g., business hours)

- **Metrics Used**:
  * Infrastructure: CPU, memory, disk I/O
  * Application: Request rate, queue length, error rate
  * Business: User logins, transactions, active sessions

- **Implementation**:
  * Cloud providers: AWS Auto Scaling, Azure Scale Sets
  * Container orchestration: Kubernetes HPA
  * Custom solutions: Self-developed scaling logic

- **Best Practices**:
  * Set appropriate thresholds
  * Implement scale-in protection
  * Create scaling policies for different scenarios
  * Combine approaches for optimal results

## 11. Data Sharding Techniques

- **Concept**: Split data across multiple database instances
- **Sharding Keys**: The basis for data distribution
- **Strategies**:
  * Range-based: Split by data ranges (A-M, N-Z)
  * Hash-based: Distribute by hash of key
  * Directory-based: Maintain lookup table for shard locations
  * Geographic: Distribute by user location

- **Challenges**:
  * Joining data across shards
  * Resharding as data grows
  * Hotspots (uneven distribution)
  * Schema changes across all shards
  * Transaction management

- **Real-world Examples**:
  * Instagram: Photo storage sharding
  * Twitter: Timeline service sharding
  * Facebook: User data sharding
  * Google: BigTable sharding

## 12. Advanced System Design Patterns

- **Statelessness**: Design services without server-side state
  * Benefits: Easy scaling, resilience, redeployment
  * Implementation: Store state in databases, caches, client-side

- **CAP Theorem**: Choose two - Consistency, Availability, Partition tolerance
  * CA: Traditional RDBMS (sacrifices partition tolerance)
  * CP: MongoDB, HBase (sacrifices some availability)
  * AP: Cassandra, DynamoDB (sacrifices strong consistency)

- **Eventual Consistency**: System will become consistent over time
  * Use cases: Social media feeds, comment systems
  * Benefits: Higher availability, better performance

- **CQRS**: Separate read and write operations
  * Command (write) models optimized for updates
  * Query (read) models optimized for reads
  * Benefits: Independent scaling, performance optimization

- **Event Sourcing**: Store changes as events
  * Enables complete audit history
  * Enables replay of system state
  * Works well with CQRS

- **Idempotent Operations**: Multiple identical requests have same effect as one
  * Critical for distributed systems reliability
  * Implementation: Request IDs, conditional operations

- **Circuit Breakers**: Prevent cascading failures
  * States: Closed (normal), Open (failing), Half-open (testing)
  * Implementation: Hystrix, Resilience4j

## 13. Real-world Case Studies

### Netflix
- Microservices architecture (hundreds of services)
- Multi-region deployment for resilience
- Chaos Engineering (Chaos Monkey)
- Sophisticated caching for video content
- Predictive scaling for viewing patterns

### Facebook
- Custom sharding for user data
- Specialized data stores (TAO for social graph)
- Geographically distributed infrastructure
- Multi-layer caching
- Optimized for read-heavy workloads

### Twitter
- Timeline service scaling challenges
- Real-time processing of tweets
- Fanout service for distributing tweets
- Redis-based temporal feeds
- Manhattan database system

### Amazon
- Service-oriented architecture
- Two-pizza teams (organizational scaling)
- Dynamo DB for high availability
- Recommendation engine scaling
- Prioritization of critical path operations

## 14. Scaling Best Practices

- **Start Simple**: Begin with simple architecture, scale as needed
- **Measure Everything**: Implement comprehensive monitoring
- **Identify Bottlenecks**: Use data to find scaling limitations
- **Test at Scale**: Load testing under realistic conditions
- **Design for Failure**: Assume components will fail
- **Automate Everything**: Manual processes don't scale
- **Cost Optimization**: Balance performance needs with costs
- **Consider Data Locality**: Keep data close to processing
- **Minimize Cross-region Traffic**: Reduce latency and costs
- **Right-size Resources**: Avoid over or under-provisioning

## 15. Harvard CS75 Lecture 9 Key Points

- **Vertical vs. Horizontal Scaling**: Trade-offs and decision factors
- **Database Scaling Challenges**: Why databases are often the bottleneck
- **Shared-Nothing Architecture**: Independence of nodes in scaling
- **Session Management**: Strategies for maintaining user sessions
- **Caching Hierarchies**: Multi-level caching approaches
- **Real-world Scaling Stories**: Facebook, Twitter evolution

## 16. Tools and Technologies

- **Monitoring**: Prometheus, Grafana, Datadog, New Relic
- **Load Testing**: JMeter, Locust, Gatling, K6
- **Containers**: Docker, Kubernetes, ECS
- **Databases**: MongoDB, Cassandra, Redis, MySQL Cluster
- **Caching**: Redis, Memcached, Varnish
- **Load Balancers**: NGINX, HAProxy, AWS ELB
- **CDNs**: Cloudflare, Fastly, Akamai, CloudFront
- **CI/CD**: Jenkins, GitHub Actions, CircleCI (for deployment scaling)

## 17. Future Trends in Scalability

- **Serverless Architecture**: Scale to zero, automatic scaling
- **Edge Computing**: Processing closer to users
- **Multi-cloud Strategies**: Distributing across providers
- **AI-driven Scaling**: Intelligent resource allocation
- **eBPF and Observability**: Deeper system insights
- **Global Database Systems**: Globally distributed databases
- **Sustainable Scaling**: Energy-efficient infrastructure 