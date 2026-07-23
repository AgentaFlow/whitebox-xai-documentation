# Offline Mode Guide

## Overview

The WhiteBoxXAI SDK offline mode enables robust operation in environments with unreliable network connectivity. When the API is unavailable, operations are automatically queued in a persistent SQLite database and synchronized when the connection is restored.

## Key Features

- **Persistent Queue**: SQLite-based storage survives application restarts
- **Auto-Sync**: Automatic background synchronization with configurable interval
- **Priority-Based**: Operations processed by priority (CRITICAL > HIGH > NORMAL > LOW)
- **Retry Logic**: Automatic retry with exponential backoff
- **Thread-Safe**: Concurrent operation support
- **Resource Limits**: Configurable maximum queue size
- **Manual Control**: Optional manual sync for fine-grained control

## Quick Start

### Basic Usage

```python
from whiteboxxai import WhiteBoxXAI

# Initialize client with offline mode
client = WhiteBoxXAI(
    api_key="your_api_key",
    enable_offline=True,
    offline_dir="./offline_queue"
)

# Operations are automatically queued when API is unavailable
# Auto-sync runs in background every 60 seconds (default)

# Check offline status
status = client.get_offline_status()
print(f"Queue size: {status['queue_size']}")
print(f"Statistics: {status['statistics']}")

client.close()
```

### Context Manager

```python
with WhiteBoxXAI(
    api_key="your_api_key",
    enable_offline=True
) as client:
    # Operations here
    # Auto-sync starts automatically
    # Stops automatically on exit
    pass
```

## Configuration

### Client Parameters

```python
client = WhiteBoxXAI(
    api_key="your_api_key",

    # Offline mode settings
    enable_offline=True,                    # Enable offline mode
    offline_dir="./whiteboxxai_offline",    # Queue storage directory
    offline_max_queue_size=10000,          # Max operations (0 = unlimited)
    offline_auto_sync=True,                # Enable auto-sync
    offline_sync_interval=60,              # Sync interval in seconds
)
```

### Parameter Details

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `enable_offline` | bool | False | Enable offline mode |
| `offline_dir` | str | "./whiteboxxai_offline" | Directory for queue database |
| `offline_max_queue_size` | int | 10000 | Maximum queued operations (0 = unlimited) |
| `offline_auto_sync` | bool | True | Enable automatic background sync |
| `offline_sync_interval` | int | 60 | Seconds between sync attempts |

## Operation Types

The offline queue supports four operation types:

```python
from whiteboxxai.offline import OperationType

OperationType.PREDICT          # Prediction logging
OperationType.REGISTER_MODEL   # Model registration
OperationType.UPDATE_BASELINE  # Baseline updates
OperationType.LOG_BATCH        # Batch logging
```

## Priority Levels

Operations are processed by priority:

```python
from whiteboxxai.offline import OperationPriority

OperationPriority.CRITICAL  # Priority 4 - Processed first
OperationPriority.HIGH      # Priority 3
OperationPriority.NORMAL    # Priority 2 (default)
OperationPriority.LOW       # Priority 1 - Processed last
```

Within the same priority, operations are processed FIFO (first-in-first-out).

## Manual Sync Control

### Disable Auto-Sync

```python
client = WhiteBoxXAI(
    api_key="your_api_key",
    enable_offline=True,
    offline_auto_sync=False  # Disable auto-sync
)

# Manually trigger sync when needed
result = client.sync_offline_queue(batch_size=100)
print(f"Synced: {result['synced']}")
print(f"Failed: {result['failed']}")
print(f"Pending: {result['pending']}")
```

### Batch Processing

```python
# Sync in small batches
result = client.sync_offline_queue(batch_size=10)

# Sync all pending (up to limit)
result = client.sync_offline_queue(batch_size=10000)
```

## Queue Management

### Get Queue Status

```python
status = client.get_offline_status()
print(status)
# Output:
# {
#     'offline_enabled': True,
#     'queue_size': 42,
#     'statistics': {
#         'total': 100,
#         'pending': 42,
#         'completed': 55,
#         'failed': 3
#     }
# }
```

### Cleanup Old Operations

```python
# Remove completed operations older than 7 days
client.cleanup_offline_queue(older_than_days=7)

# Clean up immediately (for testing)
client.cleanup_offline_queue(older_than_days=0)
```

### Access Failed Operations

```python
# Get failed operations for investigation
failed_ops = client._offline_manager._queue.get_failed_operations()

for op in failed_ops:
    print(f"Operation {op['id']}: {op['last_error']}")
    print(f"Retry count: {op['retry_count']}")
```

