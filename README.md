# common-system-design-problems

## 🔹 Common Problems in E-Commerce Applications

### 1. Consistency Problems
E-commerce systems often need strong consistency (e.g., inventory, payments) but may trade off in some areas (e.g., search, recommendations).

**Examples:**

*   **Inventory Mismatch**
    *   Two users try to purchase the last item at the same time.
    *   DB replicas may be out-of-sync, leading to overselling.
*   **Cart vs Inventory Sync**
    *   Cart shows “in stock” but item is sold out by checkout.
*   **Payment & Order State**
    *   Payment succeeds but order record isn’t updated due to DB inconsistency.

**Solutions:**

*   Use distributed transactions only where absolutely necessary.
*   Event-driven architecture (e.g., Kafka) with idempotent consumers.
*   Pessimistic/Optimistic locking in inventory systems.

### 2. Availability Problems
High traffic (sales, festive events, flash sales) causes downtime or degraded performance.

**Examples:**

*   **Checkout Failure**
    *   Payment gateway timeout → user left hanging.
*   **Catalog Service Down**
    *   User cannot browse products.
*   Single Point of Failure in monolith services (e.g., order service crashing brings everything down).

**Solutions:**

*   Microservices separation (cart, order, payment, catalog).
*   Circuit breakers (if payment fails, still allow retries).
*   Graceful degradation (show limited features instead of total failure).
*   Multi-region deployment for global e-commerce.

### 3. Scalability Problems
E-commerce apps face highly variable loads (normal days vs Black Friday).

**Examples:**

*   **Database Bottlenecks**
    *   Millions of reads/writes on orders/inventory tables.
*   **Hot Products**
    *   One viral product → hotspot in DB & cache.
*   **Search Scalability**
    *   Real-time search & filtering with millions of SKUs.
*   **Recommendation Engine**
    *   Personalized recommendations for millions of users.

**Solutions:**

*   Caching (Redis, CDN) → reduce DB load.
*   Sharding/Partitioning → distribute data (e.g., users/orders by region).
*   CQRS + Event Sourcing → separate read/write paths.
*   ElasticSearch for catalog & search queries.
*   Auto-scaling (K8s, AWS ASG) for peak traffic.

### ⚖️ Trade-offs in E-Commerce
*   Inventory (Consistency > Availability) → prevent overselling.
*   Search & Recommendations (Availability > Consistency) → eventual consistency is fine.
*   Payments (Consistency > Availability) → must avoid double charges.
*   Product Reviews, Analytics (Availability > Consistency) → okay if slightly delayed.

### 🏗️ Example Scalable Design
*   Frontend → React/Next.js (CDN cached).
*   APIs → Microservices (Node.js, FastAPI, etc.) with gRPC/REST.
*   Data Layer →
    *   OLTP DB (Postgres/MySQL) for orders.
    *   Redis cache for cart & inventory stock.
    *   ElasticSearch for catalog & search.
    *   Data Lake/Warehouse for analytics.
*   Infra → Kubernetes + Auto-scaling + Load Balancers.
*   Messaging → Kafka/RabbitMQ for order events, inventory updates.
*   Resilience → Circuit breakers, retries, async workers.

### ✅ So in summary:
*   Consistency issues → overselling, payment mismatch.
*   Availability issues → downtime during checkout, single points of failure.
*   Scalability issues → handling spikes, database bottlenecks, search scale.

---

## 🔹 Common Problems in Chat Applications

### 1. Consistency Problems
Messages must be delivered in correct order and not lost, but distributed systems make this tricky.

**Examples:**

*   **Message Ordering**
    *   User A sends two messages, B sees them in wrong order.
*   **Duplicate Messages**
    *   Retries (due to network failures) cause multiple deliveries.
*   **Read Receipts Sync**
    *   Message marked as “read” on one device but not reflected on others.
*   **Offline Mode**
    *   User sends messages offline → inconsistencies when reconnecting.

**Solutions:**

*   Use message IDs + sequence numbers to enforce ordering.
*   Idempotent writes → deduplicate messages.
*   Event sourcing → append-only logs for chat history.
*   Conflict resolution (last-write-wins, vector clocks).

