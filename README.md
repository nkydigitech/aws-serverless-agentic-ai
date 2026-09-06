# AWS Serverless Agentic AI — Choreography vs Orchestration

![Banner](docs/screenshots/aws_serverless_banner.jpg)

![AWS](https://img.shields.io/badge/AWS-Serverless-FF9900?style=for-the-badge&logo=amazon-aws)
![Lambda](https://img.shields.io/badge/Agents-Lambda_Python-3776AB?style=for-the-badge&logo=python)
![EventBridge](https://img.shields.io/badge/Choreography-EventBridge-FF4F8B?style=for-the-badge)
![Step Functions](https://img.shields.io/badge/Orchestration-Step_Functions-CA0542?style=for-the-badge)
![Bedrock](https://img.shields.io/badge/Bedrock-AgentCore-00A1C9?style=for-the-badge)
![Cost](https://img.shields.io/badge/Cost-%240.52→%240.00-00C853?style=for-the-badge)
![Screenshots](https://img.shields.io/badge/Screenshots-43_Verified-7B42BC?style=for-the-badge)

> **I turn manual, 3 AM-breaking deployments into 1-min automated pipelines with AWS + Ansible + Terraform.** This repo proves it: 3 Lambda agents, EventBridge choreography, Step Functions orchestration with human-in-the-loop for $800 high-risk bookings, CloudWatch + X-Ray observability. Cost: **$0.52 Sept bill → $0.00 after cleanup.**

**Author:** Nkechi Anna Ahanonye — Cloud & DevOps Engineer | Featured: 15-Module Ansible Lab with real terminal | [GitHub @nkydigitech](https://github.com/nkydigitech)

---

## 📖 Summary

Over the course of this workshop, you explore how to build multi-agent systems on AWS using two different coordination patterns — **choreography with EventBridge and orchestration with Step Functions**. You also learn how to layer in observability using CloudWatch Logs, X-Ray, and tracing features built into AWS services.

In real systems, you may combine both patterns — using EventBridge for broad distribution and Step Functions for critical orchestrated workflows.

**You built, observed, and understood the future of serverless multi-agent systems on AWS.**

### What You Built

- **Agents on Lambda** — Planner, Weather, and Flight Booking agents, each with their own personality and tools.
- **Choreography with EventBridge** — loosely coupled agents reacting to events (DatesFinalized, FlightSearchCompleted, etc.). Scales easily by adding new rules and targets.
- **Orchestration with Step Functions** — centralized state machine controlling sequence, branching, retries, and HITL paths. Provides built-in correlation and visualization.
- **Human-in-the-Loop (HITL)** — escalation path via SQS/Activity where bookings can be paused for manual approval before proceeding.
- **Observability** — logs correlated by booking ID, tracing enabled across EventBridge, Step Functions, and Lambdas, and distributed visualization in X-Ray.

---

## 📸 Architecture — Two Patterns

### Module 1: Foundation — Choreography Pattern (Decentralized, Event-Driven)
![Module 1 Choreography](docs/screenshots/module_1_choreography_aws_serverless_architectur.jpg)

**Pattern:** No central boss. Each service reacts to events.

- **Stack:** `module-1-foundation.yaml` → `orchestration-multi-agent-workshop` base — `CREATE_COMPLETE` (screenshot `module-1/04-cloudformation-describe.png`)
- **Event Bus:** Custom EventBridge Bus `travel-booking-bus` (screenshot `05-eventbridge-bus.png`)
- **Rules & Targets:** 2 Rules filter `booking.created` → Targets: Lambda (planner) + SQS `human-review-queue` (screenshots `07-eventbridge-rules.png` + `08-eventbridge-targets.png`)
- **Storage:** S3 `sessionbucket-xxx` session store (screenshot `10-s3-session.png`)
- **Human Review:** SQS `human-review-queue` (screenshot `09-sqs-human-review.png`)
- **Observability:** CloudWatch Logs + Lambda test (screenshots `11-cloudwatch-logs.png` + `12-lambda-test.png`)
- **Flow:** `User books → EventBridge Bus → Rules → Lambda processes + SQS holds for human → S3 saves session → CloudWatch logs`
- **Screenshots:** `docs/screenshots/module-1/01-project-structure.png` → `14-eventbridge-cleanup.png` (14 images, sequential)
- **Cost:** $0.38 Bedrock AgentCore (us-east-1) + $0.00 Lambda/S3/CFN

**When to use choreography:** You expect to add/remove agents frequently. Agents run independently with minimal coupling. Event-driven scalability is priority.

### Module 2: Orchestration Pattern (Centralized, Human-in-Loop) ⭐
![Module 2 Orchestration](docs/screenshots/aws_step_functions_orchestration_diagram.jpg)

**Pattern:** Central boss (Step Functions) controls sequence, branching, retries, HITL. Built-in correlation, visualization, auditability.

- **Stack:** `orchestration-multi-agent-workshop` — CREATE_COMPLETE (screenshot `module-2/module2-01-cloudformation-stack.png`)
- **State Machine:** `travel-booking-orchestration` — ARN `arn:aws:states:us-west-2:309307206565:stateMachine:travel-booking-orchestration` (screenshot `module2-03-stepfunctions-state-machine-and-activity.png`)
- **Resources (7):** Activity + 3 Lambdas + 2 IAM Roles + S3 Bucket (screenshot `module2-02-stack-resources.png` + CLI `02b-stack-resources-cli.png`)
- **Agents (3 Lambdas):**
  - `orch-planner-agent` — Parses trip request, decides flow
  - `orch-weather-agent` — Parallel check weather
  - `orch-flight-manager-agent` — Parallel check flights
- **Human Review:** Activity `orchestration-multi-agent-workshop-human-review-activity` ARN (screenshot `04-activity-details.png`) — Blocks execution with taskToken
- **IAM Roles (2):** `shared-agent-execution-role` (AgentS3Access + LambdaBasicExecutionRole) + `stepfunctions-execution-role` (StepFunctionsLambdaAccess) (screenshots `05a-iam-shared-agent-role.png` + `05b-iam-stepfunctions-role.png`)
- **S3:** Session bucket `orchestration-multi-agent-workshop-sessionbucket-xxx` (screenshot `06-s3-session-bucket.png` + `s3bucket.png`)
- **Flow (High-Risk $800 Test — Requires Human Approval):**
  ```
  module2-10-start-execution.png (cost:800, trip_id high-risk-test)
  → module2-11-execution-running-wait-human.png (BLUE - State: HumanReview, waiting for taskToken)
  → module2-12-execution-details-input.png (input shows {"cost":800})
  → module2-13-activity-task-token.png + 13-activity-task-token-json.png (AQCsAAAAKg... temporary, expires 3600s, stored in token.json - gitignored)
  → module2-14-send-task-success.png (send-task-success {"approved":true,"reviewer":"admin"})
  → module2-15-execution-succeeded.png (Status: SUCCEEDED)
  → module2-16-final-graph-all-green.png (Graph: all states green - audit proof)
  ```
- **Screenshots:** `module-2/module2-00-personal-account-verified.png` → `16-final-graph-all-green.png` (29 images, no spaces, no duplicates)
- **Cost:** **$0.00** Lambda/S3/CFN/Step Functions/CloudWatch — Verified bill $0.00, stack deleted (screenshot `08-cleanup-verification.png` shows DELETE_COMPLETE, S3 error NoSuchBucket = already deleted = $0)

**When to use orchestration:** Order of execution matters. You need retries, error handling, compensation logic. Compliance requires full audit trails.

**Real systems combine both:** EventBridge for broad distribution + Step Functions for critical orchestrated workflows.

---

## 🎓 Key Learnings

1. **Serverless agents scale naturally:** Running on AWS Lambda removes infrastructure overhead and provides cost efficiency (pay only for execution time).
2. **Choreography = flexibility:** EventBridge fan-out makes it easy to extend systems by adding new agents without touching existing code.
3. **Orchestration = control:** Step Functions provide strict sequencing, error handling, and auditability, which is essential for workflows requiring guarantees.
4. **HITL is essential for trust:** Not all decisions should be automated; routing to SQS/Activity for manual review makes systems more reliable and compliant.
5. **Observability is non-negotiable:** CloudWatch, Logs Insights, and X-Ray give you visibility to debug, trace, and optimize distributed systems at scale.

---

## 📁 Repo Structure — 43 Screenshots Verified

```
.
├── README.md (this file)
├── .gitignore (token.json, last-exec-arn.txt, **/token.json - task tokens expire 3600s)
├── docs/
│   ├── module-1/README.md
│   ├── module-2/README.md
│   ├── module-3/README.md
│   └── screenshots/
│       ├── aws_serverless_banner.jpg (1280x640 - GitHub social preview)
│       ├── aws_step_functions_orchestration_diagram.jpg (Module 2 central)
│       ├── module_1_choreography_aws_serverless_architectur.jpg (Module 1 decentralized)
│       ├── module-1/ (14 - Choreography Foundation)
│       │   ├── 01-project-structure.png
│       │   ├── 02-cloudformation-validate.png
│       │   ├── 03-export-aws-region.png
│       │   ├── 04-cloudformation-describe.png
│       │   ├── 05-eventbridge-bus.png
│       │   ├── 06-cloudformation-deploy.png
│       │   ├── 07-eventbridge-rules.png
│       │   ├── 08-eventbridge-targets.png
│       │   ├── 09-sqs-human-review.png
│       │   ├── 10-s3-session.png
│       │   ├── 11-cloudwatch-logs.png
│       │   ├── 12-lambda-test.png
│       │   ├── 13-cleanup.png
│       │   └── 14-eventbridge-cleanup.png
│       └── module-2/ (29 - Orchestration + HITL)
│           ├── module2-00-personal-account-verified.png
│           ├── module2-01-cloudformation-stack.png (CREATE_COMPLETE)
│           ├── module2-02-stack-resources.png (7 resources: Activity + 3 Lambdas + 2 Roles + S3)
│           ├── module2-02b-stack-resources-cli.png + module2-02c-cloudformation-cli.png
│           ├── module2-03-stack-create-complete.png + module2-03-stepfunctions-state-machine-and-activity.png
│           ├── module2-04-activity-details.png + module2-04-arns-collected.png
│           ├── module2-05-asl-file-created.png + module2-06-asl-prepared-with-arns.png + module2-07-state-machine-created.png
│           ├── module2-05a-iam-shared-agent-role.png + module2-05b-iam-stepfunctions-role.png + module2-05c-iam-roles-list.png + module2-05d-iam-cli.png
│           ├── module2-06-s3-session-bucket.png + module2-07-lambda-functions.png + module2-08-cleanup-verification.png + module2-09-cloudwatch-logs.png
│           ├── module2-09-high-risk-input.png (cost 800)
│           ├── module2-10-start-execution.png
│           ├── module2-11-execution-running-wait-human.png (BLUE - waiting for human)
│           ├── module2-12-execution-details-input.png
│           ├── module2-13-activity-task-token-json.png + module2-13-activity-task-token.png
│           ├── module2-14-send-task-success.png (approve)
│           ├── module2-15-execution-succeeded.png (SUCCEEDED)
│           └── module2-16-final-graph-all-green.png (audit proof)
├── infrastructure/
│   ├── module-1-foundation.yaml
│   └── module2/
│       ├── foundation.json
│       └── travel-booking-orchestration.json (ASL state machine definition)
├── src/module2/ (Lambda source - planner, weather, flight)
└── tests/module2/
```

## 🚀 Quick Start — 1-Min Automated Pipeline

```bash
# Module 1: Deploy Foundation (Choreography)
aws cloudformation deploy --template-file infrastructure/module-1-foundation.yaml --stack-name travel-booking-foundation --capabilities CAPABILITY_NAMED_IAM --region us-west-2
aws events list-rules --region us-west-2 --query 'Rules[?contains(Name,`orchestration`)].Name' --output table

# Module 2: Deploy Orchestration (Centralized)
export STACK=orchestration-multi-agent-workshop
aws cloudformation deploy --template-file infrastructure/module2/foundation.json --stack-name $STACK --capabilities CAPABILITY_NAMED_IAM --region us-west-2
aws cloudformation list-stack-resources --stack-name $STACK --region us-west-2 --output table

# Test High-Risk $800 → Human-in-Loop
aws stepfunctions list-state-machines --region us-west-2 --query 'stateMachines[].[name,stateMachineArn]' --output table
aws stepfunctions list-activities --region us-west-2 --query 'activities[].[name]' --output table

aws stepfunctions start-execution --state-machine-arn arn:aws:states:us-west-2:309307206565:stateMachine:travel-booking-orchestration --input "{\"trip_id\":\"high-risk-test-$(date +%s)\",\"user_id\":\"user123\",\"cost\":800}" --region us-west-2 --query 'executionArn' --output text | tee last-exec-arn.txt
# → Check console: 11-execution-running-wait-human.png (BLUE)

aws stepfunctions get-activity-task --activity-arn arn:aws:states:us-west-2:309307206565:activity:orchestration-multi-agent-workshop-human-review-activity --region us-west-2 --query 'taskToken' --output text > token.json
cat token.json # AQCsAAAAKg... (temp, 3600s)

aws stepfunctions send-task-success --task-token file://token.json --task-output '{"approved":true,"reviewer":"admin"}' --region us-west-2
# → 14-send-task-success.png

aws stepfunctions describe-execution --execution-arn $(cat last-exec-arn.txt) --region us-west-2 --query 'status' --output text
# SUCCEEDED → 15-execution-succeeded.png + 16-final-graph-all-green.png

# Cleanup → $0 Cost
aws s3 ls --region us-west-2 | grep orchestration
aws s3 rm s3://orchestration-multi-agent-workshop-sessionbucket-xxx --recursive --region us-west-2
aws cloudformation delete-stack --stack-name $STACK --region us-west-2
aws cloudformation list-stacks --region us-west-2 --query 'StackSummaries[?contains(StackName,`orchestration`) && StackStatus!=`DELETE_COMPLETE`].[StackName,StackStatus]' --output table
# Should be empty = $0.00
aws logs describe-log-groups --region us-west-2 --query 'logGroups[?contains(logGroupName,`orchestration`)].logGroupName' --output text | tr '\t' '\n' | xargs -I {} aws logs delete-log-group --log-group-name {} --region us-west-2

# For full $0 after ALL modules graded, delete Bedrock resources too (us-east-1):
# Console > Bedrock > AgentCore > delete runtimes
```

## 💰 Cost — Verified Bill $0.52 Sept → $0.00

| Service | Module 1 | Module 2 | Cleanup Proof |
|---------|----------|----------|---------------|
| Lambda | $0.00 | $0.00 | Stack deleted |
| S3 | $0.00 | $0.00 | Bucket emptied + NoSuchBucket error = deleted |
| CloudFormation | $0.00 | $0.00 | DELETE_COMPLETE (screenshot 08) |
| CloudWatch Logs | $0.00 | $0.00 | Log groups deleted |
| Step Functions | - | $0.00 | State machine + Activity deleted |
| EventBridge | $0.09 + $0.00 | - | Rules deleted (14-eventbridge-cleanup.png) |
| Bedrock AgentCore | $0.38 us-east-1 | - | Delete runtime after grading |
| KMS | $0.01 | - | - |
| **Total Sept 2026** | **$0.52** | **$0.00** | **$0 next month** |

## 🔒 Security — No Credentials Exposed

```bash
grep -r "AKIA" . --exclude-dir=.git --exclude-dir=node_modules # empty = safe
grep -r "aws_secret_access_key" . --exclude-dir=.git # empty
git log --all --full-history -- "*token.json*" --oneline # empty = never committed
```

- `.gitignore`: `token.json`, `last-exec-arn.txt`, `**/token.json`, `**/last-exec-arn.txt` — Task tokens are temporary (3600s), expire after SUCCEEDED, useless
- Screenshots only show Account ID `309307206565` in ARNs (not secret, public to your account)
- `foundation.json` + `travel-booking-orchestration.json` tracked — contain ARNs, no secrets — safe
- Push `97fb712` verified: 41 objects, 2.81 MiB, branch `module-2-orchestration` → `origin`

## 🔧 Tech Stack — My 3 AM Fix

**I turn manual, 3 AM-breaking deployments into 1-min automated pipelines with AWS + Ansible + Terraform:**

- **IaC:** CloudFormation, Terraform, 15-Module Ansible Lab (real terminal)
- **Automation:** Ansible + AWS + GitHub Actions
- **AWS:** Lambda Python 3.11, Step Functions (Orchestration), EventBridge (Choreography), SQS (HITL), S3, IAM, CloudWatch, X-Ray, Bedrock AgentCore
- **Pattern:** Choreography (flexibility, scale) vs Orchestration (control, audit, retries) + HITL (trust)
- **Cost:** $0.52 → $0.00 cleanup verified

## 👩‍💻 Author — Cloud & DevOps Engineer

**Nkechi Anna Ahanonye**  
Cloud & DevOps Engineer | I turn manual, 3 AM-breaking deployments into 1-min automated pipelines with AWS + Ansible + Terraform

- **Featured Build:** 15-Module Ansible Lab with real terminal (my flagship)
- **Current Build:** This repo — AWS Serverless Agentic AI (Modules 1-3) + Terraform EKS pipelines + Ansible automation
- **Location:** Ikorodu, Lagos, Nigeria — Remote Global
- **GitHub:** [@nkydigitech](https://github.com/nkydigitech) — 43 screenshots, $0.00 cost proof
- **LinkedIn:** Nkechi Anna Ahanonye

**Current LinkedIn Headline:**
> Cloud & DevOps Engineer | I turn manual, 3 AM-breaking deployments into 1-min automated pipelines with AWS + Ansible + Terraform | Featured: 15-Module Ansible Lab with real terminal

**New Headline Suggestion (with this repo):**
> Cloud & DevOps Engineer | AWS Serverless Agentic AI (Choreography vs Orchestration + HITL + X-Ray) + 15-Module Ansible Lab | I turn 3 AM failures into 1-min pipelines with Terraform, Step Functions | Cost: $0.52→$0.00

## 📌 GitHub Right Side (About) — Set This Now

Go to repo `nkydigitech/aws-serverless-agentic-ai` → ⚙️ (gear icon next to About on right side) → Edit:

**Description:**
```
🚀 Serverless Multi-Agent Systems: Choreography (EventBridge) vs Orchestration (Step Functions + HITL for $800 bookings) + Observability (CloudWatch/X-Ray) | 43 screenshots | Lambda Planner/Weather/Flight | $0.52→$0.00 | Part of 15-Module Ansible + Terraform series
```

**Website:**
```
https://github.com/nkydigitech/aws-serverless-agentic-ai
```

**Topics (Add topics):**
```
aws lambda step-functions eventbridge bedrock agentcore ansible terraform devops cloudformation python serverless multi-agent orchestration choreography human-in-the-loop sqs s3 cloudwatch x-ray iac observability
```

**Checkboxes:**
- ✅ Include in the home page
- Releases, Packages default

**Social Preview:** Settings → General → Social preview → Upload `docs/screenshots/aws_serverless_banner.jpg` (1280x640)

---

## Wrap-Up — End of Workshop

By completing this workshop, you now have:

- Practical understanding of multi-agent systems on AWS.
- Hands-on experience building both choreographed and orchestrated workflows.
- Ability to add observability and HITL to make these systems production-ready.

You've seen how AWS services like Lambda, EventBridge, Step Functions, SQS, and X-Ray can work together to deliver intelligent, scalable, and resilient systems. With these building blocks, you can extend beyond flight booking and design your own event-driven, multi-agent applications.

**Congratulations — you've built, observed, and understood the future of serverless multi-agent systems on AWS!**

**Clean Up:** If running as part of AWS-hosted event (re:Invent, Immersion Day, Summit lab), environment cleans automatically. For personal account: delete CloudFormation stacks, S3 buckets (empty first), CloudWatch log groups, Step Functions state machines & activities, EventBridge rules/bus, and Bedrock AgentCore runtimes in us-east-1 for full $0.

End of Workshop — Thank you for participating in Building Serverless Multi-Agent Systems. You now have foundations to design intelligent, event-driven applications that scale automatically, remain flexible, and are observable end-to-end.

---

⭐ Star this if it helped you understand Choreography vs Orchestration vs HITL! PRs welcome for Module 3 (Bedrock AgentCore Advanced).
