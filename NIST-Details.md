
## NIST CSF Functions and How They Apply

### 1. Identify (ID)
**Relevant Activities:**
- **Resource Inventory**: The template defines and names all necessary AWS resources (S3 buckets, Lambda function, IAM role, SSM Parameter). While not a full-blown asset inventory, having explicit CloudFormation definitions clarifies which assets exist and their configurations (ID.AM-1, ID.AM-2).
- **Roles and Responsibilities**: The IAM Role for the Lambda function clearly delineates who (or what service) can access specific AWS resources (ID.AM-6, ID.GV-2). This approach helps ensure the right controls are in place from the start.

### 2. Protect (PR)
**Relevant Activities:**
- **Access Control**: 
  - **IAM Role with Least Privilege**: The `LambdaExecutionRole` is restricted to only the actions needed (e.g., `s3:PutObject`, `s3:GetObject` for the stock-data bucket, and limited write permissions for the logs bucket). This directly aligns with PR.AC-4 (access permissions enforce least privilege).
- **Data Security**:
  - **SSM Parameter Store for API Keys**: API keys are stored in an encrypted parameter (`StockApiKey`) rather than being hardcoded or placed in environment variables. This helps ensure data-at-rest is protected (PR.DS-1) and further restricts access to the parameter (PR.DS-6).
  - **S3 Bucket Lifecycle**: The main S3 bucket has a lifecycle rule that automatically deletes objects after 30 days, which helps mitigate risks associated with indefinite data retention (PR.IP-6). 
- **Protective Technology**:
  - **Scoped Bucket Policies**: By limiting the Lambda function’s ability to write only to the necessary S3 buckets, you are enforcing the principle of least functionality (PR.PT-3).

### 3. Detect (DE)
Although this stack does not implement full intrusion detection, it does include:
- **Logging**: The Lambda function writes logs to a dedicated S3 bucket (`magnificent7-logs`). Storing logs separately can help facilitate future anomaly detection or threat hunting (DE.CM-1).  
- **Monitoring Hooks**: The scheduled CloudWatch Event triggers the Lambda on a recurring basis, which can be extended later to include alerting or detection logic if desired (DE.DP-4).

### 4. Respond (RS)
- **Incident Response Readiness**:  
  - While the template does not define a full response plan, the practice of storing logs separately and using a well-defined IAM role positions the solution for faster isolation and investigation if a breach occurs (RS.CO-2, RS.AN-1).

### 5. Recover (RC)
- **Recovery Planning**:
  - Because everything is deployed via Infrastructure as Code (IaC), you can quickly redeploy the environment in another region or account if needed (RC.RP-1). 
  - Storing logs (and stock-data snapshots) in S3 also facilitates post-incident review and data restoration (RC.CO-1, RC.IM-2).

---

## Key Takeaways

- **IAM Roles with Scoped Permissions** (Protect -> PR.AC)  
  Ensures that the Lambda function only has the exact permissions needed to interact with S3 and retrieve the API key from SSM Parameter Store.  

- **Secure Storage of Secrets** (Protect -> PR.DS)  
  The API key is placed in AWS SSM Parameter Store with encryption at rest, preventing it from being visible in the code or environment variables.  

- **Lifecycle Policies** (Protect -> PR.IP)  
  Automatically removing objects after 30 days reduces the data footprint and helps comply with data-retention policies.  

- **Logging and Separation of Duties** (Detect -> DE.CM)  
  Keeping logs in a dedicated S3 bucket ensures that operational (stock data) writes remain distinct from security/event logs, setting the foundation for eventual monitoring and detection.  

- **Infrastructure as Code** (Recover -> RC.RP)  
  Using CloudFormation means a known-good environment can be re-created quickly if a security event necessitates a rebuild or rollback.
