
# Cloud Security Posture Audit using Prowler on GCP
<img width="1063" height="538" alt="Screenshot 2026-09-19 at 14 20 04" src="https://github.com/user-attachments/assets/cd3c2fa2-9f27-4dba-b68a-050e8d5957b4" />


## Project Overview
It is a step-by-step guide to auditing a Google Cloud environment against the CIS Google Cloud Platform Foundations Benchmark using Prowler. This includes from setup, to scan, to remediation, to verification.

## What I did
- Set up a GCP project with the the right permission for auditing.
- Created a storage bucket and made it public by adding the **allUsers** principal, assigned the role **Storage Object Viewer** and a service account with overprivileged permissions, granting it an **Editor** basic role at the project level.
<img width="1321" height="792" alt="Screenshot 2026-09-19 at 13 57 32" src="https://github.com/user-attachments/assets/6992f7ad-88e8-4509-9c6b-5a5db507dbc4" />

<img width="748" height="738" alt="Screenshot 2026-09-19 at 14 05 03" src="https://github.com/user-attachments/assets/71b5d491-c12c-4e9a-8fa6-97b74dd96d30" />

- Installed and authenticated Prowler
- Ran a full security scan against the CIS GCP benchmark
- Read and prioritize the findings
- Remediate real misconfigurations
- Re-scan to verify the fix

### Step 1: Enabled the required APIs
Prowler needs these APIs enabled on the target project to enumerate resources and check configuration:
```
gcloud services enable \
  compute.googleapis.com \
  storage.googleapis.com \
  iam.googleapis.com \
  cloudresourcemanager.googleapis.com \
  cloudasset.googleapis.com \
  serviceusage.googleapis.com
```

### Step 2: Grant the Auditing Identity the Right Permissions
Decide whether you'll scan as your own user account or a dedicated service account (recommended for anything beyond a one-off test). Either way, grant it:
```
# Replace IDENTITY with your email or a service account address,
# and YOUR_PROJECT_ID with your project ID
gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
  --member="user:IDENTITY" \
  --role="roles/viewer"

gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
  --member="user:IDENTITY" \
  --role="roles/serviceusage.serviceUsageConsumer"
```
Without these, Prowler runs but returns permission-denied errors on most checks rather than a clear auth failure
### Step 3: Authenticate
```
gcloud auth application-default login
gcloud config set project YOUR_PROJECT_ID
gcloud auth application-default set-quota-project YOUR_PROJECT_ID
```
### Step 4: Install Prowler
Install into a virtual environment
```
python3 -m venv prowler-env
source prowler-env/bin/activate
pip install prowler
```
Confirm it installed correctly:
```
prowler --version
```
### Step 5: Run Your First Scan
```
prowler gcp --project-ids YOUR_PROJECT_ID
```
This runs Prowler's full default GCP check set.

### Step 6: Review the Report
Prowler writes output to an output/ folder in table, CSV, JSON, and HTML formats. 
Open the HTML report first as it's the most readable. 
You can run it by running:
```
python3 "the file"
```
For each finding, note:

Severity: how Prowler ranked it
Resource: exactly what's affected
Real-world risk: what an attacker could actually do with it (don't just triage by severity label alone)

### Step 7: Remediate

Filter and work through findings one at a time. The most common ones you'll see on a fresh project, and their fixes:
| Finding | Typical Fix | 
| -------- | -------- | 
| Public Cloud Storage bucket | Remove allUsers/allAuthenticatedUsers IAM bindings (Enable Public Access Prevention )| 
| Overly broad IAM role (e.g. Editor/Owner) | Replace with a least-privilege custom role. Recommendations for a suggested scoped role |

### Step 8: Re-scan and Verify
```
prowler gcp --project-ids YOUR_PROJECT_ID
```
Compare this report against the first report. There will be the drop in finding count and rise in compliance percentage is your evidence that the remediation actually worked, not just that you made a change.