### 2. Availability Problems
Chats must be available 24/7, but real-time systems are vulnerable.

**Examples:**

*   **Server Outage**
    *   Entire region’s users can’t send/receive messages.
*   **Group Chats**
    *   1 message to 1,000 members → fan-out bottleneck.
*   **Typing Indicators / Presence**
    *   Delays or missing updates → poor UX.
*   **Push Notifications**
    *   Failures mean users miss new messages.

**Solutions:**

*   Multi-region replication with active-active clusters.
*   Publish/Subscribe model (Kafka, Redis PubSub, MQTT).
*   Presence service with short TTLs (e.g., Redis with expiry).
*   Fallbacks (if WebSocket fails → switch to long-polling).

### 3. Scalability Problems
Billions of messages per day → huge scale issues.

**Examples:**

*   **Storage Growth**
    *   Storing years of chat history for millions of users.
*   **Hot Partitions**
    *   Popular chat groups overwhelm one shard.
*   **Search in Messages**
    *   Millions of chats → hard to search quickly.
*   **Real-time Fan-out**
    *   Group chat delivery must scale horizontally.

**Solutions:**

*   Sharding by userId / chatId across databases.
*   Cold storage for old chats (S3, Hadoop).
*   ElasticSearch for chat search.
*   Async fan-out (store once, deliver via pub-sub).
*   CQRS (separate read/write paths).

### 4. Reliability Problems
Users expect no message loss, even during failures.

**Examples:**

*   **Network Failures**
    *   Messages dropped during poor connectivity.
*   **Mobile Device Issues**
    *   App killed in background → messages missed.
*   **Exactly-once Delivery**
    *   Hard in distributed systems; often at-least-once with deduplication.

**Solutions:**

*   Message queues (Kafka, RabbitMQ) for persistence.
*   At-least-once delivery + deduplication.
*   Acknowledgments (ACKs) for sender confirmation.
*   Store-and-forward model (server keeps until ACK received).

### 5. Security & Privacy Problems
Chats handle sensitive data.

**Examples:**

*   Eavesdropping (MITM attacks).
*   Data Breach (chat history leaked).
*   Unauthorized Access (session hijacking).

**Solutions:**

*   End-to-End Encryption (E2EE) → only sender & receiver can read.
*   Token-based authentication (JWT, OAuth).
*   Transport Layer Security (TLS).
*   Zero-knowledge storage (server stores only encrypted blobs).

### ⚖️ Trade-offs in Chat Apps
*   Consistency vs Availability
    *   WhatsApp may show delayed delivery if server is down (consistency prioritized).
    *   Slack may temporarily allow message sending even if presence updates lag (availability prioritized).
*   Real-time vs Cost
    *   WebSockets scale well but are costly; fallback to polling for inactive users.

### 🏗️ Scalable Chat App Design (High Level)
*   Frontend → React Native / Flutter with WebSocket + Push Notifications.
*   Gateway Layer → Load Balancer + WebSocket servers.
*   Messaging Layer → Kafka / Redis PubSub for fan-out.
*   Storage →
    *   Cassandra/DynamoDB for chat history.
    *   ElasticSearch for search.
    *   Redis for online status & recent chats.
*   Microservices → Authentication, Messaging, Notification, Search, Presence.
*   Infra → Multi-region deployment, Kubernetes, Auto-scaling.

### ✅ In summary, chat apps face challenges in:
*   Consistency → ordering, duplication, read receipts.
*   Availability → downtime, group scaling, notifications.
*   Scalability → billions of messages, fan-out, storage.
*   Reliability → network failures, retries, ACKs.
*   Security → encryption, authentication, privacy.

---

## 🔹 Common Problems in Video Streaming Applications

### 1. Latency & Playback Problems
Delivering smooth, low-latency video globally is challenging.

**Examples:**

*   **Buffering**
    *   Slow network or inadequate bandwidth causes frequent pauses.
*   **Startup Delay**
    *   Users wait too long for video to start.
