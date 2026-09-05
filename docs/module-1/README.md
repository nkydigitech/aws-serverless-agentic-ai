# Module 1: Event-Driven Multi-Agent Architecture

## Overview

Module 1 demonstrates an event-driven multi-agent architecture for a travel-planning workflow using AWS serverless services.

The architecture uses Amazon EventBridge as the communication layer between three specialized agents:

- Planner Agent
- Weather Agent
- Flight Manager Agent

Supporting AWS services include:

- AWS Lambda for agent execution
- Amazon EventBridge for event-driven communication
- Amazon SQS for human-in-the-loop review
- Amazon S3 for session persistence
- Amazon CloudWatch for monitoring
- AWS IAM for service permissions
- AWS CloudFormation for infrastructure provisioning

The primary architectural pattern demonstrated in this module is **event-driven choreography**.

---

## Learning Objectives

By completing this module, I learned how to:

- Design a multi-agent architecture using AWS serverless services.
- Use EventBridge as an event-driven communication layer.
- Connect Lambda-based agents through events.
- Create EventBridge rules using event patterns.
- Configure EventBridge targets.
- Grant EventBridge permission to invoke Lambda.
- Use a booking ID to correlate workflow events.
- Introduce human approval into an agentic workflow.
- Use SQS as a human-review queue.
- Use S3 for session persistence.
- Use CloudWatch for event monitoring.
- Test Lambda functions independently.
- Diagnose unexpected behaviour in an event-driven architecture.
- Clean up temporary AWS resources after testing.

---

# Architecture

The Module 1 architecture follows an event-driven choreography pattern.

