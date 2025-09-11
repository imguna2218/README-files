# EvalX Optimization Plan: Phase 1 - Quick Wins

## Overview
This document outlines the first phase of optimizations for evalX, focusing on immediate performance improvements to handle **1000+ requests with 20 test cases each in under 5 seconds**. These changes leverage your existing tech stack (Rust, Axum, Docker, Redis) while preparing for more advanced optimizations in later phases.

**Key Goals for Phase 1:**
- Reduce latency from ~10s+ to <5s for 1000 requests × 20 test cases
- Maintain stability and prevent crashes under high load
- Implement foundation for distributed processing
- Add basic monitoring to identify bottlenecks
- All changes are backward compatible and don't break existing functionality

## Current Architecture Assessment
Your current setup has these characteristics:
- **Web Server**: Axum (Rust) handling HTTP requests
- **Sandboxes**: Docker containers for code execution
- **Queue**: In-memory priority queue with mutex locks
- **Cache**: Redis for storing execution results
- **Limitation**: Single-instance, monolithic design that can't scale horizontally

## Phase 1 Changes Summary

### 1. Enhanced Container Pool Management
**Current State**: Fixed-size container pools that exhaust quickly under load, causing queuing delays.

**Changes Needed:**
- Increase `CONTAINER_POOL_SIZE` to 50 (from current ~10-20) in `.env`
- Add `MAX_CONCURRENT_CONTAINERS=500` to prevent overloading system
- Implement dynamic pool scaling in `src/container_management/container_pool.rs`:
  - Monitor pool utilization (percentage of containers in use)
  - If utilization >90%, spawn additional containers in background
  - If utilization <10%, gradually reduce pool size to conserve resources
  - Use `num_cpus::get() * 2` as maximum container limit for safety

**Benefits:**
- Eliminates container startup delays during traffic spikes
- Reduces average request latency by 30-50%
- Prevents pool exhaustion that causes request failures

**Files to Modify:**
- `.env` - Add new configuration variables
- `src/container_management/container_pool.rs` - Add dynamic scaling logic
- `src/container_management/init.rs` - Initialize larger pools
- `build-images.ps1` - Convert to bash script for Linux compatibility

### 2. Batch Execution Optimization
**Current State**: Each test case potentially recompiles code or uses separate containers.

**Changes Needed:**
- In `src/executor.rs`, modify `execute_batch` function to:
  - Compile code once (for compiled languages)
  - Execute all 20 test cases in parallel using `tokio::spawn` and `futures::join_all`
  - Reuse the same container/artifact for all test cases in a batch
  - Aggregate results efficiently without blocking
- In `src/controllers/executionControllers.rs`, add validation to ensure:
  - All test cases in a batch use the same code and language
  - Use the first request as template for the entire batch

**Benefits:**
- Reduces batch execution time by 70-80% (from ~10s to ~2s)
- Eliminates redundant compilation and container initialization
- Enables true parallel execution of test cases

**Files to Modify:**
- `src/executor.rs` - Major changes to batch execution logic
- `src/controllers/executionControllers.rs` - Add batch validation
- `src/caching/redis_client.rs` - Add batch-level caching

### 3. Redis-Based Distributed Queue
**Current State**: In-memory queue with mutex locks that becomes a bottleneck under high load.

**Changes Needed:**
- Replace in-memory `PriorityQueue` with Redis-based queue in `src/queue_management/manager.rs`:
  - Use Redis Lists with `LPUSH` for enqueue and `RPOP` for dequeue
  - Implement priority using Redis Sorted Sets (`ZADD` with score=priority)
  - Remove mutex locks since Redis handles concurrency
- In `src/queue_management/mod.rs`, update `enqueue`/`dequeue` methods to use Redis connection
- Reduce worker polling frequency from 100ms to 1s since Redis supports blocking pops
- Implement immediate execution path when containers are available and queue is empty

**Benefits:**
- Eliminates queue-based latency under high concurrency
- Enables horizontal scaling (multiple worker instances)
- Provides persistence - survives application restarts
- Reduces memory pressure on main application

**Files to Modify:**
- `src/queue_management/manager.rs` - Complete rewrite of queue logic
- `src/queue_management/mod.rs` - Update interface methods
- `src/queue_management/task.rs` - Add Redis serialization/deserialization
- `src/setup.rs` - Pass Redis connection to queue manager

### 4. Enhanced Caching Strategy
**Current State**: Basic Redis caching that doesn't leverage batch patterns.

