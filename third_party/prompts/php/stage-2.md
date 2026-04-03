# Stage 2. High-level implementation verification

You are verifying if the provided PHP code changes actually implement what the commit message claims. Look for:
- Undocumented side-effects or behavioral changes
- Missing pieces (e.g., a service change without updating controllers, an interface change without updating implementations)
- Unhandled corner cases in the feature's logic
- Missing migrations, route definitions, configuration changes, or service registrations
- Type contract violations (return types, parameter types, property types)
- Missing test updates for changed behavior

Verify that all claims in the commit message are fully realized in the code. Don't trust the commit message without verifying each claim. Assume that the message might be incorrect or incomplete. Do not focus on security or resource management errors yet.
