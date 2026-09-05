# Workshop Documentation

## Building Agentic AI Architectures with AWS Serverless

This documentation records the complete implementation of the workshop from environment preparation through deployment, testing, monitoring, and cleanup.

The documentation is being created alongside the implementation so that no deployment phase, command, configuration, or important observation is lost.

---

## Modules

### Module 1 — Choreography

Event-driven coordination using Amazon EventBridge.

[Open Module 1 Documentation](./module-1/README.md)

### Module 2 — Orchestration

Centralized workflow coordination using AWS Step Functions.

[Open Module 2 Documentation](./module-2/README.md)

### Module 3 — Optional Extension

Adding a Hotel Recommendation Agent to the choreography pattern.

[Open Module 3 Documentation](./module-3/README.md)

---

## Documentation Method

For every implementation phase, we record:

- Objective
- Architecture
- Prerequisites
- Commands
- Files created or modified
- AWS resources
- Configuration
- Expected results
- Actual results
- Screenshots
- Troubleshooting
- Cost impact
- Cost-saving alternatives
- Verification
- Cleanup
- Lessons learned

---

## Cost Control

AWS usage will be treated carefully.

Before creating potentially billable resources, a cost warning will be recorded.

### Cost categories

🟢 **No AWS cost / negligible**

Local development, documentation, Git operations, and other actions that do not create billable AWS resources.

🟡 **Potential AWS cost**

Resources or operations that may incur charges depending on usage, region, account status, or free-tier eligibility.

🔴 **Billable risk**

Resources that can generate meaningful charges if left running or used beyond free-tier allowances.

---

## Screenshots

Screenshots will be stored separately and referenced from the relevant module documentation.

See [`screenshots/`](./screenshots/).
