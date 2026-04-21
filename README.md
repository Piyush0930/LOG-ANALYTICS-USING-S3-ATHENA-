# 📊 Log Analytics Pipeline using AWS S3 + Athena (POC)

---

## 🚀 Overview

This project demonstrates a **simple, scalable log analytics solution** using AWS services to eliminate manual log downloads and enable:

* 📅 Day-wise log retrieval
* 📆 Monthly log aggregation
* 📥 Single-click CSV export
* ⚡ Fast querying using SQL

---

## 🧱 Architecture

```
Application Logs → Amazon S3 → Amazon Athena → Query → CSV Output
```

---

## 🎯 Problem Statement

Current process:

* Logs are downloaded manually **day-by-day**
* ~500 logs/day → time-consuming and inefficient

Solution:

* Store logs in structured format in S3
* Query using Athena
* Generate reports in seconds

---

## 📂 S3 Bucket Structure

```
s3://poc-logs-bucket/
│
├── logs/
│   ├── year=2026/
│   │   ├── month=02/
│   │   │   ├── day=02/
│   │   │   │   ├── log1.log
│   │   │   │   ├── log2.log
│   │   │   ├── day=03/
│   │   │       ├── log3.log
│
└── athena-results/
```

---

## 🔐 IAM Permissions

Attach the following policy to your IAM user/role:

```json
{
  "Effect": "Allow",
  "Action": ["s3:GetObject", "s3:ListBucket"],
  "Resource": [
    "arn:aws:s3:::poc-logs-bucket",
    "arn:aws:s3:::poc-logs-bucket/*"
  ]
}
```

---

## ⚙️ Athena Setup

### 1. Set Query Result Location

```
s3://poc-logs-bucket/athena-results/
```

---

## 🗄️ Create Table in Athena

```sql
CREATE EXTERNAL TABLE logs (
  type string,
  time string,
  elb string,
  client_ip_port string,
  target_ip_port string,
  request_processing_time double,
  target_processing_time double,
  response_processing_time double,
  elb_status_code int,
  target_status_code int,
  received_bytes bigint,
  sent_bytes bigint,
  request string,
  user_agent string,
  ssl_cipher string,
  ssl_protocol string
)
PARTITIONED BY (year string, month string, day string)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
WITH SERDEPROPERTIES (
  'input.regex' = '^(\\S+) (\\S+) (\\S+) (\\S+) (\\S+) (\\S+) (\\S+) (\\S+) (\\S+) (\\S+) (\\S+) (\\S+) "([^"]*)" "([^"]*)" (\\S+) (\\S+).*'
)
LOCATION 's3://poc-logs-bucket/logs/';
```

---

## 🔄 Load Partitions

```sql
MSCK REPAIR TABLE logs;
```

> Required only when **new day folder is added**

---

## ✅ Sample Queries

### 🔹 Fetch Day-wise Logs

```sql
SELECT *
FROM logs
WHERE year='2026' AND month='02' AND day='02';
```

---

### 🔹 Fetch Monthly Logs

```sql
SELECT *
FROM logs
WHERE year='2026' AND month='02';
```

---

### 🔹 Count Logs

```sql
SELECT COUNT(*)
FROM logs
WHERE year='2026' AND month='02';
```

---

### 🔹 Top Client IPs

```sql
SELECT client_ip_port, COUNT(*) AS hits
FROM logs
GROUP BY client_ip_port
ORDER BY hits DESC
LIMIT 10;
```

---

## 📥 Output

Query results are automatically stored in:

```
s3://poc-logs-bucket/athena-results/
```

* Format: CSV
* Ready to download and share with client

---

## 🔁 Workflow

| Step | Action                                      |
| ---- | ------------------------------------------- |
| 1    | Upload logs to S3                           |
| 2    | Run `MSCK REPAIR` (only for new partitions) |
| 3    | Run Athena query                            |
| 4    | Download CSV                                |

---

## 🚨 Common Mistakes

* ❌ Wrong folder structure (no partitions)
* ❌ Not setting Athena result location
* ❌ Missing IAM permissions
* ❌ Using Glue crawler unnecessarily
* ❌ Uploading logs randomly

---

## ⚡ Future Improvements

* Automate using AWS Lambda
* Schedule queries using EventBridge
* Convert logs to Parquet (cost optimization)
* Build dashboards (Power BI / Grafana)

---

## 🧠 Key Learnings

* Athena queries data directly from S3
* Proper folder structure = everything
* Partitioning improves performance
* No need for database servers

---

## 🎯 Conclusion

This setup replaces:

> Manual log downloads ❌

With:

> Automated, query-based log extraction ✅

---

## 👨‍💻 Author

Piyush Patil
(Log Analytics POC – AWS S3 + Athena)

---
