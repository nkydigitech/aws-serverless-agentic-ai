# Module 2: Orchestration Pattern

## Overview

Module 2 demonstrates the **orchestration pattern** for coordinating a multi-agent travel booking workflow using **AWS Step Functions**.

Unlike the event-driven choreography pattern demonstrated in Module 1, where agents communicate by reacting to events, orchestration introduces a central workflow controller.

In this architecture, **AWS Step Functions** controls:

- Workflow execution order
- Parallel execution
- State management
- Conditional branching
- Error handling
- Retries
- Timeouts
- Human-in-the-loop approval

The same three specialized agents from Module 1 are used:

- Planner Agent
- Weather Agent
- Flight Manager Agent

The key architectural difference is that Step Functions explicitly controls how these agents execute and how their outputs move through the workflow.

---

## Learning Objectives

By completing this module, I learned how to:

- Design an orchestrated multi-agent architecture.
- Use AWS Step Functions as a central workflow controller.
- Invoke Lambda-based agents from a Step Functions state machine.
- Execute Weather and Flight agents in parallel.
- Preserve intermediate workflow results using `ResultPath`.
- Implement conditional decision branching.
- Configure retries and exponential backoff.
- Handle workflow errors with `Catch`.
- Configure state and workflow timeouts.
- Implement human-in-the-loop workflows using Step Functions Activities.
- Use activity task tokens to pause and resume workflows.
- Monitor Step Functions executions through the AWS Console and CLI.
- Compare orchestration with event-driven choreography.

---

# Orchestration of Multi-Agent Systems

## What Is Orchestration?

In an orchestrated multi-agent system, a central controller manages the workflow.

Each agent remains an independent component, but the orchestrator determines:

1. Which agent runs first.
2. Which agents run in parallel.
3. What data each agent receives.
4. Which state executes next.
5. How errors are handled.
6. Whether a human needs to intervene.
7. When the workflow is considered complete.

AWS Step Functions provides this orchestration layer.

Instead of agents independently reacting to events, the state machine contains the workflow definition.

---

## Choreography vs Orchestration

Module 1 demonstrated **event-driven choreography**.

Module 2 demonstrates **centralized orchestration**.

| Characteristic | Choreography | Orchestration |
|---|---|---|
| Coordinator | Distributed through events | Central Step Functions workflow |
| Communication | EventBridge events | State transitions and Lambda invocations |
| Workflow visibility | Distributed across rules and events | Centralized in one state machine |
| Parallel execution | Event-driven fan-out | Explicit `Parallel` state |
| State management | Distributed | Step Functions execution context |
| Error handling | Distributed across services | Centralized retry/catch logic |
| Human review | SQS-based workflow | Step Functions Activity |
| Debugging | Requires tracing events | Execution graph and history |
| Workflow order | Emerges from events | Explicitly defined |
| Auditability | Distributed | Centralized execution history |

### Key takeaway

**Choreography is decentralized coordination.**

**Orchestration is centralized coordination.**

Neither pattern is universally better. The appropriate pattern depends on the workflow requirements.

---

# Module 2 Architecture

The orchestration workflow follows this sequence:

