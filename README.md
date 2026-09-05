# Building Agentic AI Architectures with AWS Serverless

A hands-on implementation and documentation of AWS serverless multi-agent architectures using choreography and orchestration patterns.

## Workshop

**Building Agentic AI Architectures with AWS Serverless**

## Project Focus

This project explores how multiple AI agents can be built, connected, orchestrated, monitored, and extended using AWS serverless services.

### Coordination Patterns

- Choreography using Amazon EventBridge
- Orchestration using AWS Step Functions
- Human-in-the-loop workflows
- Event-driven multi-agent communication
- Parallel agent execution
- Error handling and retries
- Observability and distributed tracing

## Agents

- Planner Agent
- Weather Agent
- Flight Manager Agent
- Hotel Recommendation Agent (optional extension)

## AWS Services

The implementation may use:

- AWS Lambda
- Amazon EventBridge
- AWS Step Functions
- Amazon SQS
- Amazon SNS
- Amazon S3
- Amazon CloudWatch
- AWS X-Ray
- AWS Identity and Access Management (IAM)

## Documentation

Detailed implementation notes are maintained under [`docs/`](./docs/).

- [Workshop Documentation](./docs/README.md)
- [Module 1 - Choreography](./docs/module-1/README.md)
- [Module 2 - Orchestration](./docs/module-2/README.md)
- [Module 3 - Optional Hotel Agent](./docs/module-3/README.md)
- [Infrastructure](./infrastructure/README.md)
- [Source Code](./src/README.md)
- [Tests](./tests/README.md)

## Cost-Control Principle

AWS resources will be created only when required by the workshop.

Before each AWS deployment step, the documentation will identify:

1. Whether the resource can incur charges
2. Whether a free-tier or lower-cost alternative exists
3. What should be deleted after testing
4. How to verify that the resource has been removed

## Important

This repository is both a learning record and a technical portfolio.

Commands, configuration, architecture decisions, screenshots, test results, troubleshooting notes, and cleanup procedures will be documented as the implementation progresses.
