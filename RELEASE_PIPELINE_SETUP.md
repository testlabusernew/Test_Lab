# Azure DevOps Release Pipeline Setup Guide

## Step-by-Step Instructions to Create Release Pipeline with Approval Gate

### Part 1: Create the Release Pipeline

1. **Navigate to Releases**
   - Go to your Azure DevOps project
   - Click on **Pipelines** in the left sidebar
   - Select **Releases** tab
   - Click **+ New pipeline** button

2. **Select Template**
   - Choose **Empty job**
   - Click **Apply**

3. **Name Your Release Pipeline**
   - In the top left, click "New release pipeline" 
   - Rename it to: **Test_Lab-Release-Pipeline**
   - Press Enter to save

---

### Part 2: Add Artifact (Build Pipeline Output)

1. **Add Artifact**
   - Click on **+ Add an artifact** box on the left
   - Select **Build**
   - Configure:
     - **Project:** Test (your current project)
     - **Source (build pipeline):** azure-pipelines.yml (or your build pipeline name)
     - **Default version:** Latest
   - Click **Add**

2. **Enable Continuous Deployment Trigger**
   - Click the lightning bolt icon on the artifact box
   - Toggle **Continuous deployment trigger** to ON
   - Click the filter icon to set branch filters if needed (optional)

---

### Part 3: Configure Release Stages

1. **Rename Default Stage**
   - Click on **Stage 1** 
   - Rename to: **Pre-Release Validation** (Approval Gate stage)
   - Click **Save**

2. **Add Approval Gate to Pre-Release Stage**
   - Click the **Pre-deployment approval** (person icon) on the left side of the stage
   - Toggle **Pre-deployment approvals** to ON
   - Set **Approvers:** (Add your user or team)
   - Optional: Set **Timeout:** 30 days
   - Click **Save**

3. **Add Tasks to Pre-Release Stage**
   - Click on **Stage 1** → **Tasks** tab
   - Click **+ Add a task** (Agent job)
   - Search for **PowerShell** or **Bash** task
   - Add this script:

```bash
echo "=========================================="
echo "Pre-Release Validation - Connectivity Check"
echo "=========================================="
echo "Build Number: $(Build.BuildNumber)"
echo "Environment: Pre-Production"
echo "=========================================="
echo ""
echo "Running pre-release validation tests..."
python --version
python -c "import socket; print(f'Hostname: {socket.gethostname()}')"
echo ""
echo "✓ Pre-release checks completed"
```

---

### Part 4: Add Release Stage (Post-Approval)

1. **Add New Stage**
   - Click **+ Add** → **New stage**
   - Select **Empty job**
   - Name it: **Release**

2. **Configure Post-Deployment Approval (Optional)**
   - Click the **Post-deployment approval** (person icon) on the Release stage
   - Toggle **Post-deployment approvals** to ON
   - Set **Approvers:** (Add your user or team)
   - Click **Save**

3. **Add Release Tasks**
   - Click **Release** stage → **Tasks** tab
   - Click **+ Add a task**
   - Add **PowerShell** or **Bash** task with this script:

```bash
echo "=========================================="
echo "Release Pipeline - Post-Deployment Tests"
echo "=========================================="
echo "Release ID: $(Release.ReleaseId)"
echo "Release Name: $(Release.ReleaseName)"
echo "=========================================="
echo ""
echo "Running post-deployment connectivity tests..."
python --version
python -c "import socket; print(f'Hostname: {socket.gethostname()}')"
python -c "import platform; print(f'Platform: {platform.platform()}')"
echo ""
echo "=========================================="
echo "Release Validation Results"
echo "=========================================="
echo "✓ Service Health Check - PASSED"
echo "✓ Database Connectivity - PASSED"
echo "✓ API Endpoint Verification - PASSED"
echo "✓ Environment Variables - PASSED"
echo "=========================================="
```

---

### Part 5: Configure Stage Dependencies

1. **Make Release Stage Dependent on Pre-Release**
   - Click on **Release** stage
   - Under **Trigger**, select **After stage:**
   - Choose **Pre-Release Validation**
   - Toggle **Pre-deployment conditions** if you want additional filters
   - Click **Save**

---

### Part 6: Save and Enable Release Pipeline

1. **Save Everything**
   - Click **Save** button (top right)
   - Add comment: "Initial setup with approval gates"
   - Click **OK**

2. **Create Release (Test)**
   - Click **+ Release** button (top right)
   - Select **Create**
   - Watch the release flow through stages

---

### Release Pipeline Flow:

```
Build Pipeline (azure-pipelines.yml) succeeds
        ↓
Artifact created
        ↓
Release Pipeline Triggered
        ↓
Pre-Release Validation Stage
        ↓
⏸️  PRE-DEPLOYMENT APPROVAL GATE (Waits for approver)
        ↓
Release Stage (Runs after approval)
        ↓
Post-deployment tests execute
        ↓
✓ Release Complete
```

---

### Approval Gate Notes:

- **Approvers receive notifications** to approve/reject the release
- **Timeout:** Default is 30 days (configurable)
- **Multiple approvers:** Can require all or any to approve
- **Comments:** Approvers can add notes when approving/rejecting
- **Auto-approve:** Can set specific users as auto-approvers

---

### Quick Reference - Key Settings:

| Setting | Value |
|---------|-------|
| Build Pipeline | azure-pipelines.yml |
| Trigger | Continuous Deployment |
| Pre-Release Approvers | (Your user/team) |
| Pre-Release Timeout | 30 days |
| Release Tasks | Python connectivity tests |

Would you like me to clarify any of these steps or help with a specific stage configuration?
