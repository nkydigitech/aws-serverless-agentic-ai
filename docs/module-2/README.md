# Module 2: Step Functions Orchestration

## Overview

This module implements a serverless multi-agent architecture using AWS Step Functions to orchestrate the agent workflow.

## Architecture

Step Functions acts as the central controller for the multi-agent workflow.

The workflow coordinates:

1. Planner Agent
2. Weather Agent
3. Flight Manager Agent
4. Planner decision
5. Automatic approval or human review
6. Final booking

## Agents

### Planner Agent

The Planner Agent extracts travel details and makes the booking decision.

### Weather Agent

The Weather Agent analyzes weather conditions for the requested destination.

### Flight Manager Agent

The Flight Manager Agent searches and evaluates flight options.

## Workflow

The Step Functions workflow uses:

- Sequential execution
- Parallel execution
- `ResultPath`
- Retry policies
- Exponential backoff
- Error handling
- Timeouts
- Human-in-the-loop approval

## Human-in-the-Loop

High-risk bookings can be routed for human review using a Step Functions Activity.

The workflow waits for the human decision before continuing.

## Testing

Testing and screenshots will be added after the AWS resources are deployed and verified.

## Evidence

Evidence screenshots will be added during the hands-on deployment.

## Deployment Status

Pending AWS deployment.

## Cleanup

AWS resources will be removed after testing to avoid unnecessary ongoing usage.