*   **Quality Switching (ABR)**
    *   Video quality drops suddenly or doesn't adapt well.
*   **Synchronization**
    *   Audio/video desync, or live stream desync across viewers.

**Solutions:**

*   CDNs (Content Delivery Networks) → distribute content closer to users.
*   Adaptive Bitrate Streaming (HLS, DASH) → dynamically adjust quality.
*   Edge caching → cache popular segments at network edge.
*   Optimized encoding (HEVC, AV1) → reduce bandwidth.
*   Low-latency streaming protocols (LL-HLS, WebRTC).

### 2. Scalability & Cost Problems
Streaming video to millions is resource-intensive and expensive.

**Examples:**

*   **Bandwidth Cost**
    *   Massive data transfer costs, especially for global reach.
*   **Transcoding/Encoding**
    *   Converting videos to multiple formats/qualities is CPU-heavy.
*   **Live Event Spikes**
    *   Sudden millions of viewers overwhelm servers.
*   **Storage**
    *   Storing huge video libraries (VOD) and live archives.

**Solutions:**

*   Cloud-based transcoding services (AWS Elemental MediaConvert).
*   Layered caching (CDN + origin caching).
*   Geo-distributed storage (S3, object storage).
*   Auto-scaling media servers and origin infrastructure.
*   Peer-to-peer (P2P) delivery for some use cases.

### 3. Reliability & Resiliency Problems
Video delivery must be robust against failures.

**Examples:**

*   **Origin Server Failure**
    *   Source of video content goes down.
*   **CDN Outage**
    *   Content unavailable for a region.
*   **Encoder Failure (Live)**
    *   Live stream interrupted or corrupted.
*   **DRM/Copy Protection**
    *   Breaches lead to content piracy.

**Solutions:**

*   Redundant origin servers + failover mechanisms.
*   Multi-CDN strategy for high availability.
*   Transcoding pipelines with built-in retry logic.
*   Robust DRM systems + watermarking.
*   Monitoring & alerting (bandwidth, errors, uptime).

### 4. Quality of Experience (QoE) Problems
Ensuring a good user experience beyond just technical delivery.

**Examples:**

*   **User Engagement**
    *   Poor quality leads to users abandoning streams.
*   **Personalization**
    *   Lack of relevant content or recommendations.
*   **Analytics**
    *   Difficulty tracking viewership, engagement, and errors.
*   **Monetization**
    *   Ad-blocking or ad delivery failures.

**Solutions:**

*   A/B testing for different encoding settings.
*   Recommendation engines (ML-based).
*   Comprehensive analytics platforms (video-specific metrics).
*   Dynamic Ad Insertion (DAI) for seamless ad delivery.
*   Optimized UI/UX for playback controls.

### ⚖️ Trade-offs in Video Streaming
*   **Quality vs Cost**
    *   Higher resolution/bitrate = better quality but higher storage/bandwidth costs.
*   **Latency vs Consistency**
    *   Lower latency often means less buffering but can introduce synchronization challenges.
*   **Security vs User Experience**
    *   Strong DRM can sometimes add friction to playback.

### 🏗️ Scalable Video Streaming Design (High Level)
*   **Ingest:** Live encoders, upload services.
*   **Processing:** Transcoding/Transmuxing farm (e.g., AWS Elemental MediaConvert, FFMPEG).
*   **Storage:** Object storage (S3) for VOD assets.
*   **Delivery:** CDN (Akamai, CloudFront) for global distribution.
*   **Streaming Protocols:** HLS/DASH for ABR, WebRTC for low-latency live.
*   **Player:** HTML5 video player with ABR logic.
*   **DRM:** Widevine, PlayReady, FairPlay for content protection.
*   **Monitoring:** QoE metrics, error rates, performance.

### ✅ In summary, video streaming apps face challenges in:
*   **Latency & Playback** → buffering, startup, quality adaptation.
*   **Scalability & Cost** → bandwidth, transcoding, peak traffic.
*   **Reliability & Resiliency** → server failures, DRM.
*   **Quality of Experience** → engagement, personalization, analytics.