## Error Handling

### Automatic Retry

Failed operations are automatically retried with exponential backoff:

1. First attempt fails → queued for retry
2. Second attempt fails → queued for retry
3. Third attempt fails → marked as permanently failed

```python
# Configure max retries (default: 3)
# Set when creating OfflineManager directly
from whiteboxxai.offline import OfflineManager

manager = OfflineManager(
    offline_dir="./queue",
    max_retries=5  # Custom retry limit
)
```

### Handling Permanent Failures

```python
# Retrieve permanently failed operations
failed = client._offline_manager._queue.get_failed_operations()

# Investigate and potentially re-queue
for op in failed:
    print(f"Failed: {op['operation_type']}")
    print(f"Error: {op['last_error']}")
    print(f"Data: {op['data']}")

    # Could re-queue after fixing issue:
    # client._offline_manager._queue.enqueue(...)
```

## Production Patterns

### High-Reliability Setup

```python
client = WhiteBoxXAI(
    api_key="your_api_key",
    enable_offline=True,
    offline_dir="/var/lib/whiteboxxai/queue",  # Persistent location
    offline_max_queue_size=100000,            # Large queue
    offline_auto_sync=True,
    offline_sync_interval=30,                 # Frequent sync
)
```

### Low-Bandwidth Setup

```python
client = WhiteBoxXAI(
    api_key="your_api_key",
    enable_offline=True,
    offline_auto_sync=True,
    offline_sync_interval=300,  # Sync every 5 minutes
)

# Use batch operations when possible
# Queue accumulates, syncs in batches
```

### Edge Deployment

```python
# Minimal connectivity environment
client = WhiteBoxXAI(
    api_key="your_api_key",
    enable_offline=True,
    offline_dir="./local_queue",
    offline_auto_sync=False,  # Manual control
)

# Make predictions offline
# ...model predictions...

# Manually sync when connection available
try:
    result = client.sync_offline_queue()
    print(f"Synced {result['synced']} operations")
except Exception as e:
    print(f"Sync failed: {e}")
```

## ML Model Integration

### With ModelMonitor

```python
from whiteboxxai import WhiteBoxXAI, ModelMonitor

client = WhiteBoxXAI(
    api_key="your_api_key",
    enable_offline=True
)

# Create monitor
monitor = ModelMonitor(
    client=client,
    model_name="fraud_detector",
    model_type="classification"
)

# Track predictions - automatically queued if offline
y_pred = model.predict(X)
monitor.track(inputs=X, outputs=y_pred)

# Predictions queued and synced automatically
```

### Batch Prediction Pattern

```python
# Process large batches with offline support
predictions = []

for batch in batches:
    y_pred = model.predict(batch)
    predictions.append(y_pred)

    # Log to WhiteBoxXAI - queued if offline
    try:
        client.predictions.log(
            model_id=model_id,
            inputs=batch,
            outputs=y_pred
        )
    except Exception:
        # Automatically queued in offline mode
        pass

# Check queue after processing
status = client.get_offline_status()
print(f"Queued: {status['queue_size']}")
```

## Monitoring

### Queue Health Checks

```python
def check_queue_health(client):
    """Monitor queue health."""
    if not client.is_offline_enabled():
        return {"status": "offline_disabled"}

    status = client.get_offline_status()
    stats = status['statistics']

    # Check queue size
    if stats['pending'] > 5000:
        print("WARNING: Large queue size")

    # Check failure rate
    total = stats['total']
    failed = stats['failed']
    if total > 0 and (failed / total) > 0.1:
        print("WARNING: High failure rate")

    return status

# Run periodically
status = check_queue_health(client)
```

### Metrics Export

```python
# Export queue metrics for monitoring systems
def get_queue_metrics(client):
    """Get metrics for external monitoring."""
    status = client.get_offline_status()
    stats = status['statistics']

    return {
        'whiteboxxai_queue_size': stats['pending'],
        'whiteboxxai_queue_completed': stats['completed'],
        'whiteboxxai_queue_failed': stats['failed'],
        'whiteboxxai_queue_total': stats['total'],
    }

# Send to monitoring system
metrics = get_queue_metrics(client)
# ... send to Azure Monitor / Application Insights, Prometheus, etc.
```

## Troubleshooting

### Queue Database Locked

If you see "database is locked" errors:

```python
# Ensure only one client instance per queue directory
# Or use different offline_dir for each instance

client1 = WhiteBoxXAI(enable_offline=True, offline_dir="./queue1")
client2 = WhiteBoxXAI(enable_offline=True, offline_dir="./queue2")
```

