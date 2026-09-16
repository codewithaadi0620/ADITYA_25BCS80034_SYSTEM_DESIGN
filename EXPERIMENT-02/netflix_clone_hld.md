# Netflix Clone --- System Design HLD

## Functional Requirements

1.  Users should be able to register, log in, and authenticate securely.
2.  Users should be able to browse movies, TV shows, and other video
    content.
3.  Users should be able to search content using titles, genres, actors,
    or keywords.
4.  Users should be able to stream videos in multiple resolutions based
    on network conditions.
5.  Users should receive personalized content recommendations based on
    their watch history and preferences.
6.  Users should be able to continue watching from the last playback
    position across multiple devices.
7.  Users should be able to download selected content for offline
    viewing.

## Non-Functional Requirements

1.  **Availability:** The platform should remain available with minimal
    downtime.
2.  **Scalability:** It should support millions of concurrent users and
    continuous traffic growth.
3.  **Reliability:** Video playback should be smooth with minimal
    interruptions or failures.
4.  **Low Latency:** Videos should start quickly with minimal buffering.
5.  **Fault Tolerance:** The system should continue operating even if
    individual servers or services fail.
6.  **Performance:** The platform should deliver high-quality streaming
    across different devices and network conditions.
7.  **Security:** User accounts, subscriptions, and streaming data
    should be securely protected.
8.  **Global Content Delivery:** Videos should be served efficiently
    worldwide using Content Delivery Networks (CDNs).

------------------------------------------------------------------------

# High-Level Architecture

``` text
CLIENT
   |
   v
API Gateway
   |
   v
Load Balancer
   |
   +---------------------------------------------------+
   |                                                   |
   v                                                   v
User Service     Content Service     Streaming Service
Recommendation Service              Search Service
   |                                                   |
   +----------------------+----------------------------+
                          |
                          v
                    Message Queue
                       (Kafka)
                          |
                          v
                     Redis Cache
                          |
                          v
                      DATABASE
              (Metadata, Users, Profiles, etc.)

Streaming Service
        |
        v
Object Storage
(Videos, Images, Thumbnails, etc.)

API Gateway / Services
        |
        v
Push Notification Service
(APNs / FCM)
```

## Main Components

### Client

-   Web, mobile, and TV clients.
-   Sends requests to the backend through the API Gateway.

### API Gateway

-   Single entry point for client requests.
-   Handles routing and authentication.
-   Can also perform rate limiting and request validation.

### Load Balancer

-   Distributes incoming requests across backend instances.
-   Helps with horizontal scaling and availability.

### User Service

-   Registration and login.
-   User profiles and preferences.
-   Authentication-related operations.

### Content Service

-   Movies and TV shows.
-   Metadata, genres, actors, thumbnails, etc.

### Streaming Service

-   Creates playback sessions.
-   Handles video streaming.
-   Supports different resolutions depending on network conditions.

### Recommendation Service

-   Generates personalized recommendations.
-   Uses watch history and user preferences.

### Search Service

-   Searches movies and shows by title, genre, actor, or keyword.

### Message Queue --- Kafka

-   Handles asynchronous events.
-   Useful for watch events, notifications, analytics, and background
    processing.
-   Reduces direct coupling between services.

### Redis Cache

-   Caches frequently accessed data.
-   Examples:
    -   Movie metadata
    -   Recommendations
    -   Search results
    -   User sessions

### Database

Stores: - User information - Profiles - Movie/show metadata - Watch
history - Playback position - Other application data

### Object Storage

Stores large media files such as: - Videos - Images - Thumbnails

The streaming service can fetch video content from object storage, while
a CDN can be placed in front of it for global delivery.

### Push Notification Service

-   Sends notifications using APNs / FCM.
-   Can notify users about new content, reminders, etc.

------------------------------------------------------------------------

# Request Flow

## Normal API Request

``` text
Client
  ↓
API Gateway
  ↓
Load Balancer
  ↓
Required Microservice
  ↓
Redis Cache
  ↓
Database (on cache miss)
```

## Video Playback Flow

``` text
Client
  ↓
API Gateway
  ↓
Streaming Service
  ↓
Object Storage / CDN
  ↓
Video Stream
```

## Asynchronous Processing

``` text
Service
  ↓
Kafka
  ↓
Background Consumer
  ↓
Notification / Analytics / Other Processing
```

------------------------------------------------------------------------

# Scalability Considerations

-   Keep backend services stateless where possible.
-   Horizontally scale services by adding more instances.
-   Use load balancing for distributing requests.
-   Use Redis for frequently accessed data.
-   Use Kafka for high-throughput asynchronous events.
-   Store videos in object storage rather than the application servers.
-   Use a CDN to reduce latency and origin-server load.
-   Use database replication/read replicas for read-heavy workloads.
-   Use monitoring and automatic scaling for traffic spikes.

# Reliability and Availability

-   Deploy services across multiple availability zones.
-   Use database replication.
-   Use health checks and automatic failover.
-   Keep redundant application instances.
-   Use CDN redundancy where appropriate.
-   Monitor latency, errors, CPU, memory, and traffic.