---

## 🔹 Common Problems in Delivery Tracking Applications

### 1. Consistency Problems
Tracking must always reflect accurate, real-time delivery partner location.
But due to distributed systems, mobile networks, and caching, consistency is hard.

**Examples:**

*   **Stale Location Updates** → Partner already moved, but customer sees an old position.
*   **Out-of-Order Updates** → Network delays cause locations to arrive in the wrong order.
*   **Multi-device Sync** → Customer checks app on mobile and web → sees different ETAs.
*   **Edge Cases** → Rider goes offline or switches network → gaps in tracking.

**Solutions:**

*   Use event time + sequence IDs to order updates.
*   Last-write-wins or vector clocks for conflicting updates.
*   WebSockets / MQTT for real-time streams, with retries.
*   Interpolation (map-matching) to smooth location jumps.

### 2. Availability Problems
Tracking must be available 24/7 even with outages.

**Examples:**

*   **Service Downtime** → Location API unavailable → customer sees “tracking unavailable”.
*   **Network Drop** → Rider goes offline → app shows them stuck.
*   **Overloaded System** → On festivals/sales (Amazon/Flipkart Big Billion Days), real-time tracking lags.

**Solutions:**

*   Multi-region deployments with failover.
*   Graceful degradation (e.g., fallback to SMS/periodic updates if real-time fails).
*   Retry with exponential backoff for location updates.
*   Edge caching of last known location.

### 3. Scalability Problems
Tracking millions of concurrent deliveries in real time is huge scale.

**Examples:**

*   **High Volume Location Streams** → Each rider sends GPS updates every 2–5 seconds.
*   **Fan-out** → 1 location update → multiple consumers (customer, restaurant, support, analytics).
*   **Hotspots** → Popular areas (airports, metro cities) generate extreme traffic.
*   **Storage Growth** → Billions of GPS points per day (location history).

**Solutions:**

*   Message Queues (Kafka, Pulsar, Kinesis) for ingestion.
*   Stream Processing (Flink, Spark Streaming) for ETA calculations.
*   Geo-sharding → Partition data by region.
*   TTL-based storage → Keep detailed GPS points for 7 days, aggregate older ones.

### 4. Reliability Problems
Tracking must be accurate and never lost.

**Examples:**

*   **Dropped Updates** → Rider sends update but server drops it.
*   **Duplicate Updates** → Same location processed twice.
*   **ETA Inaccuracy** → Poor traffic models → wrong delivery estimates.

**Solutions:**

*   Idempotent APIs with unique update IDs.
*   At-least-once delivery via queues, with deduplication.
*   Predictive ETAs → Machine learning models + live traffic APIs.
*   Store-and-forward → Rider app buffers updates if offline.

### 5. Security & Privacy Problems
Location = sensitive personal data.

**Examples:**

*   **Data Leaks** → Unauthorized access to rider location history.
*   **Spoofing** → Fake GPS apps send wrong coordinates.
*   **Man-in-the-Middle Attacks** → Location hijacked in transit.

**Solutions:**

*   JWT/OAuth2 auth for APIs.
*   End-to-End TLS encryption for location updates.
*   Device attestation (check for rooted devices, GPS spoofing).
*   Role-based access control (only customer + restaurant + delivery ops see location).

### 6. User Experience Problems
Even if backend works, UX can fail.

**Examples:**

*   **Jumpy Maps** → Rider’s icon jumps back and forth.
*   **Wrong ETA** → Customer frustration.
*   **No Update During Traffic Jams** → Looks like rider is idle.
*   **Delayed Notifications** → Customer gets “Your order is arriving” after it’s already delivered.

**Solutions:**

*   Map-matching & interpolation → smooth rider movement.
*   ML-based ETA → trained on past trips.
*   Push notifications + WebSockets → real-time status.
*   Fallback UX → “Rider is nearby” instead of exact meters when accuracy is low.

### 🏗️ Scalable Delivery Tracking System Design (High-Level)
*   **Mobile App (Rider)**
    *   Sends GPS updates every 2–5 seconds.
    *   Buffered when offline.
