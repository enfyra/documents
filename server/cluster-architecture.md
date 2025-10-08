# Cluster Architecture

Enfyra is cluster-ready because it uses Redis for distributed coordination and maintains zero in-memory state. Here's why it works:

## Why Enfyra is Cluster-Ready

### Stateless Architecture
Enfyra stores no state in memory. Everything lives in the database or Redis, so any instance can handle any request.

### Schema Synchronization via Redis
When one instance changes the database schema, it broadcasts the change through Redis Pub/Sub. Other instances automatically reload their schema, keeping everything in sync.

### Distributed Locking
Critical operations like schema updates use Redis-based distributed locks to prevent conflicts. Only one instance can modify the schema at a time.

### Smart Instance Coordination
Each instance has a unique ID and knows its node. This enables intelligent coordination:
- Same instance: Skip (prevent self-triggering)  
- Same node: Lightweight reload
- Different node: Full sync with locking

## How SyncAll Works

1. Instance acquires Redis lock (`schema:sync:processing_lock`)
2. Updates database schema and generates entities
3. Creates backup and publishes event to Redis
4. Other instances receive event and reload schema
5. Lock released, process complete

## Technical Implementation

**Redis Keys Used:**
- `schema:sync:processing_lock` - Prevents concurrent schema changes
- `enfyra:schema-updated` - Pub/Sub channel for change notifications
- `schema:sync:latest` - Tracks latest schema version

**Process Flow:**
```
Schema Change → Acquire Lock → Update DB → Generate Code → Broadcast Event → Other Instances Reload
```

## Fault Tolerance

- **Redis failure**: Instances continue serving, schema changes queue until recovery
- **Sync failure**: Automatic rollback to previous schema version
- **Lock timeout**: 60-second TTL prevents deadlocks
- **Split-brain**: Latest timestamp wins, version reconciliation on recovery

## Environment Setup

**Critical**: All instances must use the same Redis instance for coordination.

```bash
# All instances need the same Redis
REDIS_URL=redis://your-shared-redis:6379

# But different node names for different physical nodes
# Node 1
NODE_NAME=production-node-1

# Node 2  
NODE_NAME=production-node-2
```

**Note**: When using `npx @enfyra/create-server`, you'll be prompted for the node name during setup.

## Result

Multiple Enfyra instances coordinate automatically through Redis. **Requirements**: 
- Same Redis instance for all instances (for synchronization)
- Same database for all instances (for data)
- Unique `NODE_NAME` for instances on different physical nodes