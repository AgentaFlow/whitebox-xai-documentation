# Offline Mode Guide

## Overview

Offline mode lets the SDK keep working in environments with unreliable network
connectivity. When the API is unreachable, operations are queued in a persistent SQLite
database and synchronized once the connection is restored.

!!! info "Requires SDK 1.0.0 or later"
    Offline mode was not functional before 1.0.0: nothing was ever queued automatically, and
    `sync()` failed against methods that didn't exist. If you're on an older version,
    upgrade — `pip install --upgrade whitebox-xai-sdk` — before relying on any of this.

## Key Features

- **Persistent Queue**: SQLite-based storage survives application restarts
- **Automatic Fallback**: Operations queue themselves on a connection failure — no special
  code path in your application
- **Auto-Sync**: Automatic background synchronization with configurable interval
- **Priority-Based**: Operations processed by priority (CRITICAL > HIGH > NORMAL > LOW)
- **Retry Logic**: Automatic retry with exponential backoff, and a path back for
  permanently-failed operations
- **Thread-Safe**: Concurrent operation support
- **Resource Limits**: Configurable maximum queue size
- **Manual Control**: Optional manual sync for fine-grained control

## What gets queued, and when

With `enable_offline=True`, these operations fall back to the queue automatically:

- `client.models.register()`
- `client.models.update_baseline()`
- `client.predictions.log()`
- `client.predictions.log_batch()`

**Only a genuine connection failure triggers the fallback.** This distinction matters: if the
backend received your request and rejected it — a validation error, an expired credential, a
missing model — that raises normally, as it should. Queueing a request the server has already
refused would just replay the same rejection later while hiding the error from you now.

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

# Operations are queued automatically when the API is unreachable.
# Auto-sync runs in the background every 60 seconds (default).
client.predictions.log(
    model_id=model_id,
    input_data={"amount": 150.50},
    output_data={"prediction": 0, "probability": 0.92},
)

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
failed_ops = client._offline_manager.queue.get_failed_operations()

for op in failed_ops:
    print(f"Operation {op['id']}: {op['last_error']}")
    print(f"Retry count: {op['retry_count']}")
```

!!! note "Queue inspection uses a private attribute"
    The client's day-to-day offline methods are public — `get_offline_status()`,
    `sync_offline_queue()`, `cleanup_offline_queue()`. Reaching the queue object itself for
    inspection or requeueing still goes through `client._offline_manager`, which is private
    and may change. You don't need it for normal operation.

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

An operation that exhausts `max_retries` is marked `failed` and won't be retried on its own.
Use `requeue_failed()` to reset those operations back to `pending` for another attempt —
typically after you've fixed whatever was causing them to fail:

```python
queue = client._offline_manager.queue

# Investigate first
for op in queue.get_failed_operations():
    print(f"Failed: {op['operation_type']}")
    print(f"Error: {op['last_error']}")
    print(f"Data: {op['data']}")

# Reset all permanently-failed operations to pending
count = queue.requeue_failed()
print(f"Requeued {count} operations")

# Then sync
client.sync_offline_queue()
```

!!! tip "Check the error before requeueing"
    `requeue_failed()` resets everything that's failed. If the failures were caused by a
    genuine problem with the data rather than by connectivity, they'll simply fail again —
    read `last_error` first.

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
            input_data=batch,
            output_data=y_pred
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

You don't normally construct this yourself — the client creates one via `OfflineManager`.
Note that it takes a `db_path`, not a directory (`offline_dir` is `OfflineManager`'s
parameter).

```python
from whiteboxxai.offline import OfflineQueue

queue = OfflineQueue(
    db_path="./queue/queue.db",
    max_queue_size=10000,
    auto_sync=True,
)

queue.enqueue(operation_type, data, priority) -> int
"""Add operation to queue."""

queue.dequeue(limit=100) -> List[Tuple[int, OperationType, Dict]]
"""Get pending operations."""

queue.mark_success(operation_id: int)
"""Mark operation as completed."""

queue.mark_failure(operation_id: int, error: str, max_retries: int = 3)
"""Mark operation as failed, retry if under limit."""

queue.get_queue_size(status: str = "pending") -> int
"""Count operations in a given state."""

queue.get_statistics() -> Dict[str, int]
"""Get queue statistics."""

queue.get_failed_operations() -> List[Dict]
"""Get permanently failed operations."""

queue.requeue_failed() -> int
"""Reset permanently-failed operations to pending. Returns the count reset."""

queue.clear_completed(older_than_days: int = 7)
"""Remove completed operations older than the cutoff."""

queue.clear_all()
"""Empty the queue entirely."""
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

manager.cleanup(older_than_days: int = 7)
"""Remove old completed operations."""

manager.queue
"""The underlying OfflineQueue instance."""
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
5. **Error Monitoring**: Track failed operations and investigate patterns —
   `requeue_failed()` only helps once you've fixed the cause
6. **Test Offline Scenarios**: Verify behavior when API is unavailable
7. **Context Managers**: Use `with` statement for proper cleanup
8. **Upgrade to 1.0.0+**: Earlier versions queued nothing and could not sync

## Security

The offline queue stores operation data in SQLite:

- Database file location: `{offline_dir}/queue.db`
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