*   **API Gateway**
    *   Auth + rate limiting.
    *   Routes updates to ingestion layer.
*   **Ingestion Layer**
    *   Kafka / Pulsar / Kinesis for high-throughput GPS streams.
*   **Processing Layer**
    *   Stream processors (Flink/Spark) calculate:
        *   ETA
        *   Route deviations
        *   Aggregated positions
*   **Storage**
    *   Redis → current live location.
    *   Cassandra/DynamoDB → historical GPS.
    *   ElasticSearch → location-based queries (nearest rider).
*   **Delivery to Clients**
    *   WebSockets / MQTT → customers + restaurants.
    *   Push notifications → fallback.
*   **Analytics**
    *   Heatmaps, traffic prediction, rider efficiency.

### ✅ In summary, delivery tracking apps face problems in:
*   Consistency → ordering, stale data, offline mode.
*   Availability → downtime, overload, network drop.
*   Scalability → millions of updates/sec, geo-sharding.
*   Reliability → no drops, idempotency, correct ETAs.
*   Security → privacy, spoofing, leaks.
*   UX → smooth tracking, correct ETA, real-time notifications.

---

## 🔹 Common Problems in Social Media Applications

### 1. Consistency Problems
Social media feeds need to reflect updates quickly but maintaining global consistency is hard.

**Examples:**

*   **Feed Lag** → User posts, but followers see it with delay.
*   **Duplicate Posts** → Network retries cause multiple identical posts.
*   **Like/Comment Counts** → Numbers don’t sync across all replicas instantly.
*   **Deleted Content** → Content deleted, but still appears on some feeds.

**Solutions:**

*   Eventual consistency (e.g., for feeds, notifications).
*   Conflict-free Replicated Data Types (CRDTs) for some cases (e.g., counters).
*   Idempotent write APIs to prevent duplicates.
*   Read-repair mechanisms for stale data.

### 2. Availability Problems
Social media apps must be always-on, especially during peak events.

**Examples:**

*   **Service Outage** → Users can't post, view feeds, or chat.
*   **Image Upload Failures** → Users can't share photos/videos.
*   **Notification Downtime** → Users miss important alerts.
*   **Single Point of Failure** → A single database or service failure brings down the entire platform.

**Solutions:**

*   Multi-region deployment with active-active setup.
*   Microservices architecture for fault isolation.
*   Circuit breakers and fallbacks.
*   CDNs for static content (images, videos).

### 3. Scalability Problems
Handling billions of users, trillions of posts, and real-time interactions.

**Examples:**

*   **Fan-out Problem (News Feed)** → One post to millions of followers.
*   **Hot Users/Content** → Viral posts/celebrity accounts generate massive reads.
*   **Storage Growth** → Storing petabytes of user-generated content.
*   **Real-time Search** → Indexing and searching billions of posts/users.

**Solutions:**

*   Fan-out on write (push model) for celebrity feeds, fan-out on read (pull model) for general users.
*   Sharding data by user ID, content ID, or region.
*   Caching (Redis, Memcached) for popular content.
*   CDN for media files.
*   ElasticSearch/Solr for search.

### 4. Reliability Problems
Content must be persistent and accessible.

**Examples:**

*   **Data Loss** → Posts, comments, likes disappear.
*   **Permanent Deletion** → User deletes content, but it reappears from backup.
*   **Platform Crashing** → App crashes during high traffic, causing data loss.

**Solutions:**

*   Redundant storage (e.g., triple replication in S3).
*   Frequent backups and point-in-time recovery.
*   Message queues (Kafka) for durable event delivery.
*   Idempotent operations to prevent unintended state changes.

### 5. Security & Privacy Problems
User data is highly sensitive and prone to attacks.

**Examples:**

*   **Account Takeover** → Session hijacking, weak passwords.
*   **Data Breach** → User profiles, private messages leaked.
*   **Content Moderation** → Hate speech, spam, illegal content.
*   **Privacy Violations** → Unauthorized data sharing, tracking.

**Solutions:**