```text
                    ┌─────────────────────────┐
                    │     Travel Request      │
                    │       JSON Input        │
                    └────────────┬────────────┘
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │     AWS Step Functions  │
                    │      Orchestrator       │
                    └────────────┬────────────┘
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │      Planner Agent      │
                    │       Extract Trip      │
                    └────────────┬────────────┘
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │        Parallel         │
                    │     Weather + Flight    │
                    └───────┬─────────┬───────┘
                            │         │
                ┌───────────▼───┐ ┌──▼──────────────┐
                │ Weather Agent │ │ Flight Manager  │
                │ Risk Analysis │ │ Flight Search   │
                └───────┬───────┘ └──────┬─────────┘
                        │                 │
                        └────────┬────────┘
                                 ▼
                    ┌─────────────────────────┐
                    │      Planner Agent      │
                    │ Analyze & Make Decision │
                    └────────────┬────────────┘
                                 │
                         ┌───────┴────────┐
                         │                │
                    Auto Approve     Human Review
                         │                │
                         │         ┌──────▼──────┐
                         │         │   Step      │
                         │         │ Functions   │
                         │         │  Activity  │
                         │         └──────┬──────┘
                         │                │
                         │          Human Decision
                         │                │
                         └────────┬───────┘
                                  ▼
                    ┌─────────────────────────┐
                    │  Finalize Booking       │
                    │      Planner Agent      │
                    └────────────┬────────────┘
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │    Booking Success      │
                    └─────────────────────────┘
Agents Used

The orchestration pattern uses the same specialized agents introduced in Module 1.

Planner Agent

The Planner Agent is responsible for:

Extracting travel requirements.
Validating trip details.
Identifying travel dates.
Identifying origin and destination.
Identifying travelers.
Identifying the budget.
Analyzing Weather and Flight results.
Making the booking decision.
Requesting human review when required.
Finalizing the booking.

The Planner participates at multiple stages of the workflow.

Weather Agent

The Weather Agent evaluates travel conditions.

Its responsibilities include:

Receiving travel details.
Analyzing weather conditions.
Assessing travel risk.
Returning structured weather results.
Flight Manager Agent

The Flight Manager Agent evaluates flight options.

Its responsibilities include:

Receiving travel details.
Searching available flights.
Evaluating flight options.
Checking availability.
Returning structured flight information.
Step Functions State Machine

The state machine defines the complete workflow.

The major states are:

PlannerExtract
       │
       ▼
ParallelWeatherAndFlight
       │
       ▼
PlannerAnalyzeAndBook
       │
       ▼
CheckPlannerDecision
       │
   ┌───┴──────────────┐
   │                  │
booked         needs_human_review
   │                  │
   ▼                  ▼
BookingSuccess    WaitForHuman
                       │
                       ▼
                Human Decision
                       │
                       ▼
              Finalize Booking
                       │
                       ▼
                 BookingSuccess
Key State Machine Design Decisions
1. Parallel Execution

Weather and Flight analysis are independent operations.

Step Functions therefore executes them in parallel.

This provides two benefits:

Reduced total workflow time.
Centralized control over both branches.

The orchestrator waits until the required parallel branches complete before continuing.

2. ResultPath

The workflow preserves intermediate results using ResultPath.

Important result paths include:

plannerExtractResult
parallelResults
plannerResult

This allows information returned from earlier states to remain available to later states.

It also improves:

Debugging
Observability
Auditability
State inspection
3. Error Handling

The workflow is designed to handle agent failures without immediately terminating the entire workflow.

Step Functions provides:

Retry
Exponential backoff
Catch
Structured error handling
Timeout management

This moves error-handling logic into the workflow definition instead of requiring every agent to implement the entire workflow-control mechanism.

4. Human-in-the-Loop

The orchestration pattern uses Step Functions Activities for human review.

This differs from Module 1, where SQS was used for human-review requests.

The Activity allows the workflow to pause while waiting for an external human decision.

A task token identifies the specific waiting task.

After the human decision is supplied, the workflow continues.

5. Timeout Management

The human-review state uses a defined timeout.

The workshop specifies a one-hour timeout for human review.

This prevents the workflow from remaining indefinitely in a waiting state.

Required AWS Components

The module requires the following resources:

Resource	Purpose
AWS Lambda	Hosts the three orchestration agents
AWS Step Functions	Central workflow orchestrator
Step Functions Activity	Human-review integration
IAM execution role	Grants Step Functions required permissions
ASL definition	Defines the state machine workflow

The Step Functions execution role must provide permissions required to:

Invoke Lambda functions.
Manage Step Functions operations.
Work with Activities.
Deploying the State Machine

COST ALERT

Creating and executing AWS resources can result in AWS charges. This is especially important when using a personal AWS account.

The commands in this section document the workshop deployment procedure. They should not be executed solely for documentation purposes.

Before deployment, verify the AWS account, region, existing resources, and expected charges.

The workshop uses:

export AWS_REGION=us-west-2
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
export STACK_NAME=orchestration-multi-agent-workshop
Step 1: Retrieve Required ARNs

The state machine requires the ARNs of the Lambda agents, the human-review Activity, and the Step Functions execution role.

export AWS_REGION=us-west-2
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
export STACK_NAME=orchestration-multi-agent-workshop

ORCH_PLANNER_ARN=$(aws lambda get-function \
  --function-name $STACK_NAME-orch-planner-agent \
  --query 'Configuration.FunctionArn' \
  --output text)

ORCH_WEATHER_ARN=$(aws lambda get-function \
  --function-name $STACK_NAME-orch-weather-agent \
  --query 'Configuration.FunctionArn' \
  --output text)

ORCH_FLIGHT_ARN=$(aws lambda get-function \
  --function-name $STACK_NAME-orch-flight-manager-agent \
  --query 'Configuration.FunctionArn' \
  --output text)

ACTIVITY_ARN=$(aws stepfunctions list-activities \
  --query 'activities[?contains(name, `human-review`)].activityArn' \
  --output text)

ROLE_ARN=$(aws iam get-role \
  --role-name $STACK_NAME-stepfunctions-execution-role \
  --query 'Role.Arn' \
  --output text)

echo "Planner Function: $ORCH_PLANNER_ARN"
echo "Weather Function: $ORCH_WEATHER_ARN"
echo "Flight Function: $ORCH_FLIGHT_ARN"
echo "Activity ARN: $ACTIVITY_ARN"
echo "Execution Role: $ROLE_ARN"

These variables allow the ASL definition to reference the actual resources without manually copying long ARN values.

Step 2: Prepare the ASL Definition

The state machine is defined using Amazon States Language (ASL).

The workshop requires creating:

travel-booking-orchestration.json

The ASL definition describes:

State transitions
Lambda invocations
Parallel execution
Result paths
Conditional branching
Retry behavior
Error handling
Human-review Activity
Timeouts

The workshop uses placeholders such as:

${OrchPlannerFunctionArn}
${OrchWeatherFunctionArn}
${OrchFlightFunctionArn}
${HumanReviewActivityArn}

These placeholders are replaced with the actual resource ARNs.

Replace the Resource Placeholders
sed -i.bak "s|\${OrchPlannerFunctionArn}|$ORCH_PLANNER_ARN|g" travel-booking-orchestration.json

sed -i.bak "s|\${OrchWeatherFunctionArn}|$ORCH_WEATHER_ARN|g" travel-booking-orchestration.json

sed -i.bak "s|\${OrchFlightFunctionArn}|$ORCH_FLIGHT_ARN|g" travel-booking-orchestration.json

sed -i.bak "s|\${HumanReviewActivityArn}|$ACTIVITY_ARN|g" travel-booking-orchestration.json

echo "ASL definition prepared with actual ARNs"

The .bak extension creates a backup copy of the file.

Step 3: Create the State Machine

COST ALERT: This command creates an AWS Step Functions state machine. Do not execute it in a personal account unless the deployment is intentional and the associated resources and charges have been reviewed.

aws stepfunctions create-state-machine \
  --name "travel-booking-orchestration" \
  --definition file://travel-booking-orchestration.json \
  --role-arn $ROLE_ARN \
  --type STANDARD

The workshop then retrieves the state machine ARN:

STATE_MACHINE_ARN=$(aws stepfunctions list-state-machines \
  --query 'stateMachines[?contains(name, `travel-booking`)].stateMachineArn' \
  --output text)

echo "State Machine ARN: $STATE_MACHINE_ARN"

The STANDARD workflow type provides execution history that is useful for debugging and auditing.

Executing the Workflow

The completed state machine can be tested using a high-risk travel scenario.

COST ALERT

Starting a Step Functions execution invokes the configured Lambda agents and can generate service usage. Do not run this test simply to reproduce the workshop.

If the resources are no longer required, they should be cleaned up after testing.

High-Risk Booking Scenario

The workshop provides the following test input:

{
  "bookingID": "booking-high-risk-001",
  "userId": "user-test-001",
  "origin": "New York, NY",
  "destination": "Miami, FL",
  "travel_dates": {
    "departure": "2026-09-15",
    "return": "2026-09-20"
  },
  "travelers": {
    "adults": 2,
    "children": 0
  },
  "budget": 800,
  "airline_preference": "American",
  "interests": [
    "beach",
    "nightlife",
    "dining"
  ]
}

Save this as:

high-risk-booking.json
Workshop Content Note

The workshop JSON specifies travel dates of September 15–20, 2026, but the accompanying explanation refers to March travel and hurricane-season conditions.

The documentation preserves the JSON supplied by the workshop rather than silently changing it.

Start the State Machine Execution
export AWS_REGION=us-west-2
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

STATE_MACHINE_ARN="arn:aws:states:$AWS_REGION:$AWS_ACCOUNT_ID:stateMachine:travel-booking-orchestration"

EXEC_ARN=$(aws stepfunctions start-execution \
  --state-machine-arn "$STATE_MACHINE_ARN" \
  --input file://high-risk-booking.json \
  --name "high-risk-test-$(date +%s)" \
  --query executionArn \
  --output text)

echo "Started high-risk execution: $EXEC_ARN"

The execution ARN identifies the specific workflow run.

Monitoring the Execution

The Step Functions Console provides a visual representation of the state machine execution.

The execution graph makes it possible to see:

Which states completed.
Which state is currently active.
Which branch was taken.
Where the workflow is waiting.
State input and output.
Execution history.

The expected workflow is:

PlannerExtract
      ↓
ParallelWeatherAndFlight
      ↓
PlannerAnalyzeAndBook
      ↓
CheckPlannerDecision
      ↓
WaitForHuman

For the high-risk scenario, the workflow should reach the human-review state.

CLI Monitoring

The execution can also be monitored from the AWS CLI.

Check Execution Status
aws stepfunctions describe-execution \
  --execution-arn "$EXEC_ARN" \
  --query '{Status:status,StartDate:startDate,Input:input}' \
  --output json
View Execution History
aws stepfunctions get-execution-history \
  --execution-arn "$EXEC_ARN" \
  --max-items 10 \
  --query 'events[*].{Type:type,Timestamp:timestamp,StateEntered:stateEnteredEventDetails.name}' \
  --output table

This provides a chronological view of state transitions.

Human-in-the-Loop with Step Functions Activities

When the workflow reaches WaitForHuman, the state machine pauses.

The human-review process uses a Step Functions Activity.

The workflow waits for an external worker or reviewer to provide a decision.

This is different from Module 1's SQS-based approach.

Step 4: Retrieve the Pending Activity Task
aws stepfunctions get-activity-task \
  --activity-arn "$ACTIVITY_ARN" \
  --query '{TaskToken:taskToken,Input:input}' \
  --output json

The response contains:

Task token
Booking input

The task token uniquely identifies the waiting workflow task.

The token must be preserved because it is required to resume the execution.

Step 5: Simulate Human Approval

The workshop uses send-task-success to return the human decision.

Replace the placeholder with the actual task token:

TASK_TOKEN="YOUR_TASK_TOKEN_HERE"

Then:

aws stepfunctions send-task-success \
  --task-token "$TASK_TOKEN" \
  --task-output '{
    "decision": "approved",
    "reason": "Customer accepts the risks and wants to proceed despite high weather risk and budget constraints",
    "approved_by": "workshop-participant",
    "approval_timestamp": "2026-11-03T16:25:30Z",
    "risk_acknowledgment": "Customer acknowledges thunderstorm risks and budget overrun"
  }'
Workshop Content Note

The workshop's example approval timestamp is in November 2026, while the test travel dates are in September 2026.

This is retained as workshop-provided example data and should not be interpreted as a logically required production timestamp.

Step 6: Verify Workflow Completion

After human approval, the state machine resumes and continues through the final booking states.

aws stepfunctions describe-execution \
  --execution-arn "$EXEC_ARN" \
  --query '{Status:status,Output:output,StopDate:stopDate}' \
  --output json

The expected final state is:

SUCCEEDED

The final output should contain the booking confirmation and accumulated workflow information.

Understanding the Orchestration Flow
Parallel Execution

The Weather and Flight agents run independently at the same time.

The orchestrator controls the parallel branches and waits for the required results before continuing.

This demonstrates how orchestration can provide concurrency without requiring the agents themselves to coordinate.

Decision Branching

After the parallel analysis, the Planner evaluates the combined results.

The workflow can follow two paths:

Planner Decision
      │
      ├── booked
      │      ↓
      │  BookingSuccess
      │
      └── needs_human_review
             ↓
        WaitForHuman
             ↓
       Human Decision
             ↓
       Finalize Booking

This makes the business decision visible inside the workflow definition.

Human-in-the-Loop Integration

Step Functions Activities allow a workflow to pause while awaiting human input.

The task token connects the external human decision back to the correct workflow execution.

In a production implementation, the external reviewer could be:

A web application
An internal approval system
An operations dashboard
A customer-service workflow

The workshop simulates this process from the CLI.

Error Resilience

The orchestration workflow uses Step Functions capabilities for:

Retry policies
Exponential backoff
Catch blocks
Structured errors
Timeout management

This allows the workflow controller to determine what should happen when an agent fails.

Evidence and Screenshots

Screenshots should be added to:

docs/screenshots/

Use descriptive names following the Module 2 convention:

module2-01-stepfunctions-state-machine.png
module2-02-state-machine-definition.png
module2-03-state-machine-created.png
module2-04-high-risk-execution.png
module2-05-execution-graph.png
module2-06-wait-for-human.png
module2-07-activity-task-token.png
module2-08-human-approval.png
module2-09-execution-succeeded.png

Only include screenshots that actually exist.

Do not create screenshots merely to make the documentation appear complete.

Module 1 vs Module 2

The two modules demonstrate two different ways of coordinating the same multi-agent system.

Module 1: Choreography
Travel Request
      ↓
EventBridge
      ↓
Planner
      ↓
EventBridge
   ↙       ↘
Weather   Flight
   ↘       ↙
     Events
       ↓
    Planner

Coordination is distributed across events and EventBridge rules.

Module 2: Orchestration
             Step Functions
                   │
                   ▼
                Planner
                   │
             ┌─────┴─────┐
             ▼           ▼
          Weather      Flight
             └─────┬─────┘
                   ▼
                Planner
                   │
             ┌─────┴─────┐
             ▼           ▼
          Automatic    Human
           Approval     Review

The workflow is centralized and explicitly defined.

Why Orchestration Matters

Centralized orchestration is particularly useful when a workflow requires:

Strict sequencing.
Conditional branching.
Parallel execution.
Reliable retries.
Timeouts.
Human approval.
Centralized execution history.
Strong auditability.

The workflow becomes easier to inspect because the sequence of states is defined in one place.

Important Technical Concepts Learned
Amazon States Language

ASL is the JSON-based language used to define Step Functions workflows.

It describes the states and transitions that make up the workflow.

ResultPath

ResultPath determines where a state's result is stored in the execution data.

This allows intermediate results to be preserved and passed to later states.

Parallel State

The Parallel state allows independent workflow branches to execute concurrently.

In this module:

Weather Agent
     +
Flight Manager

execute as parallel branches.

Retry

Step Functions can retry failed states automatically.

Retry policies can include exponential backoff to prevent repeatedly hammering a failing service.

Catch

Catch provides controlled error paths when a state cannot successfully complete.

Instead of allowing an error to terminate the entire workflow, the state machine can route execution to an appropriate recovery path.

Timeout

Timeouts prevent workflows from waiting indefinitely.

This is particularly important for human-in-the-loop workflows.

Step Functions Activity

An Activity provides a mechanism for an external worker to retrieve a task and return the result.

In this workshop, it is used to represent human approval.

Key Lessons
1. Orchestration creates centralized control

Step Functions makes the complete workflow visible in one state machine.

2. Agents can remain independently focused

Each agent performs a specialized task while Step Functions manages coordination.

3. Parallelism does not require decentralized coordination

Weather and Flight can run simultaneously while Step Functions maintains control of the overall workflow.

4. State management belongs to the orchestrator

Intermediate results can be preserved using Step Functions state data and ResultPath.

5. Human approval can be part of the workflow

Step Functions Activities allow an automated workflow to pause and resume around human decisions.

6. Error handling becomes part of the architecture

Retry, Catch, and timeout behavior can be defined at the workflow level.

7. Orchestration improves visibility

The Step Functions execution graph and history provide a centralized view of the workflow.

Cost and Account Safety

Because this workshop was performed using AWS resources, cost awareness is part of the implementation process.

For personal AWS accounts:

Confirm the active AWS account before deployment.
Confirm the region.
Avoid creating resources solely for screenshots.
Monitor AWS Billing after hands-on sessions.
Delete temporary resources when the module is complete.
Verify that resources have actually been removed.

The workshop region for this module is:

us-west-2

The deployment commands should therefore not be copied into another AWS account without first verifying the intended environment.

Cleanup Checklist

After completing the hands-on exercise, verify that temporary resources are no longer required.

Potential resources to review include:

[ ] Step Functions state machine
[ ] Step Functions Activity
[ ] Orchestration Lambda functions
[ ] IAM execution role
[ ] IAM policies created specifically for the workshop

Do not delete shared or pre-existing resources without confirming that they belong to the workshop.

For a personal AWS account, cleanup should be followed by verification using the AWS CLI and Console.

Module 2 Completion Checklist
Component	Status
Understand orchestration pattern	Complete
Compare orchestration with choreography	Complete
Understand Step Functions architecture	Complete
Define ASL workflow	Complete
Configure Planner Agent	Complete
Configure Weather Agent	Complete
Configure Flight Manager Agent	Complete
Configure parallel execution	Complete
Configure ResultPath	Complete
Configure retries	Complete
Configure Catch handling	Complete
Configure timeout management	Complete
Configure human-review Activity	Complete
Execute high-risk workflow	Workshop procedure documented
Monitor execution	Workshop procedure documented
Handle human approval	Workshop procedure documented
Verify successful execution	Workshop procedure documented
Document cleanup	Complete
Repository Evidence

The Module 2 documentation belongs in:

docs/
├── module-1/
│   └── README.md
│
├── module-2/
│   └── README.md
│
├── module-3/
│   └── README.md
│
└── screenshots/

Infrastructure definitions belong under:

infrastructure/

Source code belongs under:

src/

Tests belong under:

tests/
Module 2 Summary

Module 2 demonstrates how AWS Step Functions can act as the central orchestration layer for a multi-agent system.

The workflow coordinates:

Planner
   ↓
Weather + Flight
   ↓
Planner Decision
   ↓
Automatic Approval
       OR
Human Review
   ↓
Final Booking

The key difference from Module 1 is that coordination is no longer distributed across EventBridge events.

Instead, the workflow is explicitly defined and managed by Step Functions.

This provides centralized:

Control
State management
Parallel execution
Branching
Error handling
Retry behavior
Timeout management
Human approval
Execution visibility

The module demonstrates why orchestration is useful for workflows where execution order, state, decision logic, error recovery, and human intervention must be explicitly controlled.

Module 2 Status

Documentation prepared.

Hands-on AWS deployment and execution should only be marked complete when supported by actual implementation evidence and screenshots.

Module 2: Orchestration Pattern documented.
