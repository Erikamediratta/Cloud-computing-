## 6. Monitor Resources Using AWS CloudWatch
**Objective:** Use CloudWatch to monitor AWS resources.

**a)** Set up CloudWatch metrics for an EC2 instance (e.g., CPU utilization).

**b)** Create a CloudWatch Alarm to send notifications when a threshold is exceeded.

**c)** Configure an SNS topic for email notifications.

**d)** Test the setup by simulating high CPU usage.

<img width="1053" height="66" alt="image" src="https://github.com/user-attachments/assets/d2d4ed6a-1de1-4167-b69b-885ce479456e" />


<img width="1277" height="292" alt="image" src="https://github.com/user-attachments/assets/10cac100-88eb-451e-be79-3333ec4d6595" />


```

**Create a Custom Dashboard:**
1. Go to **CloudWatch → Dashboards → Create dashboard**
2. Dashboard name: `EC2-Monitoring-Dashboard`
3. Click **Create dashboard**
4. Add widget → **Line** → **Metrics**
5. Select: `AWS/EC2 → Per-Instance Metrics → CPUUtilization`
6. Select your instance → **Create widget**
7. Add another widget for `NetworkIn`, `NetworkOut`
8. Save dashboard

---
```

<img width="1134" height="609" alt="image" src="https://github.com/user-attachments/assets/3fc9d651-dc43-47ad-a77d-e2b820671749" />

### Step B — Create a CloudWatch Alarm

1. Go to **CloudWatch → Alarms → All alarms → Create alarm**

**Step 1 — Select metric:**
```
Select metric → EC2 → Per-Instance Metrics → CPUUtilization
Select your instance → Select metric

Statistic:   Average
Period:      5 minutes
```

**Step 2 — Specify conditions:**
```
Threshold type:              Static
Condition:                   Greater/Equal
Than:                        70    (70%)

Additional configuration:
  Datapoints to alarm:       2 out of 3
  (This means 2 consecutive data points must exceed threshold)
  
  Missing data treatment:    Treat missing data as missing
```

**Step 3 — Configure actions:**
```
Alarm state trigger:         In alarm

Send notification to:        Create new SNS topic →
  Create a new topic
  Topic name:                ec2-cpu-alerts
  Email endpoints:           your@email.com
→ Create topic
```

**Step 4 — Name and description:**
```
Alarm name:        EC2-High-CPU-Alarm
Description:       Alert when EC2 CPU exceeds 70% for 10 minutes
```

**Step 5 — Preview and create → Create alarm**

> 📧 **Check your email** — you'll receive a subscription confirmation from AWS SNS. You MUST click **Confirm subscription** or you won't receive alerts!

---


<img width="956" height="284" alt="image" src="https://github.com/user-attachments/assets/5d4229c3-a6aa-40a0-9e03-48a74687febe" />


### Step C — Configure SNS Topic for Email Notifications

Let's explore SNS in more detail.

#### View the SNS Topic
```
AWS Console → SNS → Topics → ec2-cpu-alerts
→ View subscriptions, ARN, access policy
```

#### Add Additional Subscribers
```
SNS → Topics → ec2-cpu-alerts → Create subscription
  Protocol: Email
  Endpoint: another@email.com
→ Create subscription
(The new subscriber must confirm via email)
```

#### Test SNS Directly
```
SNS → Topics → ec2-cpu-alerts → Publish message
  Subject:   Test Notification
  Message:   This is a test message from AWS SNS
→ Publish message
```

You should receive the test email immediately.

#### SNS Topic Policy (Access Control)
```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudwatch.amazonaws.com"
      },
      "Action": "SNS:Publish",
      "Resource": "arn:aws:sns:us-east-1:ACCOUNT_ID:ec2-cpu-alerts",
      "Condition": {
        "StringEquals": {
          "AWS:SourceAccount": "ACCOUNT_ID"
        }
      }
    }
  ]
}
```

---

<img width="935" height="433" alt="image" src="https://github.com/user-attachments/assets/5ebf5e5d-7445-43af-b35b-538001732abb" />


### Step D — Test by Simulating High CPU Usage

#### SSH into your EC2 instance
```bash
ssh -i my-ec2-keypair.pem ec2-user@<public-ip>
```

#### Method 1 — Using `stress`
```bash
# Install stress utility
sudo yum install -y stress      # Amazon Linux
# or
sudo apt install -y stress      # Ubuntu

# Stress all CPUs for 15 minutes
sudo stress --cpu $(nproc) --timeout 900

# Check CPU usage in real time
top
# Press 'q' to quit top
```

#### Method 2 — Using `dd` (disk + CPU)
```bash
# This stresses both CPU and I/O
dd if=/dev/zero of=/dev/null bs=1M count=100000
```

#### Method 3 — Using a Bash Loop
```bash
# Pure CPU stress with bash
while true; do :; done &
while true; do :; done &
while true; do :; done &
while true; do :; done

# To stop:
kill $(jobs -p)
```

<img width="950" height="415" alt="image" src="https://github.com/user-attachments/assets/8d48f742-ba39-429d-bc79-d60c7176fc9f" />

