# NIST CSF Functions and How They Apply

**Implementing robust security in cloud environments benefits greatly from a structured approach like NIST CSF v1.1, which helps align AWS resources and practices with recognized security posture frameworks. 
Below is an overview of how this solution maps to the NIST CSF Functions and Subcategories.**

---


### 1. Identify (ID)

**Relevant Activities:**
- **Resource Inventory**  
  - The template defines all AWS resources (S3 buckets, Lambda, IAM role, SSM Parameter) in code. While not a full-blown asset inventory, using CloudFormation clarifies which assets exist and their configurations.  
  - **(ID.AM-1, ID.AM-2)**  

- **Roles and Responsibilities**  
  - The **IAM Role** for the Lambda function clearly delineates what service can access specific AWS resources.  
  - **(ID.AM-6, ID.GV-2)**  
  - Ensures controls are in place from the start and helps identify who is responsible for what.

---

### 2. Protect (PR)

**Relevant Activities:**

- **Access Control**  
  - **IAM Role with Least Privilege**:  
    The `LambdaExecutionRole` is restricted to only the actions needed (e.g., `s3:PutObject`, `s3:GetObject` for the stock-data bucket, and limited write to the logs bucket).  
    - **(PR.AC-4)**: Enforces least privilege.

- **Data Security**  
  - **SSM Parameter Store for API Keys**:  
    The API key is stored in an encrypted parameter rather than hardcoded, ensuring data-at-rest is protected and access is restricted.  
    - **(PR.DS-1, PR.DS-6)**  

  - **S3 Bucket Lifecycle**:  
    The main S3 bucket has a lifecycle rule that **deletes objects after 30 days**, helping reduce unnecessary data retention.  
    - **(PR.IP-6)**

- **Protective Technology**  
  - **Scoped Bucket Policies**:  
    Limiting the Lambda function’s write actions to only necessary S3 buckets enforces the principle of least functionality.  
    - **(PR.PT-3)**

---

### 3. Detect (DE)

While this template doesn’t include a full intrusion detection solution, it does incorporate:

- **Logging**  
  - The Lambda function writes logs to a dedicated S3 bucket (`magnificent7-logs`). This separation facilitates future anomaly detection or threat hunting.  
  - **(DE.CM-1)**

- **Monitoring Hooks**  
  - A CloudWatch Event rule triggers the Lambda function on a regular schedule; this can be extended to include **alerting or detection** logic for anomalies.  
  - **(DE.DP-4)**

---

### 4. Respond (RS)

- **Incident Response Readiness**  
  - Although the stack does not define a full response plan, the practice of storing logs separately and using scoped IAM roles supports **faster isolation and investigation** if a breach occurs.  
  - **(RS.CO-2, RS.AN-1)**

---

### 5. Recover (RC)

- **Recovery Planning**  
  - Deployed entirely via **Infrastructure as Code (CloudFormation)**, so a known-good environment can be re-created rapidly in another region or account if needed.  
  - **(RC.RP-1)**
  - Storing logs and stock-data snapshots in S3 helps with **post-incident review** and data restoration.  
  - **(RC.CO-1, RC.IM-2)**

---

## Key Takeaways

1. **IAM Roles with Scoped Permissions** *(Protect → PR.AC)*  
   - Ensures the Lambda function has only the necessary actions for S3 and SSM Parameter Store.

2. **Secure Storage of Secrets** *(Protect → PR.DS)*  
   - API key in **AWS SSM Parameter Store** with encryption at rest, removing the need to store secrets in code or environment variables.

3. **Lifecycle Policies** *(Protect → PR.IP)*  
   - Automatically removing objects after 30 days in the data bucket reduces storage costs, risk exposure, and compliance overhead.

4. **Logging and Separation of Duties** *(Detect → DE.CM)*  
   - Keeping logs in a dedicated bucket sets the foundation for future monitoring, detection, and effective audits.

5. **Infrastructure as Code** *(Recover → RC.RP)*  
   - Rapid environment rebuilds or rollbacks are possible if a security event or misconfiguration occurs.

For more details, consult the official **NIST CSF v1.1** documentation and your organization’s specific security policies.