```text
                         Travel Request
                              |
                              v
                    +---------------------+
                    |     EventBridge     |
                    |   Choreography Bus |
                    +----------+----------+
                               |
                               v
                     +-----------------+
                     |  Planner Agent  |
                     |     Lambda      |
                     +--------+--------+
                              |
                       DatesFinalized
                              |
                    +---------+---------+
                    |                   |
                    v                   v
           +-----------------+ +-------------------+
           |  Weather Agent  | |  Flight Manager   |
           |     Lambda      | |      Lambda       |
           +--------+--------+ +---------+---------+
                    |                    |
                    | Weather            | Flight
                    | Completed          | Completed
                    |                    |
                    +---------+----------+
                              |
                              v
                     +-----------------+
                     |  Planner Agent  |
                     | Decision Stage  |
                     +--------+--------+
                              |
                    +---------+---------+
                    |                   |
                    v                   v
                 Low Risk           High Risk
                    |                   |
                    v                   v
              Finalize Booking     Human Review
                                        |
                                        v
                                +---------------+
                                |      SQS      |
                                | Human Review  |
                                +---------------+
The agents communicate through events rather than directly invoking one another.

This reduces direct coupling between the individual agents and demonstrates how event-driven systems can coordinate distributed components.

AWS Services Used
AWS Service	Purpose
AWS Lambda	Hosts the Planner, Weather and Flight Manager agents
Amazon EventBridge	Provides event-driven communication
Amazon SQS	Receives human-review requests
Amazon S3	Stores session information
Amazon CloudWatch	Provides monitoring and logging
AWS IAM	Controls permissions between services
AWS CloudFormation	Provisions the infrastructure foundation
The Three Agents
Planner Agent

The Planner Agent coordinates the travel workflow.

Its responsibilities include:

Extracting travel requirements.
Identifying travel dates.
Identifying origin and destination.
Identifying number of travelers.
Identifying the travel budget.
Publishing the DatesFinalized event.
Receiving results from the Weather and Flight agents.
Making the workflow decision.
Requesting human review when required.
Finalizing the booking.
Planner Tools

The workshop identifies these tools:

extract_travel_details
analyze_and_decide
finalize_booking
Weather Agent

The Weather Agent evaluates weather conditions for the requested destination.

Its responsibilities include:

Retrieving a weather forecast.
Analysing travel risk.
Generating recommendations.
Publishing a weather-completion event.
Weather Tools
get_weather_forecast
analyze_travel_risk
generate_recommendations
Flight Manager Agent

The Flight Manager Agent handles flight-related processing.

Its responsibilities include:

Searching for flights.
Evaluating flight options.
Checking availability.
Returning flight information to the Planner.
Flight Tools
search_flights
evaluate_options
check_availability
Event-Driven Choreography

The central architectural concept in Module 1 is choreography.

There is no single workflow controller coordinating every operation.

Instead, agents publish events and other agents react to events that match their responsibilities.

TravelRequestSubmitted
          |
          v
       Planner
          |
          v
    DatesFinalized
       /       \
      /         \
     v           v
 Weather       Flight
   Agent       Manager
     |           |
     v           v
Weather       Flight
Analysis      Search
Completed     Completed
     \           /
      \         /
       v       v
        Planner

This pattern allows the individual components to remain relatively independent.

EventBridge Event Bus

A custom EventBridge event bus was used for the multi-agent choreography.

The workshop event bus separates the application's custom events from unrelated events on the default event bus.

Evidence

Infrastructure Provisioning

The initial infrastructure foundation was defined with AWS CloudFormation.

The foundation included:

Custom EventBridge event bus.
Planner Lambda.
Weather Lambda.
Flight Manager Lambda.
Lambda execution IAM role.
S3 session bucket.

The workshop stack was:

merged-multi-agent-workshop
Project Structure

CloudFormation Template Validation

Before deployment, the CloudFormation template was validated.

AWS Region

The workshop used:

export AWS_REGION=us-west-2

CloudFormation Stack

The stack outputs were inspected to retrieve the resources required by the workflow.

Deployment

The infrastructure foundation was deployed using CloudFormation.

EventBridge Rules

The workflow uses EventBridge rules to route specific event types to the correct targets.

The main rules were:

Rule	Event	Target
InitialTravelRequestRule	TravelRequestSubmitted	Planner Lambda
PlannerDatesRule	DatesFinalized	Weather + Flight
WeatherCompletedRule	WeatherAnalysisCompleted	Planner Lambda
FlightCompletedRule	FlightSearchCompleted	Planner Lambda
HumanReviewRule	HumanReviewRequired	SQS
HumanApprovalRule	HumanApprovalDecision	Planner Lambda
CatchAllEventsRule	Workshop events	CloudWatch Logs
EventBridge Rules Evidence

EventBridge Targets

Each EventBridge rule requires one or more targets.

For example:

InitialTravelRequestRule
          |
          v
    Planner Lambda

The Planner's DatesFinalized event is routed to both downstream agents:

             Planner
                |
        DatesFinalized
           /        \
          v          v
      Weather      Flight
       Lambda       Lambda

Lambda Invocation Permissions

EventBridge must have permission to invoke the Lambda functions configured as rule targets.

Lambda resource-based permissions were therefore added for the relevant EventBridge rules.

The relationship is:

EventBridge Rule
       |
       | Invoke
       v
Lambda Function

Without this permission, the EventBridge rule may exist and have a target configured, but Lambda invocation will not succeed.

Human-in-the-Loop

The architecture introduces human oversight for situations where the automated workflow should not proceed without additional review.

When the Planner determines that a request requires human intervention, it can publish:

HumanReviewRequired

The event is routed to an SQS queue.

Planner
   |
   | HumanReviewRequired
   v
EventBridge
   |
   v
SQS Human Review Queue
Human Review Queue

The queue used by the workshop was:

multi-agent-human-review

This demonstrates a decoupled human-in-the-loop pattern.

The Planner does not need to directly communicate with the human reviewer.

Human Approval

After reviewing the request, a human decision can be returned to the workflow through EventBridge.

The approval event uses:

Source:
workshop.human-review

and:

DetailType:
HumanApprovalDecision

The event contains a booking identifier so the decision can be correlated with the correct workflow.

Example:

{
  "bookingID": "high-risk-test-456",
  "decision": "approved",
  "reviewer": "workshop-admin"
}

The workshop material contains an inconsistency between bookingID and booking_id in different examples. This is worth noting because consistent event schemas are important in distributed systems.

Session Persistence with Amazon S3

The architecture uses Amazon S3 as a session persistence layer.

The booking identifier can be used to associate session data with a particular travel workflow.

Conceptually:

bookingID
    |
    v
S3 Session Object

The Lambda functions were configured with a SESSION_BUCKET environment variable.

This demonstrates how state can be persisted outside an individual Lambda execution.

CloudWatch Monitoring

Event-driven systems can be difficult to troubleshoot because several services participate in a single workflow.

CloudWatch was therefore used to provide visibility into the EventBridge activity.

The workshop log group was:

/aws/events/multi-agent-workshop

The logs can be used to investigate:

Event timestamps.
Event sources.
Event types.
Event IDs.
Booking IDs.
Workflow progression.

The bookingID acts as an important correlation value when tracing a request across multiple events.

Lambda Testing

The individual Lambda functions were tested independently before relying on the complete event-driven workflow.

The tests confirmed that the Planner, Weather and Flight Manager Lambda functions could be invoked successfully and return structured responses.

This separation between unit-level function testing and distributed workflow testing was useful when troubleshooting the architecture.

Event-Driven Debugging: Recursive Event Loop

During testing, an important architectural issue was discovered.

The EventBridge configuration could create a recursive event flow.

The sequence could become:

Planner
   |
   v
DatesFinalized
   |
   +---------> Weather
   |              |
   |              v
   |       WeatherAnalysisCompleted
   |              |
   |              v
   +----------> Planner
                  |
                  v
            DatesFinalized
                  |
                  +----> Weather
                  |
                  +----> Flight
                         |
                         v
                   FlightSearchCompleted
                         |
                         v
                       Planner
                         |
                         +----> ...

The problem occurred because the Planner was configured to respond to the completion events and could subsequently publish another DatesFinalized event.

This is a classic distributed event-design problem.

How the Event Loop Was Handled

The affected completion rules were disabled during testing:

WeatherCompletedRule
FlightCompletedRule

This prevented the recursive event propagation from continuing while the architecture was being investigated.

The decision was also important from a cost-control perspective because uncontrolled event loops can result in unnecessary Lambda invocations and downstream service usage.

The architecture was not presented as a fully successful end-to-end production workflow after this discovery.

Instead, the issue was documented as part of the hands-on engineering experience.

Engineering Lesson: Event Contracts Matter

The event-loop issue demonstrated that event-driven architecture requires more than simply creating producers and consumers.

A robust implementation should consider:

Clear event ownership.
Precise event patterns.
State-aware transitions.
Idempotency.
Correlation IDs.
Duplicate-event handling.
Loop prevention.
Retry behaviour.
Failure handling.
Dead-letter strategies.
Observability.

A rule that looks correct in isolation can still create an unintended system-level behaviour when combined with other rules.

High-Risk Test Scenario

The workshop provided a high-risk travel request to exercise the workflow.

The test used:

bookingID:
high-risk-test-456

The request included travel information such as:

Origin: LAX
Destination: Miami
Travelers: 2
Budget: 1000
Airline preference: American
Interests: beaches, nightlife and culture

The purpose was to demonstrate how the workflow could reach a human-review path when the travel conditions or booking decision required additional consideration.

Workshop Documentation Note

The workshop material contains a date inconsistency in the example request. The JSON payload uses September 20–23, 2026, while some surrounding instructions refer to March 20–23, 2026.

The implementation documentation therefore preserves the actual payload used rather than silently changing the workshop material.

Expected Event Flow

The intended choreography can be summarized as:

TravelRequestSubmitted
          |
          v
       Planner
          |
          v
    DatesFinalized
       /       \
      v         v
   Weather    Flight
      |         |
      v         v
 Weather      Flight
 Analysis     Search
 Completed    Completed
      \         /
       \       /
        v     v
         Planner
            |
            v
      Decision
       /     \
      v       v
 Finalize   Human Review
              |
              v
          SQS Queue

During the hands-on implementation, testing exposed the recursive-event behaviour described earlier.

Evidence Gallery

All Module 1 screenshots are stored in:

docs/screenshots/
1. Project Structure

2. CloudFormation Template Validation

3. AWS Region Configuration

4. CloudFormation Stack Outputs

5. EventBridge Event Bus

6. CloudFormation Deployment

7. EventBridge Rules

8. EventBridge Targets

9. SQS Human Review

10. S3 Session Store

11. CloudWatch Logs

12. Lambda Test Results

13. AWS Cleanup Verification

14. EventBridge Cleanup

Cleanup

Because the workshop was performed using a personal AWS account, cleanup was treated as an essential part of the hands-on exercise.

The temporary resources created for Module 1 were removed after testing.

Resources cleaned up included:

SQS human-review queue.
CloudWatch event log group.
S3 session bucket.
CloudFormation stack.
Lambda functions.
EventBridge rules.
EventBridge targets.
Custom EventBridge event bus.
Cleanup Verification

The cleanup process was verified rather than assuming that deletion had succeeded.

The CloudFormation stack:

merged-multi-agent-workshop

was confirmed to no longer exist.

The workshop Lambda functions were no longer present.

The human-review SQS queue was removed.

The workshop S3 session bucket was removed.

The EventBridge rules were removed.

The custom EventBridge event bus was removed.

The final EventBridge check showed only the default event bus.

This confirmed that the temporary Module 1 infrastructure had been cleaned up.

Cost Awareness

This module also reinforced the importance of cost awareness when working with AWS.

Several of the services used in this architecture can incur charges depending on usage and how long resources remain active.

Potentially billable services included:

AWS Lambda
Amazon EventBridge
Amazon SQS
Amazon S3
Amazon CloudWatch

The event-loop issue made cost awareness particularly relevant because uncontrolled event propagation could create unnecessary service activity.

The infrastructure was therefore cleaned up after testing.

No AWS resources were retained solely for documentation purposes.

Choreography vs Orchestration

Module 1 demonstrates choreography.

The important characteristic is that the individual agents react to events instead of being controlled by a single central workflow engine.

Agent
  |
  v
EventBridge
  |
  v
Another Agent

Each component knows which events it should consume and what event it should publish next.

This differs from an orchestration pattern where a central workflow engine explicitly controls the sequence.

The next module introduces AWS Step Functions orchestration as an alternative architecture.

Key Takeaways
Event-driven systems require careful design

Creating EventBridge rules is only the beginning.

The interaction between rules must be considered as a complete system.

Agents can be loosely coupled

EventBridge allows agents to communicate without directly depending on each other's implementation.

Human oversight can be part of an agentic workflow

Not every decision should be fully automated.

Human review can be introduced when risk, uncertainty or business policy requires it.

Persistence is important

S3 provides a way to store session information outside an individual Lambda execution.

Observability is essential

CloudWatch helps trace events across distributed components and makes debugging possible.

Distributed AI systems have distributed-systems problems

Agentic applications still require engineering principles such as:

Event contracts.
Correlation IDs.
Idempotency.
Retry handling.
Failure handling.
Loop prevention.
Observability.
Resource cleanup.
Module 1 Completion Status
Component	Status
Project structure	Completed
CloudFormation foundation	Completed
EventBridge event bus	Completed
Planner Lambda	Tested
Weather Lambda	Tested
Flight Manager Lambda	Tested
EventBridge rules	Implemented
EventBridge targets	Implemented
Lambda invocation permissions	Implemented
SQS human review	Implemented
S3 session store	Implemented
CloudWatch monitoring	Implemented
Lambda testing	Completed
Event-loop investigation	Completed
AWS resource cleanup	Completed
Cleanup verification	Completed
Screenshot evidence	Completed
GitHub documentation	Completed
Final Status

Module 1 is complete from a hands-on documentation and infrastructure-lifecycle perspective.

The module demonstrated an event-driven multi-agent architecture using AWS Lambda, EventBridge, SQS, S3, CloudWatch, IAM and CloudFormation.

The infrastructure was:

Defined.
Validated.
Deployed.
Tested.
Investigated.
Debugged.
Cleaned up.
Verified.

The most valuable engineering lesson was discovering that event-driven systems can develop unintended recursive behaviour when event producers and consumers are not carefully designed.

That experience provides a practical foundation for understanding why centralized orchestration with AWS Step Functions can be useful for certain multi-agent workflows.

Repository Evidence
aws-serverless-agentic-ai/
|
+-- docs/
|   |
|   +-- module-1/
|   |   +-- README.md
|   |
|   +-- module-2/
|   |   +-- README.md
|   |
|   +-- module-3/
|   |   +-- README.md
|   |
|   +-- screenshots/
|       +-- module1-01-project-structure.png
|       +-- module1-02-cloudformationvalidate-template.png
|       +-- module1-03-exportawsregion.png
|       +-- module1-04-cloudformationdescribe-stack.png
|       +-- module1-04-eventbridge-event-bus.png
|       +-- module1-05-cloudformationdeploy.png
|       +-- module1-05-eventbridge-rules.png
|       +-- module1-06-eventbridge-targets.png
|       +-- module1-07-sqs-human-review.png
|       +-- module1-08-s3-session-store.png
|       +-- module1-09-cloudwatch-logs.png
|       +-- module1-10-lambda-test-results.png
|       +-- module1-12-aws-cleanup-verification.png
|       +-- module1-13-eventbridge-cleanup.png
|
+-- infrastructure/
|   +-- README.md
|   +-- module-1-foundation.yaml
|
+-- src/
|   +-- README.md
|
+-- tests/
    +-- README.md

Module 1: Completed.
