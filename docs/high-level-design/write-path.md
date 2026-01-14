# Write Path


## Create Post

1. Client uploads media to object storage
2. Media URL returned
3. Client sends post metadata to API
4. API validates and stores post in the primary database 
5. Cache invalidation: 
    - Home feed cache
    - Post cache
6. Success response is returned 


## Notes
- Writes go only to primary DB  

## Future Improvements
If write volume or follower count increases, the system can be extended with 
**asynchronous fan-out on write**
- Post creation publishes an event to a message queue.
- Background workers distribute post references to follower feed stores.
- This shifts complexity from read time to write time while keeping write latency low.