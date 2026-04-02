---
name: php-debug
description: Debug PHP errors and issues
---

Using the prompt REVIEW_DIR/debugging.md, analyze the provided error
information, logs, or stack trace.

Load REVIEW_DIR/technical-patterns.md first, then follow the complete
debugging protocol in debugging.md.

Expected input:
- PHP error/exception output
- Stack trace
- Error logs (Laravel log, Symfony profiler, PHP error_log)
- Reproduction steps

The protocol will:
1. Extract error information (exception type, message, trace)
2. Identify the affected component and framework layer
3. Analyze the code for root cause
4. Check for common PHP pitfalls (type errors, null references, etc.)
5. Create debug-report.txt with findings