### Queue Growing Too Large

```python
# Monitor and limit queue size
status = client.get_offline_status()
if status['queue_size'] > 10000:
    # Clean up old completed operations
    client.cleanup_offline_queue(older_than_days=1)

    # Or reduce queue size limit
    # (requires recreating client with new limit)
```

### Sync Not Working

```python
# Check if auto-sync is running
if client._offline_manager._sync_running:
    print("Auto-sync is active")
else:
    print("Auto-sync is stopped")
    # Restart if needed
    client._offline_manager.start_auto_sync()

# Force manual sync
result = client.sync_offline_queue()
print(result)
```

## Performance Considerations

### Queue Size Limits

- SQLite performs well up to 100K operations
- Set `offline_max_queue_size` based on available disk space
- Regular cleanup prevents unbounded growth

### Sync Interval Tuning

- **High-frequency** (10-30s): Real-time monitoring, good connectivity
- **Medium** (60-300s): Standard production use
- **Low-frequency** (300+s): Low bandwidth, edge deployments

### Batch Size

- Larger batches: More efficient, but longer sync time
- Smaller batches: More responsive, but more overhead
- Default (100): Good balance for most use cases

## API Reference

### WhiteBoxXAI Client

```python
client.is_offline_enabled() -> bool
"""Check if offline mode is enabled."""

client.sync_offline_queue(batch_size: int = 100) -> Dict[str, int]
"""Manually sync offline queue."""

client.get_offline_status() -> Dict[str, Any]
"""Get offline queue status and statistics."""

client.cleanup_offline_queue(older_than_days: int = 7)
"""Remove old completed operations."""
```

### OfflineQueue

```python
from whiteboxxai.offline import OfflineQueue

queue = OfflineQueue(
    offline_dir="./queue",
    max_queue_size=10000
)

queue.enqueue(operation_type, data, priority) -> int
"""Add operation to queue."""

queue.dequeue(limit=100) -> List[Dict]
"""Get pending operations."""

queue.mark_success(operation_id: int)
"""Mark operation as completed."""

queue.mark_failure(operation_id: int, error: str, max_retries: int = 3)
"""Mark operation as failed, retry if under limit."""

queue.get_statistics() -> Dict[str, int]
"""Get queue statistics."""

queue.get_failed_operations() -> List[Dict]
"""Get permanently failed operations."""
```

### OfflineManager

```python
from whiteboxxai.offline import OfflineManager

manager = OfflineManager(
    offline_dir="./queue",
    max_queue_size=10000,
    auto_sync=True,
    sync_interval=60,
    max_retries=3
)

manager.set_client(client)
"""Set WhiteBoxXAI client for syncing."""

manager.start_auto_sync()
"""Start background sync thread."""

manager.stop_auto_sync()
"""Stop background sync thread."""

manager.sync(batch_size=100) -> Dict[str, int]
"""Manually sync queued operations."""

manager.get_status() -> Dict[str, Any]
"""Get status and statistics."""
```

## Examples

See `sdk/examples/offline_mode_example.py` for complete examples:

1. Basic offline mode with auto-sync
2. Manual sync control
3. Priority-based syncing
4. Queue management
5. ML model integration
6. Error handling and retry
7. Context manager usage

## Best Practices

1. **Enable for Production**: Always use offline mode in production
2. **Monitor Queue Size**: Set up alerts for large queues
3. **Regular Cleanup**: Clean old operations to prevent disk bloat
4. **Persistent Location**: Use `/var/lib` or similar for queue storage
5. **Error Monitoring**: Track failed operations and investigate patterns
6. **Test Offline Scenarios**: Verify behavior when API is unavailable
7. **Context Managers**: Use `with` statement for proper cleanup

## Security

The offline queue stores operation data in SQLite:

- Database file location: `{offline_dir}/offline_queue.db`
- Contains API call data (inputs, outputs, metadata)
- Ensure appropriate file permissions in production
- Consider encryption for sensitive data directories

```bash
# Set appropriate permissions
chmod 700 /var/lib/whiteboxxai/queue
```

## Migration

### From Non-Offline Client

```python
# Before
client = WhiteBoxXAI(api_key="your_api_key")

# After (backward compatible)
client = WhiteBoxXAI(
    api_key="your_api_key",
    enable_offline=True  # Add this line
)
```

No code changes required - offline mode is transparent when enabled.

## Support

For issues or questions:

- GitHub Issues: https://github.com/whiteboxxai/sdk/issues
- Documentation: https://docs.whiteboxxai.com/sdk/offline-mode
- Email: support@whiteboxxai.com
