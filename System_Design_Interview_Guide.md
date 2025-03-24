# System Design Interview Guide

## Interview Process Overview

System design interviews typically last 45-60 minutes and evaluate your ability to design large-scale distributed systems. The interviewer will present an open-ended design problem, and you'll need to lead the discussion through various aspects of the system.

### What Interviewers Are Looking For

1. **Systematic approach** - Following a clear methodology
2. **Communication skills** - Articulating thoughts clearly and collaborating
3. **Technical knowledge** - Understanding of fundamental concepts and trade-offs
4. **Problem-solving ability** - Breaking down complex problems methodically
5. **Experience** - Drawing from past work when applicable

## The SNAKE Approach

Use the SNAKE methodology to structure your system design interview:

### S - Scope the Problem
- Clarify requirements and constraints
- Define functional and non-functional requirements
- Establish scale (users, data volume, etc.)
- Identify priorities and trade-offs

### N - Non-functional Requirements
- Reliability
- Availability
- Scalability
- Performance
- Security
- Cost
- Maintainability

### A - Architecture Design
- High-level components
- Data flow diagrams
- API endpoints
- Database schema

### K - Key Components Deep Dive
- Choose critical components to elaborate on
- Discuss algorithm choices
- Explain data structures
- Detail communications protocols

### E - Evaluate the Design
- Identify bottlenecks
- Address single points of failure
- Discuss scaling strategies
- Handle edge cases

## Common System Design Interview Questions

### 1. Design a URL Shortening Service (like bit.ly)

**Key considerations:**
- URL shortening algorithm
- Redirecting to original URLs
- Analytics and tracking
- Database schema and caching strategy
- Load balancing and scaling

**Quick approach:**
1. **Scope:** URL shortening, retrieval, analytics, user accounts
2. **Scale:** Millions of URLs, high read-to-write ratio
3. **Architecture:** Web servers, application servers, database, cache
4. **Algorithm:** Base62 encoding or MD5/SHA-1 hash with counter
5. **Database:** NoSQL for URL mappings, SQL for user data
6. **Scaling:** Caching, database sharding, CDN for static assets

![URL Shortener Design](https://i.imgur.com/L53OGDT.png)

### 2. Design Twitter

**Key considerations:**
- Tweet storage and retrieval
- Timeline generation
- Following/follower relationship
- Notifications
- Search functionality

**Quick approach:**
1. **Scope:** Post tweets, follow users, generate timeline, search
2. **Scale:** Millions of users, billions of tweets
3. **Architecture:** Microservices for different functions
4. **Data model:** Graph database for social connections, document store for tweets
5. **Timeline generation:** Fan-out on write vs. fan-out on read
6. **Scaling:** Caching, partitioning by user ID, read replicas

![Twitter System Design](https://i.imgur.com/NyBn02P.png)

### 3. Design a Distributed Key-Value Store

**Key considerations:**
- Read/write consistency
- Partitioning strategy
- Replication approach
- Fault tolerance
- Scalability

**Quick approach:**
1. **Scope:** Basic CRUD operations, configurable consistency
2. **Architecture:** Consistent hashing for node distribution
3. **Replication:** Primary-secondary replication with quorum consensus
4. **Consistency:** Support different consistency levels (eventual to strong)
5. **Fault tolerance:** Gossip protocol for failure detection
6. **Scaling:** Horizontal sharding, auto-scaling based on load

![Distributed KV Store](https://i.imgur.com/5p7uFZt.png)

### 4. Design YouTube

**Key considerations:**
- Video storage and streaming
- Transcoding pipeline
- Recommendation system
- Search functionality
- Analytics and monetization

**Quick approach:**
1. **Scope:** Upload, transcode, store, stream videos, user interactions
2. **Scale:** Petabytes of video data, millions of concurrent streams
3. **Architecture:** Upload service, transcoding service, metadata service, streaming service
4. **Storage:** Blob storage for videos, CDN for delivery
5. **Database:** Metadata in SQL, user data in NoSQL
6. **Scaling:** Geographic distribution, CDN caching, video chunking

![YouTube System Design](https://i.imgur.com/aZ7R9qP.png)

### 5. Design a Rate Limiter

**Key considerations:**
- Rate limiting algorithm
- Distributed coordination
- Storage for counters
- Response handling
- Configuration flexibility

**Quick approach:**
1. **Scope:** Limit requests by IP, user ID, or API key
2. **Algorithms:** Token bucket, leaky bucket, fixed window, sliding window
3. **Storage:** In-memory cache (Redis) for counters
4. **Distributed environment:** Synchronization across multiple instances
5. **Response:** HTTP 429 (Too Many Requests) with Retry-After header
6. **Scaling:** Shard counters by user/IP, centralized Redis cluster

![Rate Limiter Design](https://i.imgur.com/eIFPgCC.png)

## Common Mistakes to Avoid

1. **Diving into details too early** - Start with high-level design
2. **Not clarifying requirements** - Ask questions before designing
3. **Ignoring scalability** - Always consider how the system grows
4. **Overlooking data consistency** - Be explicit about consistency trade-offs
5. **Not considering failures** - Design for fault tolerance
6. **Being too vague** - Provide concrete numbers and specifics
7. **Overengineering** - Keep it as simple as possible while meeting requirements

## Sample Timeline for a 45-Minute Interview

- **0-5 minutes:** Clarify requirements and scope
- **5-10 minutes:** Define non-functional requirements, scale, and constraints
- **10-25 minutes:** High-level architecture design
- **25-40 minutes:** Deep dive into key components
- **40-45 minutes:** Evaluate design and discuss trade-offs

## Technical Concepts to Review

Before your interview, make sure you understand these concepts:

- **Load balancing** - Algorithms, health checks, session persistence
- **Caching** - Strategies, eviction policies, cache coherence
- **Database scaling** - Replication, sharding, denormalization
- **Consistency models** - Strong, eventual, causal consistency
- **Microservices** - Service discovery, API gateway, communication patterns
- **Distributed consensus** - Paxos, Raft, leader election
- **Messaging systems** - Pub/sub, queue-based, event-driven architecture
- **CDN** - Edge caching, geographical distribution, cache invalidation
- **Monitoring and analytics** - Logging, metrics, anomaly detection

## Conclusion

System design interviews assess your ability to architect complex systems that are scalable, reliable, and maintainable. By following a structured approach and understanding fundamental concepts, you can showcase your design thinking and technical knowledge effectively.

Remember that there's rarely one perfect solution – interviewers value your reasoning process and awareness of trade-offs more than arriving at a specific design. 