
# 🔷 PART 1 — LAUNCH WINDOWS EC2

## 1. Launch Instance

* AMI: `Windows Server 2022 Base`
* Instance type: `t2.micro / t3.micro`
* Key pair: create/download `.pem`
* Network:

  * Public subnet
  * Auto-assign public IP = **Enable**

---

## 2. Security Group

| Type         | Port | Source    |
| ------------ | ---- | --------- |
| RDP          | 3389 | YOUR IP   |
| All outbound | ALL  | 0.0.0.0/0 |

---

## 3. Attach IAM Role

Attach role with policies:

```
AmazonSSMManagedInstanceCore
CloudWatchAgentServerPolicy
amazonec2rolessm
CloudWatchAgentAdminPolicy
 
```

---

## 4. Connect via RDP

* Get password using `.pem`
* Connect using Remote Desktop

---

# 🔷 PART 2 — INSTALL CLOUDWATCH AGENT

---

## 1. Open PowerShell as Administrator

---

## 2. Create temp directory

```
mkdir C:\Temp
```

---

## 3. Download Agent

```
Invoke-WebRequest https://s3.amazonaws.com/amazoncloudwatch-agent/windows/amd64/latest/amazon-cloudwatch-agent.msi -OutFile C:\Temp\amazon-cloudwatch-agent.msi
```

---

## 4. Verify file

```
dir C:\Temp\amazon-cloudwatch-agent.msi
```

👉 File size should be ~60MB+

---

## 5. Install agent

```
msiexec /i C:\Temp\amazon-cloudwatch-agent.msi
```

---

## 6. Verify service exists

```
Get-Service AmazonCloudWatchAgent
```

Expected:

```
Stopped
```

---

# 🔷 PART 3 — CONFIGURE AGENT

---

## 1. Go to install directory

```
cd "C:\Program Files\Amazon\AmazonCloudWatchAgent"
```

---

## 2. Run config wizard

```
.\amazon-cloudwatch-agent-config-wizard.exe
```

---

# 🔷 Wizard Answers (IMPORTANT)

```
OS → 2 (windows)
```

```
EC2 or On-Prem → 1 (EC2)
```

```
StatsD → 2 (no)
```

```
Existing config → 2 (no)
```

```
Monitor host metrics → 1 (yes)
```

```
CPU per core → 2 (no)
```

```
Add EC2 dimensions → 1 (yes)
```

```
Aggregate dimensions → 1 (yes)
```

```
Resolution → 4 (60s)
```

```
Metrics config → 2 (standard)
```

```
Satisfied config → 1 (yes)
```

```
Custom log files → 2 (no)
```

```
Windows event logs → 2 (no)
```

```
X-Ray → 2 (no)
```

```
Store in SSM → 2 (no)
```

---

## 3. Verify config file created

```
dir "C:\Program Files\Amazon\AmazonCloudWatchAgent\config.json"
```

---

## 4. Move config to correct path

```
move "C:\Program Files\Amazon\AmazonCloudWatchAgent\config.json" "C:\ProgramData\Amazon\AmazonCloudWatchAgent\config.json"
```

---

# 🔷 PART 4 — START CLOUDWATCH AGENT

---

## 1. Start agent with config

```
& "C:\Program Files\Amazon\AmazonCloudWatchAgent\amazon-cloudwatch-agent-ctl.ps1" -a fetch-config -m ec2 -c file:"C:\ProgramData\Amazon\AmazonCloudWatchAgent\config.json" -s
```

---

## 2. Verify service is running

```
Get-Service AmazonCloudWatchAgent
```

Expected:

```
Running
```

---

# 🔷 PART 5 — VERIFY METRICS IN AWS

---

## 1. Go to AWS Console

```
CloudWatch → Metrics → All metrics → CWAgent
```

---

## 2. Check metrics

You should see:

* Memory (`% Committed Bytes In Use`)
* Disk (`% Free Space`)
* CPU (`Processor metrics`)

---

## 3. If metrics not visible

Check logs:

```
Get-Content "C:\ProgramData\Amazon\AmazonCloudWatchAgent\Logs\amazon-cloudwatch-agent.log" -Tail 50
```

---

## 4. Check region

Ensure you are in correct AWS region (same as EC2)

---

# 🔷 PART 6 — (OPTIONAL BUT IMPORTANT) CLOUDWATCH ALARM

---

## 1. Create SNS Topic

AWS Console:

```
SNS → Topics → Create topic
```

Type:

```
Standard
```

---

## 2. Create subscription

Protocol:

```
Email
```

Confirm email

---

## 3. Create Alarm

```
CloudWatch → Alarms → Create alarm
```

---

## 4. Select metric

```
EC2 → CPUUtilization
```

---

## 5. Condition

```
Greater than 70%
Period: 5 minutes
```

---

## 6. Action

Select SNS topic

---

## 7. Create alarm

---

# 🔷 PART 7 — TROUBLESHOOTING

---

## Check service

```
Get-Service AmazonCloudWatchAgent
```

---

## Restart agent

```
Restart-Service AmazonCloudWatchAgent
```

---

## Check logs

```
notepad C:\ProgramData\Amazon\AmazonCloudWatchAgent\Logs\amazon-cloudwatch-agent.log
```

---

## Check config

```
notepad C:\ProgramData\Amazon\AmazonCloudWatchAgent\config.json
```

---



That’s where your project becomes **interview-ready**.
