# Image Compression Metadata Race Condition - Root Cause and Fix Plan

## Problem Statement

Intermittent error appears during background image optimization:

`[Image Optimization] FileMetadata not found for ID: {}. Skipping optimization.`

This happens after image upload/save flows when compression is triggered asynchronously.

## Root Cause

Compression is started via `@Async` using `metadataId` immediately after metadata save in `FileService`.
In flows that run inside a broader `@Transactional` boundary, the async worker can start before the transaction commits.

So the async thread does:

- `fileMetadataRepository.findById(metadataId)`

before commit visibility is guaranteed, and gets no row.

This is a transaction visibility race condition between:

- request thread (write transaction), and
- async compression thread (read + process).

## Affected Scenarios

The issue can occur in any flow that triggers optimization before commit:

- regular file upload
- AI image download/store flow
- block media upload

## Proposed Fix (Minimal and Safe)

Keep the async method signature unchanged:

- `optimizeImageAsync(String metadataId)`

Change only trigger timing in `FileService`:

1. Introduce helper method `triggerImageOptimization(...)`.
2. If a transaction is active, register `TransactionSynchronization.afterCommit(...)`.
3. Trigger `imageCompressionService.optimizeImageAsync(metadataId)` only inside `afterCommit`.
4. If no transaction is active, trigger immediately (existing behavior).

## Why We Keep Passing Metadata ID (Not Entity Object)

- Avoid detached JPA entity issues across async/thread boundaries.
- Async worker reads latest committed state from DB.
- Lower coupling between request thread entity state and async processing.
- Minimal code change with clear ownership in `ImageCompressionService`.

## Expected Impact

### User Activity / Performance

- No user-facing blocking introduced (still async).
- Compression start may shift by milliseconds (after commit).
- Throughput remains similar (same executor/pool behavior).

### Concurrency

- Race condition is removed for commit-visibility cases.
- More deterministic behavior under parallel uploads.

### Rollback Semantics

- If transaction rolls back, optimization will not trigger (correct behavior).

## Residual Edge Cases

Rare `not found` can still happen if metadata is deleted by another flow before async execution.
That is separate from this transaction race and should remain a warning-level log.

## Validation Plan

1. Upload one large image and verify compression runs successfully.
2. Validate transactional flow where upload is part of outer `@Transactional` call.
3. Run concurrent large uploads and verify no metadata visibility failures.
4. Force rollback in a test path and confirm optimization is not triggered.
5. Verify non-image and below-threshold images keep current behavior.

## Monitoring After Deployment

Track for 24-72 hours:

- count of `FileMetadata not found` warnings in optimization service
- compression success/failure counts
- executor queue depth and saturation
- compression latency percentiles

## Rollback Plan

If needed, revert trigger helper usage to direct async invocation.
This restores old behavior but may reintroduce the race condition.

## Team Summary

- Root cause: async compression reads metadata before transaction commit.
- Fix: trigger compression in `afterCommit` while still passing `metadataId`.
- Result: preserves async UX, improves reliability under concurrency, and prevents commit-visibility race failures.