**Changes Needed:**
- In `src/caching/redis_client.rs`, implement:
  - Batch-level caching: Cache entire batch results with key derived from code hash + sorted test case hashes
  - Partial cache utilization: Reuse compilation artifacts even when test cases differ
  - Cache hit rate tracking: Increment `cache:hits` and `cache:misses` counters in Redis
- In `Docker/redis/redis.conf`, configure:
  - `maxmemory 512mb` - Limit Redis memory usage
  - `maxmemory-policy allkeys-lru` - Automatically evict least recently used keys
  - `save 300 100` - Persist data every 5 minutes if at least 100 keys changed

**Benefits:**
- Increases cache hit rate from ~20% to ~80% for similar code patterns
- Reduces execution time by 50% for cache hits (<100ms response)
- Prevents Redis memory exhaustion through intelligent eviction

**Files to Modify:**
- `src/caching/redis_client.rs` - Enhanced caching logic
- `Docker/redis/redis.conf` - Redis configuration updates
- `src/executor.rs` - Integrate with enhanced cache checking

### 5. Basic Performance Monitoring
**Current State**: No visibility into system performance under load.

**Changes Needed:**
- Add `prometheus = "0.13"` to `Cargo.toml` dependencies
- Create `src/monitoring/` directory with:
  - `mod.rs` - Module declaration
  - `metrics.rs` - Define Prometheus metrics (queue depth, container usage, cache hit rate)
- In `src/routes.rs`, add `/metrics` endpoint exposing Prometheus data
- Install and configure Grafana for visualization:
  - `sudo apt install grafana`
  - Configure Prometheus as data source
  - Create dashboard with key metrics

**Benefits:**
- Identifies bottlenecks before they impact users
- Provides data-driven optimization decisions
- Enables capacity planning based on actual usage patterns

**Files to Modify:**
- `Cargo.toml` - Add Prometheus dependency
- New: `src/monitoring/mod.rs` - Monitoring module
- New: `src/monitoring/metrics.rs` - Metric definitions
- `src/routes.rs` - Add metrics endpoint

### 6. Local Orchestration with Docker Compose
**Current State**: Manual process for starting application components.

**Changes Needed:**
- Create `docker-compose.yml` with services:
  - `app` - Main Axum application (build from Dockerfile)
  - `redis` - Redis server with custom configuration
  - `worker` - Worker processes (multiple replicas)
- Configure service dependencies and health checks
- Set up named volumes for Redis persistence
- Create startup script that builds images and starts composition

**Benefits:**
- Consistent development environment
- Easy scaling of worker processes
- Simplified dependency management
- Preparation for production deployment

**Files to Modify:**
- New: `docker-compose.yml` - Multi-container orchestration
- Update: `Dockerfile` (if exists) or create new one for application
- New: `start-dev.sh` - Development environment startup script

## Implementation Sequence

1. **First**: Convert `build-images.ps1` to `build-images.sh` and test Docker image building
2. **Second**: Implement Redis-based queue to eliminate immediate bottlenecks
3. **Third**: Enhance container pool management for better resource utilization
4. **Fourth**: Optimize batch execution for parallel test case processing
5. **Fifth**: Add monitoring to validate improvements and identify new bottlenecks
6. **Finally**: Set up Docker Compose for simplified development and testing

## Testing Strategy

**Load Testing Tools:**
- `wrk` - HTTP load testing (`wrk -t10 -c100 -d30s http://localhost:3000/execute-batch`)
- `redis-cli` - Monitor queue length and cache performance
- Custom scripts - Generate test batches with 20 test cases each

**Success Metrics:**
- ✅ 1000 requests × 20 test cases completed in <5 seconds
- ✅ No failed requests due to queue full errors
- ✅ CPU utilization <80% during peak load
- ✅ Memory usage stable without leaks

## Risk Mitigation

1. **Backward Compatibility**: All changes maintain existing API contracts
2. **Fallback Mechanisms**: Redis queue falls back to in-memory if Redis unavailable
3. **Gradual Deployment**: Implement features behind feature flags where possible
4. **Monitoring**: Extensive metrics to quickly identify regressions

## Expected Results

After Phase 1 implementation, evalX will:
- Handle 1000+ requests with 20 test cases each in under 5 seconds
- Scale to 10,000+ requests through better resource utilization
- Provide visibility into system performance through monitoring
- Establish foundation for Phase 2 optimizations (Isolate integration)

This phase focuses on maximizing performance from your current stack without introducing new dependencies, ensuring stability while dramatically improving throughput and latency.
