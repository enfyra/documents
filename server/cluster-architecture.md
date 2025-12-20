# Cluster Architecture

Enfyra is designed to run in a cluster environment where multiple instances can work together seamlessly. The system uses Redis for coordination and maintains no state in memory, making it easy to scale horizontally.

## Core Principles

### Stateless Design
Every Enfyra instance is completely stateless. All data lives in the database or Redis cache. This means any instance can handle any request without needing to know about previous requests.

### Shared Resources
All instances in a cluster share the same database and Redis instance. This ensures they all have access to the same data and can coordinate their actions.

### Instance Identity
Each instance gets a unique ID when it starts up. This ID is used to identify which instance is performing actions and to prevent instances from reacting to their own events.

## How Instances Coordinate

### Cache Synchronization
When one instance needs to reload cached data (like table schemas, routes, or packages), it follows this pattern:

1. The instance tries to acquire a lock in Redis
2. If another instance already has the lock, it waits for that instance to finish and broadcast the results
3. If it gets the lock, it loads fresh data from the database
4. It broadcasts the new cache data to all other instances via Redis Pub/Sub
5. Other instances receive the broadcast and update their local cache
6. The lock is released

This ensures that only one instance does the expensive database query, while all instances benefit from the results.

### What Gets Synchronized

Several types of cache are synchronized across instances:

- **Metadata Cache**: Table definitions, columns, and relationships
- **Route Cache**: API route definitions and handlers
- **Package Cache**: List of installed npm packages
- **Storage Config Cache**: File storage configurations
- **AI Config Cache**: AI agent configurations

### Bootstrap Scripts
When the application starts, bootstrap scripts need to run to set up initial data. To prevent all instances from running these scripts at the same time:

1. Each instance tries to acquire a bootstrap execution lock
2. Only one instance gets the lock and runs the scripts
3. Other instances skip bootstrap execution
4. The lock is released when scripts complete

### Session Cleanup
Periodic cleanup of expired sessions also uses distributed locking so only one instance performs cleanup at a time.

## Process Flow Example

Here's what happens when a table schema changes:

1. Instance A receives a request to update a table
2. Instance A acquires a metadata reload lock
3. Instance A updates the database schema
4. Instance A loads fresh metadata from the database
5. Instance A publishes the new metadata to Redis Pub/Sub channel
6. Instances B, C, and D receive the broadcast message
7. Each instance checks if the message came from itself (using instance ID)
8. If not, they update their local cache with the new metadata
9. Instance A releases the lock

The same pattern applies to route updates, package installations, and other cache changes.

## Redis Channels and Keys

The system uses specific Redis channels for different types of synchronization:

- `enfyra:metadata-cache-sync` - Table metadata updates
- `enfyra:route-cache-sync` - Route definition updates
- `enfyra:package-cache-sync` - Package list updates
- `enfyra:storage-config-cache-sync` - Storage configuration updates
- `enfyra:ai-config-cache-sync` - AI configuration updates
- `enfyra:bootstrap-script-reload` - Bootstrap script reload notifications

Locks are stored as Redis keys with specific prefixes and include instance IDs to ensure only the locking instance can release them.

## Node Names and Channels

Each instance can have a node name configured via the `NODE_NAME` environment variable. Redis channels are decorated with the node name, allowing instances on the same physical node to have isolated channels while still participating in the cluster.

If no node name is set, channels use their base names and all instances communicate on the same channels.

## Fault Tolerance

### Redis Connection Loss
If Redis becomes unavailable:
- Instances continue serving requests using their existing cache
- Cache reloads will fail until Redis is restored
- Database operations continue normally
- Once Redis is back, synchronization resumes automatically

### Lock Timeouts
All locks have time-to-live (TTL) values:
- Bootstrap locks: 30 seconds
- Cache reload locks: 30 seconds
- Session cleanup locks: 1 hour

If an instance crashes while holding a lock, the lock automatically expires, allowing other instances to proceed.

### Message Handling
If an instance receives a synchronization message:
- It checks the instance ID to avoid processing its own messages
- It validates the message format before processing
- Errors in message processing are logged but don't crash the instance

## Setup Requirements

To run Enfyra in a cluster:

1. **Shared Redis**: All instances must connect to the same Redis instance
   ```
   REDIS_URI=redis://your-shared-redis:6379
   ```

2. **Shared Database**: All instances must connect to the same database
   ```
   DB_URI=mysql://user:password@your-database-host:3306/enfyra
   # Or for PostgreSQL:
   # DB_URI=postgresql://user:password@your-database-host:5432/enfyra
   ```
   
   > **Note**: If your password contains special characters (`@`, `:`, `/`, `%`, `#`, `?`, `&`), URL-encode them in the URI. For example, password `p@ssw0rd` should be written as `p%40ssw0rd` in the URI.

   **Optional - Read Replicas**: Add `DB_REPLICA_URIS` (comma-separated) to distribute read queries across replicas. Connection pool is automatically distributed between master and replicas. Read queries use round-robin routing. Set `DB_READ_FROM_MASTER=true` to include master in the round-robin pool.

3. **Node Names** (optional): Set unique node names for instances on different physical machines
   ```
   # Instance on Node 1
   NODE_NAME=production-node-1
   
   # Instance on Node 2
   NODE_NAME=production-node-2
   ```

4. **Same Configuration**: All instances should use the same configuration values for consistency

## Benefits

Running Enfyra in a cluster provides:

- **Horizontal Scaling**: Add more instances to handle increased load
- **High Availability**: If one instance fails, others continue serving requests
- **Zero Downtime Deployments**: Deploy new versions by rolling out instances one at a time
- **Load Distribution**: Requests can be distributed across multiple instances

The cluster coordination happens automatically - you just need to ensure all instances share the same Redis and database.