*   Robust authentication (2FA, OAuth2, JWT).
*   Data encryption at rest and in transit.
*   AI/ML for content moderation and anomaly detection.
*   Role-based access control (RBAC).
*   GDPR/CCPA compliance for data privacy.

### 6. Personalization & Recommendation Problems
Tailoring content to individual users at scale.

**Examples:**

*   **Relevance** → Showing irrelevant content to users.
*   **Cold Start** → New users have no data for recommendations.
*   **Bias** → Recommendation algorithms perpetuate existing biases.
*   **Real-time Updates** → Recommendations don’t adapt to recent user activity.

**Solutions:**

*   Collaborative filtering, content-based filtering, hybrid models.
*   Matrix factorization (e.g., ALS).
*   Deep learning models (e.g., embedding-based recommendations).
*   Hybrid approach for cold start users (popular items, demographic info).

### 🏗️ Scalable Social Media Design (High-Level)
*   **Frontend:** Mobile Apps (iOS/Android), Web (React/Vue).
*   **API Gateway:** Edge servers for authentication, rate limiting, routing.
*   **Services:** User Service, Post Service, Feed Service, Notification Service, Search Service, Recommendation Service.
*   **Messaging:** Kafka/RabbitMQ for async processing, fan-out, notifications.
*   **Storage:**
    *   NoSQL (Cassandra, DynamoDB) for user profiles, posts, activity feeds.
    *   Graph DB (Neo4j) for friend networks.
    *   ElasticSearch for search.
    *   CDN + Object Storage (S3) for media (images, videos).
*   **Cache:** Redis/Memcached for hot data (popular posts, user sessions).
*   **Infra:** Kubernetes, Load Balancers, Auto-scaling.

### ✅ In summary, social media apps face challenges in:
*   **Consistency** → feed lag, duplicate posts, sync issues.
*   **Availability** → service outages, upload failures.
*   **Scalability** → fan-out, hot content, storage, search.
*   **Reliability** → data loss, permanent deletion.
*   **Security & Privacy** → account takeover, data breaches, moderation.
*   **Personalization** → relevance, cold start, real-time adaptation.

---

## 🔹 Common Problems in Movie Booking Applications

### 1. Concurrency & Double Booking
**Problem:**

*   Multiple users try to book the same seat simultaneously.
*   Without proper locking, two users may see the seat as “available” → both pay → conflict.

**Example:**

*   1000 users try booking the last seat in Pathaan’s first show.

**Solutions:**

*   Row-level locking / Pessimistic locking → lock seat record until transaction finishes.
*   Optimistic locking (version numbers) → update fails if seat state changed.
*   Reservation timeout → mark seat as “blocked” for 5 mins, auto-release if not paid.
*   Message queues → serialize booking requests.

### 2. Scalability (High Traffic on Releases)
**Problem:**

*   Popular movie → millions of requests in first minutes.
*   Servers crash or booking lags.

**Examples:**

*   First-day-first-show (FDFS) for a Marvel movie.
*   IPL match tickets released at midnight → sudden spikes.

**Solutions:**

*   Queue system → Virtual waiting room (Cloudflare Waiting Room, Redis queue).
*   Horizontal scaling → auto-scaling APIs, load balancers.
*   CDNs + caching → serve static content (posters, schedules).
*   Event-driven architecture → Kafka/RabbitMQ for async processing.

### 3. Payment Failures & Rollbacks
**Problem:**

*   User selects seats → payment fails → seats stuck in blocked state.
*   Or user pays → booking fails midway → refund headaches.

**Solutions:**

*   Two-phase commit → reserve seats → confirm only after payment success.
*   Idempotent APIs → avoid double charges.
*   Compensation transactions → auto-refund if booking fails.
*   Retry with reconciliation → payment gateway sync jobs.

### 4. Consistency Across Devices
**Problem:**

*   User sees a seat as “available” on mobile → but already booked via web.
*   Stale cache causes wrong availability display.

**Solutions:**

*   Real-time sync → WebSockets/Server-Sent Events for seat updates.
*   Short cache TTL (like 2–5 seconds) for seat inventory.
*   Event sourcing → publish seat status changes to all clients.

