# PHP Debugging Protocol

## Overview

This protocol guides systematic debugging of PHP errors, crashes, and
unexpected behavior.

## Pre-Debug Setup

1. ALWAYS load `technical-patterns.md` first
2. Load subsystem-specific files based on the error context

## Debug Tasks

### TASK 1: Error Classification

Classify the error type:

| Error Type | Examples |
|-----------|----------|
| Fatal Error | Class not found, method on null, memory exhaustion |
| TypeError | Wrong argument type, wrong return type |
| Exception | Uncaught exception, unhandled error |
| Logic Error | Wrong output, unexpected behavior, data corruption |
| Performance | Slow response, high memory usage, N+1 queries |
| Security | Unauthorized access, data leak, injection |

### TASK 2: Stack Trace Analysis

1. Read the full stack trace top-to-bottom
2. Identify the FIRST application frame (skip framework internals)
3. Load the source file at the error line
4. Read surrounding context (full function/method)
5. Identify the variables and state at the error point

### TASK 3: Context Gathering

1. Trace the call path from entry point (route/command) to error site
2. Identify what data flows into the error location
3. Check for:
   - Null values reaching non-nullable parameters
   - Type mismatches (especially from database or user input)
   - Missing configuration or environment variables
   - Race conditions or timing issues
   - Resource exhaustion (memory, connections, file handles)

### TASK 4: Framework Context

Load the appropriate framework guide and check:

**Laravel**:
- Check `storage/logs/laravel.log` for related errors
- Check middleware pipeline for the route
- Check service provider bindings
- Check Eloquent relation definitions
- Check queue worker if job-related

**Symfony**:
- Check profiler/debug toolbar data
- Check service container configuration
- Check event listener registration
- Check Doctrine entity mappings
- Check security firewall configuration

**WordPress**:
- Check `WP_DEBUG_LOG` output
- Check hook execution order
- Check plugin activation/deactivation
- Check database table existence
- Check option values

### TASK 5: Root Cause Identification

1. Identify the root cause (not just the symptom)
2. Trace WHY the bad state exists, not just WHERE it crashes
3. Check if the issue is:
   - A code bug (logic error, missing check)
   - A configuration issue (missing env var, wrong setting)
   - A data issue (corrupted data, missing record)
   - A dependency issue (version mismatch, missing extension)
   - A timing issue (race condition, timeout)

### TASK 6: Report

Create `debug-report.txt` with:

```
ERROR: [error type and message]
FILE: [file path]
FUNCTION: [Class::method() or function()]

STACK TRACE ANALYSIS:
- Entry point: [route/command]
- Error site: [file:function]
- Key frames: [list intermediate calls]

ROOT CAUSE:
[Detailed explanation of why the error occurs]

EVIDENCE:
[Code snippets showing the problem]

SUGGESTED FIX:
[Code changes to resolve the issue]

ADDITIONAL NOTES:
- [PHP version considerations]
- [Framework version considerations]
- [Related issues or side effects]
```
