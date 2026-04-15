#### S3 — Cloud Storage (like Google Drive for your app
```
Stores files as objects in buckets
Has storage classes — Standard (hot data), Glacier (cold/archive data)
Versioning = keeps old copies, protects against accidents
Lifecycle policy = auto-moves or deletes files after X days to save money
Pre-signed URL = temporary link to share a private file
Security: block public access, use encryption (SSE-S3/KMS), use IAM roles
```

#### ⚖️ ELB — Traffic Distributor (Load Balancer)

```
Spreads traffic across multiple servers so none gets overwhelmed
Types: ALB (web apps, URL-based routing), NLB (ultra-fast, TCP), CLB (legacy)
Health checks = automatically stops sending traffic to broken servers
Sticky sessions = same user always hits same server
SSL termination = load balancer handles HTTPS, talks plain HTTP to your servers
```


####  IAM — Who Can Do What in AWS
```
Controls users, groups, and roles
Roles = temporary access (no password/keys) — preferred for EC2, Lambda
Policy = JSON document saying "Allow/Deny this action on this resource"
Least privilege = give only the permissions actually needed
MFA = two-factor authentication for extra security

```

#### VPC — Your Private Network in AWS
```
Isolated network where your AWS resources live
Public subnet = has internet access (web servers go here)
Private subnet = no direct internet (databases go here)
NAT Gateway = lets private servers download updates without being exposed
VPC Peering = connect two VPCs like they're on the same network
Security Groups (stateful, instance-level) vs Network ACLs (stateless, subnet-level)
```

#### RDS — Managed Relational Database (MySQL, PostgreSQL, etc.)
```
AWS handles backups, patches, failover — you just use the DB
Multi-AZ = standby copy in another zone, auto-failover if primary dies
Read Replica = extra read-only copies to handle heavy read traffic
Aurora = AWS's own super-fast DB, MySQL/Postgres compatible, auto-replicates across 3 zones
```

#### Lambda — Run Code Without Servers
```
Upload code → AWS runs it when triggered (pay per millisecond)
Triggers: S3 upload, API call, DynamoDB change, scheduled timer, etc.
Cold start = first run is slow (~100ms–1s), reuses container after that
Layers = shared libraries you attach to multiple functions
```

#### CloudWatch — Monitoring & Alerts
```
Watches your AWS services — metrics, logs, events
Alarms = alert you (or auto-scale) when a metric crosses a threshold
Logs = collect and search log files from Lambda, EC2, etc.
CloudWatch vs CloudTrail: CloudWatch = "what's happening now" | CloudTrail = "who did what" (audit trail)
```

#### DynamoDB — Fast NoSQL Database

```
Serverless, key-value database with millisecond speed at any scale
Partition key = unique ID for each item (like a row ID)
Sort key = secondary key to group/sort related items
On-demand = auto-scales, pay per request | Provisioned = cheaper if traffic is predictable
Streams = captures every change to trigger Lambda or replicate data
```

#### SNS & SQS — Messaging Between Services
```
SNS = pub/sub: one message → sent to ALL subscribers at once (fan-out)
SQS = queue: messages wait until a consumer processes them (decoupling)
FIFO queue = guaranteed order + no duplicates (slower)
Standard queue = super fast, may have duplicates or out-of-order
Dead Letter Queue (DLQ) = holds failed messages for debugging

```