### 5. Availability During Peak Loads
**Problem:**

*   System must be up even if DB or payment service partially fails.

**Examples:**

*   DB crashes → no bookings.
*   Payment provider downtime → complete outage.

**Solutions:**

*   Microservices → isolate seat reservation, payments, notifications.
*   Graceful degradation → allow browsing even if booking is down.
*   Multiple payment gateways → failover if one is down.
*   Geo-replication → handle region-specific spikes.

### 6. Fraud & Abuse
**Problem:**

*   Scalpers & bots bulk-book tickets → resell at higher price.
*   Fake bookings block seats without paying.

**Solutions:**

*   CAPTCHA / Bot detection before booking.
*   Rate limiting & throttling per user/IP.
*   Payment-first booking → limit bulk blocking.
*   AI/ML fraud detection → detect suspicious booking patterns.

### 7. User Experience Issues
**Problem:**

*   Jumpy seat availability → user clicks “book” but gets “already taken”.
*   Slow checkout → user loses seats mid-payment.
*   Confusing seat layouts.

**Solutions:**

*   Block + countdown timer for seat reservation.
*   Progressive checkout (lock seats → confirm payment → issue ticket).
*   Smooth UI updates → WebSockets to auto-refresh seat availability.
*   Retry suggestion → recommend nearest available seats if taken.

### 8. Notifications & Communication
**Problem:**

*   Delayed or missing booking confirmation.
*   Multiple tickets sent for same booking.

**Solutions:**

*   Idempotent notifications → unique booking ID for emails/SMS.
*   Async messaging (Kafka/SQS) for ticket generation.
*   Fallback channels → email + SMS + app push.

### 🏗️ High-Level Scalable Architecture
*   **Frontend (Web/Mobile)**
    *   Shows movies, schedules, seat maps.
    *   WebSockets for real-time seat status.
*   **API Gateway**
    *   Auth, rate limiting, routing.
*   **Booking Service**
    *   Handles seat locking, payments, ticketing.
    *   Strong consistency on DB row level.
*   **Inventory DB**
    *   PostgreSQL/MySQL with row-level locks.
    *   Redis for fast seat availability lookups.
*   **Payment Service**
    *   Integrates with Razorpay/Stripe/PayU.
    *   Handles retries, refunds.
*   **Message Queue (Kafka/RabbitMQ)**
    *   For notifications, analytics, async booking events.
*   **Notification Service**
    *   Sends tickets via SMS/Email/Push.
*   **Analytics Service**
    *   Tracks sales, fraud detection, load balancing.

### ✅ In short, common movie booking app problems are:
*   Concurrency & double booking
*   Scalability under high load
*   Payment failures & refunds
*   Consistency across devices
*   Availability during failures
*   Fraud & abuse (scalpers/bots)
*   Poor user experience (slow, conflicting seat states)
*   Notification delays

---

## 🔹 Common Problems of SaaS Platforms

### 1. Multi-Tenancy Challenges

SaaS usually serves many customers (tenants) on shared infrastructure.

**Problems:**

*   **Data Isolation** → Tenant A should never see Tenant B’s data.
*   **Performance Noisy Neighbors** → One heavy tenant can slow down others.
*   **Customizations per Tenant** → Each tenant may need slightly different features.

**Solutions:**

*   Tenant-aware DB schema → separate schema per tenant (Postgres schemas, Mongo collections) or tenant_id filters.
*   Rate limiting / quotas → isolate heavy tenants.
*   Config-driven customization instead of per-tenant code forks.

### 2. Scalability Issues

SaaS must scale horizontally to handle millions of users.

**Problems:**

*   **Traffic Spikes** → sudden load (Zoom calls during COVID).
*   **Database Bottlenecks** → shared DB under heavy load.
*   **Stateful Services** → scaling sticky sessions is hard.

**Solutions:**

*   Microservices + autoscaling (Kubernetes, ECS).
*   Database sharding + read replicas.
*   Stateless services with external session stores (Redis).
*   CDNs for static content.

