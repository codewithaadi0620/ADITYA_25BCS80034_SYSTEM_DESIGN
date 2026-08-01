# URL Shortener System Design

## Functional Requirements

-   Convert long URLs into short URLs
-   Redirect short URLs to the original destination
-   Store and retrieve URL mappings
-   Support URL expiration (TTL)
-   Collect redirect analytics

------------------------------------------------------------------------

## Non-Functional Requirements

-   High scalability (millions of URLs)
-   Low latency (P99 \< 100 ms)
-   High availability
-   Rate limiting (Token Bucket per IP + Device)
-   Durable storage
-   Fault tolerance

------------------------------------------------------------------------

# High-Level Architecture

``` text
Users
   │
   ▼
DNS / API Gateway
   │
   ▼
Load Balancer
   ├──────────────┐
   ▼              ▼
Creation      Redirect
Service        Service
Cluster        Cluster
   │              │
   │              ▼
   │         Redis Cache
   │        (Hot URLs)
   │          │
   │          ├── Cache Hit → Redirect
   │          ▼
   │      Cache Miss
   │          │
   ▼          ▼
Cassandra Database
(URL Mapping + TTL)
   ▲
   │
Background Jobs
(Batch Sync + Analytics)
   ▲
   │
Kafka Event Stream
```

------------------------------------------------------------------------

## Components

### 1. API Gateway

-   Routes requests
-   Authentication (optional)
-   Request validation

### 2. Load Balancer

-   Distributes traffic
-   Improves availability

### 3. Creation Service

-   Generates unique IDs
-   Encodes IDs using Base62
-   Applies TTL
-   Stores mapping in Cassandra

### 4. Redirect Service

-   Looks up short code
-   Checks Redis first
-   Reads Cassandra on cache miss
-   Returns HTTP 301/302 redirect

### 5. Redis Cache

-   Stores hot URLs
-   Reduces database reads
-   Improves response time

### 6. Cassandra

-   Persistent storage
-   Read replicas
-   TTL support

### 7. Kafka

-   Receives redirect events
-   Decouples analytics
-   Supports asynchronous processing

### 8. Background Workers

-   Aggregate analytics
-   Batch update storage
-   Generate reports

------------------------------------------------------------------------

## Request Flow

### Create Short URL

1.  Client sends long URL.
2.  Request reaches Creation Service.
3.  Generate unique ID.
4.  Encode with Base62.
5.  Store mapping in Cassandra.
6.  Return short URL.

### Redirect

1.  User opens short URL.
2.  Redirect Service checks Redis.
3.  If cache hit → Redirect.
4.  If cache miss → Read Cassandra.
5.  Update Redis.
6.  Return HTTP 301/302.

------------------------------------------------------------------------

## Rate Limiting

-   Token Bucket Algorithm
-   Key = IP + Device ID
-   Stored in Redis

------------------------------------------------------------------------

## Analytics

-   Total redirects
-   Top URLs
-   Device statistics
-   Geographic distribution
-   Daily trends

------------------------------------------------------------------------

## Possible Improvements

-   Bloom Filter to reduce invalid lookups
-   CDN for global redirection
-   Multi-region deployment
-   Read/write separation
-   Monitoring using Prometheus & Grafana
