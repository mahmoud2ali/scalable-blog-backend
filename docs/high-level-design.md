# High Level Design

## components
- Client (App)
- Api Gateway
- Auth Service
- Post Service
- Comment Service
- Cache
- Primary Database
- Read Replicas
- Object Storage
- CDN

## Data Flow Overview
- Clients communicate via REST APIs
- Media is uploaded directly to object storage
- Metadata stored in primary database
- Read-heave traffic served via cache and replicas