### 3. Availability & Reliability

Downtime = customer churn in SaaS.

**Problems:**

*   **Single region outages** → downtime for all customers.
*   **Dependency failures** → external API (Stripe, Twilio) failures cascade.
*   **Scheduled downtime** → hard to update without impacting users.

**Solutions:**

*   Multi-region deployments with failover.
*   Circuit breakers + retries for dependencies.
*   Blue-Green / Canary deployments for zero-downtime upgrades.
*   SLO/SLAs monitoring with auto-healing infra.

### 4. Consistency vs Availability

SaaS must balance real-time sync with eventual consistency.

**Problems:**

*   **Out-of-date dashboards** → stale analytics due to async pipelines.
*   **Conflicting updates** → two users edit same document (Google Docs style).
*   **Delayed propagation** → updates in one service take time to reflect in others.

**Solutions:**

*   Event-driven architecture (Kafka, Pulsar) for async sync.
*   Conflict resolution strategies (CRDTs, last-write-wins, OT).
*   Caching with invalidation for near real-time dashboards.

### 5. Security & Compliance

SaaS = high-value target for hackers.

**Problems:**

*   **Data Breaches** → leaks across tenants.
*   **Weak Auth** → account takeover.
*   **Compliance (GDPR, HIPAA, SOC2)**.

**Solutions:**

*   RBAC / ABAC for role-based access.
*   Encryption (in-transit TLS, at-rest AES-256).
*   Audit logs & monitoring.
*   Data residency options (per-region storage).

### 6. Cost Optimization

Cloud infra can explode costs if not managed.

**Problems:**

*   **Over-provisioning** → paying for idle capacity.
*   **Under-provisioning** → latency, poor UX.
*   **Per-tenant cost tracking** → hard to allocate cloud bills.

**Solutions:**

*   Auto-scaling infra (K8s, serverless).
*   Usage-based billing → map infra to tenant usage.
*   Multi-tenant shared resources with isolation.
*   FinOps dashboards.

### 7. Integration & Extensibility

Customers expect SaaS to plug into their ecosystem.

**Problems:**

*   **API Rate Limits** → SaaS APIs hit 3rd party limits.
*   **Versioning** → Breaking changes affect integrations.
*   **Marketplace Apps** → Hard to maintain ecosystem.

**Solutions:**

*   Stable REST/GraphQL APIs with backward compatibility.
*   Webhooks + event-driven APIs for integrations.
*   SDKs + Marketplace for ecosystem support.

### 8. Observability & Monitoring

With many tenants → debugging is tough.

**Problems:**

*   **Shared logs** → hard to filter by tenant.
*   **Performance bottlenecks** → one slow query affects everyone.
*   **Silent failures** → no alerts until tenant complains.

**Solutions:**

*   Tenant-aware logging & tracing (ELK, OpenTelemetry).
*   Rate-limited tenant metrics → detect noisy neighbors.
*   Synthetic monitoring (simulate tenant activity).

🏗️ **Example SaaS Platform Design (High-Level)**

*   Frontend → Multi-tenant React/Next.js app.
*   API Gateway → Auth, rate limiting, tenant-routing.
*   Microservices → Tenant-isolated services (Payments, Users, Billing).
*   DB Layer → Postgres (schema-per-tenant) or DynamoDB with tenant_id.
*   Message Queue → Kafka/PubSub for async events.
*   Monitoring → Prometheus + Grafana + Tenant metrics.
*   Security Layer → JWT, RBAC, tenant isolation policies.
*   Deployment → Kubernetes multi-region clusters.

✅ **In summary, common SaaS platform problems are:**

*   Multi-tenancy (isolation, noisy neighbors, customizations).
*   Scalability (traffic spikes, DB bottlenecks).
*   Availability (failures, zero-downtime deploys).
*   Consistency (real-time sync, conflict resolution).
*   Security (data breaches, compliance).
*   Cost optimization (FinOps).
*   Integration & extensibility (APIs, SDKs).
*   Observability (tenant-aware logs, monitoring).