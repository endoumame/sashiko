# Queue & Job Subsystem Guide

## Queue Safety

### Idempotency
- Jobs may execute multiple times due to retries or infrastructure failures
- ALWAYS design jobs to be safe for repeated execution
- Use unique identifiers to detect duplicate processing
- Database operations: use upserts or check-before-insert patterns

### Job Isolation
- Jobs should not depend on request state (session, auth context)
- Pass all required data via constructor or payload
- Serialize only data, not service objects or connections
- Large payloads: store in database/storage, pass reference ID

### Error Handling
- Define `$tries` and `$timeout` to prevent infinite retries
- Use exponential backoff for transient failures
- `failed()` method for cleanup on permanent failure
- Log job failures with context for debugging

## Framework-Specific

### Laravel Queues
- `ShouldQueue` interface marks class as queueable
- `dispatch()->afterCommit()`: wait for DB transaction commit
- `Bus::chain()`: sequential job execution
- `Bus::batch()`: parallel with progress tracking
- `WithoutOverlapping`: prevent concurrent same-job execution
- `ShouldBeUnique`: prevent duplicate dispatch
- Queue connections: redis, database, sqs - different guarantees
- `php artisan queue:work` vs `queue:listen`: worker vs listener

### Symfony Messenger
- Message + Handler pattern
- Async transport: amqp, doctrine, redis
- Stamps control routing, retry, delay
- `HandlerFailedException` wraps original exception
- Middleware: logging, validation, transaction
- `#[AsMessageHandler]` attribute for handler registration

## Common Queue Regressions

1. **Non-idempotent job**: Duplicate processing creates duplicates
2. **Serialization failure**: Unserializable data in job payload
3. **Missing afterCommit**: Job dispatched before transaction commits
4. **Memory leak**: Long-running worker accumulates memory
5. **Dead letter**: Failed jobs silently lost without monitoring
6. **Timeout**: Job exceeds timeout, restarted mid-operation
7. **Order dependency**: Jobs that assume sequential execution
