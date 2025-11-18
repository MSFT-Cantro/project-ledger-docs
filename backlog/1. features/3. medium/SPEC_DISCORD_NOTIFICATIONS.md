# Discord Notifications for Documentation and Releases

**Date:** November 18, 2025  
**Version:** 1.0  
**Status:** Ready for Implementation  
**Priority:** Medium - Improves Team Communication and Transparency  
**Location:** `docs/backlog/1. features/3. medium/`  
**Completed:** N/A

---

## 📊 Implementation Progress Summary

### Overall Completion: 50% Complete (2/4 items)

```
Documentation Notifications  ████████████████████ 100% 🟢
Release Notifications        ░░░░░░░░░░░░░░░░░░░░ 0% 🔴
GitHub Workflow Integration  ████████████████████ 100% 🟢
Testing & Documentation      ░░░░░░░░░░░░░░░░░░░░ 0% 🔴
```

### 🎯 Current Status
**Phase:** Implementation - Documentation Notifications Complete  
**Started:** November 18, 2025  
**Status:** GitHub workflow implemented and ready for testing. Release notifications pending.

### Quick Reference
- **Documentation Webhook**: 🟢 Complete and Configured
- **Release Webhook**: 🔴 Not Started
- **GitHub Actions**: 🟢 Complete
- **Testing**: 🟡 In Progress

### Progress Indicators

- 🔴 Not Started
- 🟡 In Progress
- 🟢 Complete
- ⚠️ Blocked
- ⏸️ Paused

---

## 📋 Overview

This specification outlines the implementation of Discord webhooks to automatically notify the team when:

1. **Documentation Changes**: Documentation files in the `project-ledger-docs` repository are updated and merged to the `main` branch
2. **Production Releases**: New versions of the application are deployed to Azure production environment

**Why This Is Important:**
- Improves team visibility into project documentation changes
- Creates transparency around release deployments
- Centralizes notifications in team's primary communication platform (Discord)
- Provides real-time updates without requiring manual announcements
- Creates searchable history of changes and deployments

**Key Benefits:**
- Instant notifications when specifications or documentation are updated
- Automatic alerts when production deployments occur
- Reduced manual communication overhead
- Better team coordination across time zones
- Audit trail of changes visible to all team members

**Scope:**
- GitHub Actions workflows for triggering notifications
- Discord webhook integration for both repositories
- Formatted, informative notification messages
- Deployment script integration for release notifications
- Configuration for different notification types

**Implementation Status:**
- GitHub Workflows: 🟢 Complete
- Discord Integration: 🟡 Configuration Pending
- Deployment Script Updates: 🔴 Not Started
- Testing: 🟡 Ready for Testing

---

## 🎯 Scope & Requirements

### Functional Requirements

| Requirement ID | Description | Priority | Status |
|----------------|-------------|----------|--------|
| FR-001 | Send Discord notification when docs are merged to main | High | 🟢 Complete |
| FR-002 | Send Discord notification when releases are deployed | High | 🔴 Not Started |
| FR-003 | Include commit details in documentation notifications | Medium | 🟢 Complete |
| FR-004 | Include version and changes in release notifications | High | 🔴 Not Started |
| FR-005 | Support different Discord channels for different notification types | Low | 🟢 Complete |
| FR-006 | Include links to changed files/commits in notifications | Medium | 🟢 Complete |
| FR-007 | Format notifications with Discord embeds for better readability | Medium | 🟢 Complete |

### Non-Functional Requirements

| Requirement ID | Description | Target | Status |
|----------------|-------------|--------|--------|
| NFR-001 | Notification delivery latency | < 30 seconds after trigger | 🔴 Not Started |
| NFR-002 | Notification reliability | 99.5% delivery success rate | 🔴 Not Started |
| NFR-003 | Security | Webhook URLs stored as secrets | 🔴 Not Started |
| NFR-004 | Configurability | Easy to add/change webhooks | 🔴 Not Started |

### Out of Scope

- Notifications for pull requests (not merges)
- Interactive Discord bot commands
- Two-way communication with Discord
- Notifications for other events (issues, comments, etc.)
- Integration with other communication platforms (Slack, Teams, etc.)
- Historical notification backfill

---

## 🏗️ Architecture & Design

### System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                  GitHub Repositories                         │
│                                                              │
│  ┌──────────────────────┐    ┌─────────────────────────┐   │
│  │ project-ledger-docs  │    │ project-ledger-app      │   │
│  │                      │    │                         │   │
│  │ [Merge to main] ────►│    │ [Deploy to production]─►│   │
│  └──────────────────────┘    └─────────────────────────┘   │
│           │                            │                     │
└───────────┼────────────────────────────┼────────────────────┘
            │                            │
            │ Trigger                    │ Trigger
            ▼                            ▼
┌─────────────────────┐      ┌──────────────────────────┐
│ GitHub Action       │      │ Deployment Script        │
│ (validate-docs.yml) │      │ (release-vX.Y.Z.sh)      │
│                     │      │                          │
│ 1. Detect merge     │      │ 1. Complete deployment   │
│ 2. Get commit info  │      │ 2. Extract version info  │
│ 3. Format message   │      │ 3. Format message        │
│ 4. Send webhook     │      │ 4. Send webhook          │
└─────────────────────┘      └──────────────────────────┘
            │                            │
            │ POST                       │ POST
            ▼                            ▼
┌─────────────────────────────────────────────────────────────┐
│                     Discord Server                           │
│                                                              │
│  ┌──────────────────────┐    ┌─────────────────────────┐   │
│  │ #documentation-updates│   │ #releases               │   │
│  │                      │    │                         │   │
│  │ 📝 Docs merged!      │    │ 🚀 v1.2.0 deployed!    │   │
│  │ - Commit: abc123     │    │ - Backend: ✅           │   │
│  │ - Author: John       │    │ - Frontend: ✅          │   │
│  │ - Files: 3 changed   │    │ - Changes: Bug fixes    │   │
│  └──────────────────────┘    └─────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Component Specifications

#### Component 1: Documentation Notification Workflow
**File/Location:** `.github/workflows/validate-docs.yml` (in project-ledger-docs repo)  
**Priority:** High  
**Dependencies:** Discord webhook URL, GitHub Actions

**Purpose:**
Sends Discord notifications when documentation is merged to the main branch, including details about what changed and who made the changes.

**Specification:**
```yaml
# Webhook notification job
notify-discord:
  name: Notify Discord
  runs-on: ubuntu-latest
  if: github.event_name == 'push' && github.ref == 'refs/heads/main'
  
  steps:
    - name: Send Discord Notification
      env:
        DISCORD_WEBHOOK: ${{ secrets.DISCORD_DOCS_WEBHOOK }}
      run: |
        # Send formatted webhook
```

**Features Required:**
- Trigger only on merge to main (not PRs)
- Extract commit author, message, and changed files
- Format as Discord embed with color coding
- Include links to commit and changed files
- Handle webhook failures gracefully

#### Component 2: Release Notification Integration
**File/Location:** `tools/deployment/release-vX.Y.Z.sh` (template)  
**Priority:** High  
**Dependencies:** Discord webhook URL, curl, successful deployment

**Purpose:**
Sends Discord notifications when a production release is successfully deployed, including version number, changes, and deployment status.

**Specification:**
```bash
# Discord notification function
send_discord_notification() {
  local version="$1"
  local changes="$2"
  local status="$3"  # success or failed
  
  # Format and send webhook
  curl -H "Content-Type: application/json" \
    -d '{...}' \
    "$DISCORD_RELEASE_WEBHOOK"
}
```

**Features Required:**
- Call after successful backend deployment
- Call after successful frontend deployment
- Include version number and revision suffix
- Include deployment changes/highlights
- Include deployment timestamp
- Handle webhook failures without stopping deployment

#### Component 3: Discord Webhook Configuration
**File/Location:** GitHub Secrets (both repos)  
**Priority:** Critical  
**Dependencies:** Discord server admin access

**Purpose:**
Securely store Discord webhook URLs as GitHub secrets for use in workflows and scripts.

**Configuration Required:**
- `DISCORD_DOCS_WEBHOOK` - For documentation updates (project-ledger-docs)
- `DISCORD_RELEASE_WEBHOOK` - For production releases (project-ledger-app)
- Optional: `DISCORD_ALERT_WEBHOOK` - For deployment failures/issues

---

## 🔧 Technical Implementation

### Backend Implementation

N/A - This feature uses GitHub Actions and shell scripts, no backend API required.

---

### GitHub Actions Implementation

#### Workflow: Documentation Notifications
**File:** `.github/workflows/validate-docs.yml`  
**Repository:** `project-ledger-docs`

**Enhanced Workflow:**
```yaml
name: Validate Documentation

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  validate:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      
    - name: Check markdown files
      uses: gaurav-nelson/github-action-markdown-link-check@v1
      with:
        use-quiet-mode: 'yes'
        config-file: '.github/markdown-link-check-config.json'

  # NEW: Discord notification on merge to main
  notify-discord:
    name: Notify Discord of Documentation Update
    runs-on: ubuntu-latest
    needs: validate  # Only run if validation passes
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      with:
        fetch-depth: 2  # Need previous commit for diff
    
    - name: Get Changed Files
      id: changed-files
      run: |
        # Get list of changed files
        CHANGED_FILES=$(git diff --name-only HEAD^ HEAD | head -10)
        NUM_FILES=$(git diff --name-only HEAD^ HEAD | wc -l)
        
        # Truncate list if too long
        if [ $NUM_FILES -gt 10 ]; then
          CHANGED_FILES="$CHANGED_FILES\n... and $((NUM_FILES - 10)) more files"
        fi
        
        # Export for next step (escape newlines)
        echo "files<<EOF" >> $GITHUB_OUTPUT
        echo "$CHANGED_FILES" >> $GITHUB_OUTPUT
        echo "EOF" >> $GITHUB_OUTPUT
        echo "count=$NUM_FILES" >> $GITHUB_OUTPUT
    
    - name: Send Discord Notification
      env:
        DISCORD_WEBHOOK: ${{ secrets.DISCORD_DOCS_WEBHOOK }}
      run: |
        # Extract commit info
        COMMIT_MSG=$(git log -1 --pretty=%B | head -1)
        COMMIT_AUTHOR=$(git log -1 --pretty=%an)
        COMMIT_SHA=$(git rev-parse --short HEAD)
        COMMIT_URL="https://github.com/${{ github.repository }}/commit/${{ github.sha }}"
        
        # Format changed files for Discord
        CHANGED_FILES="${{ steps.changed-files.outputs.files }}"
        FILE_COUNT="${{ steps.changed-files.outputs.count }}"
        
        # Determine emoji based on files changed
        if echo "$CHANGED_FILES" | grep -q "SPEC_"; then
          EMOJI="📋"
          TITLE="Specification Updated"
        elif echo "$CHANGED_FILES" | grep -q "deployment/"; then
          EMOJI="🚀"
          TITLE="Deployment Documentation Updated"
        elif echo "$CHANGED_FILES" | grep -q "guides/"; then
          EMOJI="📖"
          TITLE="Guide Updated"
        else
          EMOJI="📝"
          TITLE="Documentation Updated"
        fi
        
        # Create Discord embed
        curl -H "Content-Type: application/json" \
          -d '{
            "username": "Project Ledger Docs",
            "avatar_url": "https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png",
            "embeds": [{
              "title": "'"$EMOJI $TITLE"'",
              "description": "'"$COMMIT_MSG"'",
              "url": "'"$COMMIT_URL"'",
              "color": 5814783,
              "fields": [
                {
                  "name": "Author",
                  "value": "'"$COMMIT_AUTHOR"'",
                  "inline": true
                },
                {
                  "name": "Commit",
                  "value": "[`'"$COMMIT_SHA"'`]('"$COMMIT_URL"')",
                  "inline": true
                },
                {
                  "name": "Files Changed",
                  "value": "'"$FILE_COUNT"' file(s)",
                  "inline": true
                },
                {
                  "name": "Changed Files",
                  "value": "```\n'"${CHANGED_FILES}"'\n```",
                  "inline": false
                }
              ],
              "timestamp": "'$(date -u +%Y-%m-%dT%H:%M:%S.000Z)'"
            }]
          }' \
          "$DISCORD_WEBHOOK"
```

**Key Features:**
- Only triggers on successful merge to main (not PRs)
- Waits for validation to pass before sending
- Extracts commit metadata (author, message, SHA)
- Lists changed files (limited to 10 for readability)
- Smart emoji selection based on file types
- Rich Discord embed formatting
- Clickable links to commit
- Timestamp in UTC

---

### Deployment Script Integration

#### Enhanced Release Script
**File:** `tools/deployment/release-template.sh`  
**Repository:** `project-ledger-app`

**Discord Notification Function:**
```bash
#!/bin/bash

# ============================================================================
# CONFIGURATION - UPDATE THESE FOR EACH RELEASE
# ============================================================================

VERSION="X.Y.Z"
REVISION_SUFFIX="vX-Y-Z"
CHANGES_DESCRIPTION="
  ✅ [Feature/Fix 1]
  ✅ [Feature/Fix 2]
  ✅ [Feature/Fix 3]
"

# Discord webhook URL (loaded from environment or config)
DISCORD_WEBHOOK="${DISCORD_RELEASE_WEBHOOK:-}"

# ============================================================================
# DISCORD NOTIFICATION FUNCTION
# ============================================================================

send_discord_notification() {
  local phase="$1"        # start, backend, frontend, success, failed
  local message="$2"
  local color="$3"        # Decimal color code
  
  # Skip if webhook not configured
  if [ -z "$DISCORD_WEBHOOK" ]; then
    echo "⚠️  Discord webhook not configured, skipping notification"
    return 0
  fi
  
  local timestamp=$(date -u +%Y-%m-%dT%H:%M:%S.000Z)
  
  # Create Discord embed based on phase
  case "$phase" in
    start)
      local title="🚀 Deployment Started: v$VERSION"
      local description="Starting production deployment..."
      ;;
    backend)
      local title="⚙️ Backend Deployed: v$VERSION"
      local description="Backend deployment complete"
      ;;
    frontend)
      local title="🎨 Frontend Deployed: v$VERSION"
      local description="Frontend deployment complete"
      ;;
    success)
      local title="✅ Deployment Complete: v$VERSION"
      local description="Production deployment successful!"
      ;;
    failed)
      local title="❌ Deployment Failed: v$VERSION"
      local description="Production deployment encountered errors"
      color=15158332  # Red
      ;;
  esac
  
  # Build JSON payload
  local payload=$(cat <<EOF
{
  "username": "Project Ledger Releases",
  "avatar_url": "https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png",
  "embeds": [{
    "title": "$title",
    "description": "$description",
    "color": $color,
    "fields": [
      {
        "name": "Version",
        "value": "v$VERSION",
        "inline": true
      },
      {
        "name": "Revision",
        "value": "$REVISION_SUFFIX",
        "inline": true
      },
      {
        "name": "Deployed By",
        "value": "$(whoami)",
        "inline": true
      },
      {
        "name": "Changes",
        "value": "$CHANGES_DESCRIPTION",
        "inline": false
      },
      {
        "name": "Application URLs",
        "value": "[Frontend](https://app.projectledger.ca) | [Backend](https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io/api/health)",
        "inline": false
      }
    ],
    "timestamp": "$timestamp"
  }]
}
EOF
)
  
  # Send notification
  curl -H "Content-Type: application/json" \
    -d "$payload" \
    "$DISCORD_WEBHOOK" \
    --silent --output /dev/null
  
  if [ $? -eq 0 ]; then
    echo "✅ Discord notification sent: $phase"
  else
    echo "⚠️  Failed to send Discord notification (continuing anyway)"
  fi
}

# ============================================================================
# MAIN DEPLOYMENT SCRIPT
# ============================================================================

echo "🚀 Project Ledger Deployment Script"
echo "===================================="
echo ""
echo "Version: v$VERSION"
echo "Revision: $REVISION_SUFFIX"
echo ""

# Send initial notification
send_discord_notification "start" "" 3447003  # Blue

# ... existing prerequisite checks ...

# Build and push images
echo "🏗️  Building and pushing Docker images..."

# ... existing docker build commands ...

# Deploy backend
echo "🚀 Deploying backend to Azure..."
az containerapp update \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --image projectledgerregistry.azurecr.io/projectledger-backend:latest \
  --revision-suffix ${REVISION_SUFFIX}

if [ $? -eq 0 ]; then
  echo "✅ Backend deployment successful"
  send_discord_notification "backend" "" 3066993  # Green
else
  echo "❌ Backend deployment failed"
  send_discord_notification "failed" "Backend deployment failed" 15158332
  exit 1
fi

# Deploy frontend
echo "🚀 Deploying frontend to Azure..."
az containerapp update \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --image projectledgerregistry.azurecr.io/projectledger-frontend:latest \
  --revision-suffix ${REVISION_SUFFIX}

if [ $? -eq 0 ]; then
  echo "✅ Frontend deployment successful"
  send_discord_notification "frontend" "" 3066993  # Green
else
  echo "❌ Frontend deployment failed"
  send_discord_notification "failed" "Frontend deployment failed" 15158332
  exit 1
fi

# Final success notification
echo ""
echo "✅ Deployment complete!"
send_discord_notification "success" "" 3066993  # Green

echo ""
echo "🔗 Frontend: https://app.projectledger.ca"
echo "🔗 Backend: https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io"
```

**Key Features:**
- Notification at deployment start
- Notification after backend deployment
- Notification after frontend deployment
- Final success notification with all details
- Error notification if deployment fails
- Gracefully handles missing webhook URL
- Non-blocking (failures don't stop deployment)
- Rich embeds with version, changes, and links
- Color coding (blue=start, green=success, red=failure)

---

### Frontend Implementation

N/A - This feature does not require frontend changes.

---

## 📧 Notifications & Communication

### Discord Message Templates

#### Template 1: Documentation Update
**Trigger:** Merge to main in project-ledger-docs  
**Recipients:** #documentation-updates channel  
**Format:** Discord Embed

**Content Structure:**
- **Title:** 📝 Documentation Updated (or 📋 Specification Updated, etc.)
- **Description:** Commit message
- **Fields:**
  - Author: Commit author name
  - Commit: Short SHA with link
  - Files Changed: Number of files
  - Changed Files: List of modified files (max 10)
- **Color:** Blue (#58B9F3)
- **Timestamp:** UTC timestamp of merge

**Example:**
```
📋 Specification Updated
Add Discord notifications spec

Author: John Doe
Commit: abc1234
Files Changed: 1 file(s)

Changed Files:
SPEC_DISCORD_NOTIFICATIONS.md
```

#### Template 2: Production Release (Start)
**Trigger:** Beginning of deployment script  
**Recipients:** #releases channel  
**Format:** Discord Embed

**Content Structure:**
- **Title:** 🚀 Deployment Started: vX.Y.Z
- **Description:** Starting production deployment...
- **Fields:**
  - Version: vX.Y.Z
  - Revision: vX-Y-Z
  - Deployed By: Username
  - Changes: List of changes
- **Color:** Blue (#3498DB)
- **Timestamp:** UTC timestamp

#### Template 3: Production Release (Success)
**Trigger:** Successful completion of deployment  
**Recipients:** #releases channel  
**Format:** Discord Embed

**Content Structure:**
- **Title:** ✅ Deployment Complete: vX.Y.Z
- **Description:** Production deployment successful!
- **Fields:**
  - Version: vX.Y.Z
  - Revision: vX-Y-Z
  - Deployed By: Username
  - Changes: List of changes
  - Application URLs: Links to frontend and backend
- **Color:** Green (#2ECC71)
- **Timestamp:** UTC timestamp

**Example:**
```
✅ Deployment Complete: v1.2.0
Production deployment successful!

Version: v1.2.0
Revision: v1-2-0
Deployed By: deployer
Changes:
  ✅ Fixed login timeout bug
  ✅ Improved mobile responsiveness
  ✅ Added user notifications

Application URLs: Frontend | Backend
```

#### Template 4: Production Release (Failed)
**Trigger:** Deployment failure  
**Recipients:** #releases channel  
**Format:** Discord Embed

**Content Structure:**
- **Title:** ❌ Deployment Failed: vX.Y.Z
- **Description:** Production deployment encountered errors
- **Fields:**
  - Version: vX.Y.Z
  - Revision: vX-Y-Z
  - Deployed By: Username
  - Error: Error message or phase that failed
- **Color:** Red (#E74C3C)
- **Timestamp:** UTC timestamp

---

## 🔐 Security & Compliance

### Authentication & Authorization

**Webhook Security:**
- Discord webhooks are authenticated by URL (contains secret token)
- URLs stored as GitHub Secrets (encrypted at rest)
- URLs never committed to repository
- Only GitHub Actions and deployment scripts have access
- Webhook URLs can be regenerated in Discord if compromised

**Access Control:**
- Only repository administrators can add/modify GitHub Secrets
- Only authorized deployers have access to deployment environment variables
- Discord server administrators control webhook creation/deletion

### Data Security

**Sensitive Information:**
- No PII included in notifications
- No customer data included in notifications
- No credentials or secrets in notifications
- Only public repository information shared

**Information Disclosure:**
- Commit messages are public (already visible on GitHub)
- Version numbers are non-sensitive
- Deployment status is organizational information
- File names may reveal internal structure (acceptable risk)

### Audit Trail

**Logged Events:**
- Discord notifications are visible in Discord channel (permanent history)
- GitHub Actions logs show notification attempts
- Deployment script logs show notification calls

**Logged Information:**
- What: Documentation update or release deployment
- When: Timestamp in UTC
- Who: Commit author or deployer username
- Where: Repository and branch/environment

---

## 📋 Implementation Plan

### Week 1: Discord Setup and Documentation Notifications

#### Day 1: Discord Configuration
**Developer:** DevOps/Platform Team  
**Tasks:**
1. Create Discord webhooks in server
   - #documentation-updates channel webhook
   - #releases channel webhook
   - Optional: #alerts channel webhook for failures
2. Add webhooks as GitHub Secrets
   - `DISCORD_DOCS_WEBHOOK` in project-ledger-docs repo
   - `DISCORD_RELEASE_WEBHOOK` in project-ledger-app repo
3. Test webhook connectivity
   - Send test message to each webhook
   - Verify formatting and permissions

**Success Criteria:**
- [ ] Discord webhooks created and tested
- [ ] Webhooks added to GitHub Secrets
- [ ] Test messages successfully delivered

#### Day 2-3: Documentation Workflow Implementation
**Developer:** DevOps/Platform Team  
**Tasks:**
1. Update `.github/workflows/validate-docs.yml`
   - Add `notify-discord` job
   - Implement changed files detection
   - Format Discord embed message
   - Add error handling
2. Test workflow with sample commits
   - Create test branch
   - Make documentation changes
   - Merge to main and verify notification
3. Refine message formatting
   - Adjust file list truncation
   - Test different file types (specs, guides, etc.)
   - Verify emoji selection logic

**Success Criteria:**
- [ ] GitHub workflow triggers on main merge
- [ ] Notification sent to Discord successfully
- [ ] Message formatting is clear and readable
- [ ] Changed files listed accurately
- [ ] Links to commits work correctly

### Week 2: Release Notifications and Testing

#### Day 4-5: Deployment Script Integration
**Developer:** DevOps/Platform Team  
**Tasks:**
1. Update `tools/deployment/release-template.sh`
   - Add `send_discord_notification` function
   - Integrate notification calls at key points
   - Add error handling and fallbacks
   - Test with staging environment
2. Update existing release scripts
   - Apply template changes to current release scripts
   - Verify backward compatibility
   - Test with dry-run deployment
3. Document webhook configuration
   - Add instructions to deployment docs
   - Update HOW_TO_RELEASE.md
   - Create webhook setup guide

**Success Criteria:**
- [ ] Notification function implemented and tested
- [ ] Integration points identified and working
- [ ] Error handling prevents deployment failures
- [ ] Documentation updated

#### Day 6: End-to-End Testing
**Developer:** DevOps/Platform Team  
**Tasks:**
1. Test documentation notifications
   - Make various types of documentation changes
   - Verify all notification triggers work
   - Test with multiple commits
   - Verify rate limiting behavior
2. Test release notifications
   - Run full deployment to staging
   - Verify all notification phases trigger
   - Test failure scenarios
   - Verify message accuracy
3. Fix any issues discovered
   - Adjust formatting if needed
   - Fix edge cases
   - Optimize message content

**Success Criteria:**
- [ ] All notification types tested
- [ ] Messages are accurate and timely
- [ ] No deployment disruptions
- [ ] Edge cases handled gracefully

#### Day 7: Documentation and Rollout
**Developer:** DevOps/Platform Team  
**Tasks:**
1. Create setup documentation
   - How to configure Discord webhooks
   - How to add GitHub Secrets
   - How to customize messages
   - Troubleshooting guide
2. Team communication
   - Announce new notification system
   - Share Discord channel links
   - Gather feedback
3. Monitor initial usage
   - Watch for any issues
   - Collect team feedback
   - Make adjustments as needed

**Success Criteria:**
- [ ] Documentation complete and clear
- [ ] Team informed and onboarded
- [ ] Initial notifications working smoothly
- [ ] Feedback mechanism in place

---

## 🎯 Quality Gates

### Implementation Complete Gate
**Required before marking feature as complete:**
- [ ] Discord webhooks configured for both repos
- [ ] GitHub Secrets added and verified
- [ ] Documentation workflow updated and tested
- [ ] Deployment script updated and tested
- [ ] All notification types working correctly
- [ ] Error handling prevents deployment failures
- [ ] Documentation updated (setup guide, troubleshooting)
- [ ] Team notified and trained
- [ ] At least 3 successful production deployments with notifications
- [ ] No critical bugs or issues reported

---

## 🧪 Testing Strategy

### Test Coverage

#### Unit Tests
**Scope:** Individual notification functions
- [ ] Discord webhook payload formatting
- [ ] Error handling for missing webhook URL
- [ ] Message truncation logic
- [ ] Emoji selection logic

#### Integration Tests
**Scope:** End-to-end notification flow
- [ ] GitHub workflow triggers on merge
- [ ] Deployment script sends notifications
- [ ] Discord receives and displays messages
- [ ] Links in messages are functional

#### End-to-End Tests
**Scope:** Full user experience
- [ ] Documentation update workflow (commit → merge → notification)
- [ ] Release deployment workflow (deploy → notifications → success message)
- [ ] Error scenario (failed deployment → error notification)

### Test Scenarios

#### Scenario 1: Documentation Update
**Given:** A documentation file is updated on a feature branch  
**When:** The branch is merged to main  
**Then:** Discord notification sent to #documentation-updates

**Test Steps:**
1. Create feature branch with documentation changes
2. Commit and push changes
3. Create pull request
4. Merge pull request to main
5. Wait for GitHub Actions to complete

**Expected Results:**
- Notification appears in Discord within 30 seconds
- Message includes commit author, message, and changed files
- Links to commit work correctly
- Message formatting is clean and readable

#### Scenario 2: Successful Production Deployment
**Given:** A new version is ready for deployment  
**When:** Deployment script is executed  
**Then:** Discord notifications sent at each phase

**Test Steps:**
1. Run deployment script with new version
2. Observe notifications at each stage
3. Verify final success message

**Expected Results:**
- "Deployment Started" message appears immediately
- "Backend Deployed" message after backend update
- "Frontend Deployed" message after frontend update
- "Deployment Complete" message with all details
- All messages within 30 seconds of event
- Application URLs work correctly

#### Scenario 3: Failed Deployment
**Given:** A deployment encounters an error  
**When:** Deployment script detects failure  
**Then:** Discord error notification sent

**Test Steps:**
1. Simulate deployment failure (e.g., invalid image tag)
2. Observe error notification
3. Verify deployment script exits appropriately

**Expected Results:**
- Error notification sent immediately
- Message clearly indicates failure phase
- Deployment script stops (doesn't continue after error)
- Error notification is red color
- Team can identify issue from notification

---

## 📊 Success Metrics

### Key Performance Indicators (KPIs)

| Metric | Target | Current | Status |
|--------|--------|---------|--------|
| Notification Delivery Success Rate | > 99% | N/A | 🔴 |
| Notification Latency | < 30 seconds | N/A | 🔴 |
| Team Awareness of Updates | > 80% of team sees notifications | N/A | 🔴 |
| Reduction in Manual Announcements | > 90% | N/A | 🔴 |

### User Satisfaction Metrics
- Team survey: How useful are Discord notifications? (1-5 scale)
- Feedback: Are notifications too frequent/not frequent enough?
- Engagement: How many team members react to/comment on notifications?

### Technical Metrics
- Webhook uptime: 99.9% availability
- GitHub Actions success rate: 100% (notifications shouldn't fail builds)
- Deployment script reliability: No notification failures block deployments

---

## 🚀 Deployment Plan

### Pre-Deployment Checklist
- [ ] Discord server and channels set up
- [ ] Webhooks created in Discord
- [ ] GitHub Secrets configured
- [ ] Code changes tested in development
- [ ] Documentation prepared
- [ ] Team briefed on new notification system

### Deployment Steps

#### 1. Discord Configuration
```bash
# Manual steps in Discord:
# 1. Navigate to #documentation-updates channel
# 2. Channel Settings → Integrations → Webhooks
# 3. Create Webhook → Copy Webhook URL
# 4. Repeat for #releases channel
```

#### 2. GitHub Secrets Configuration
```bash
# In GitHub UI:
# 1. Go to repository Settings → Secrets and variables → Actions
# 2. Add new secret: DISCORD_DOCS_WEBHOOK
# 3. Paste webhook URL from Discord
# 4. Repeat for project-ledger-app repo with DISCORD_RELEASE_WEBHOOK
```

#### 3. Deploy Documentation Workflow
```bash
# 1. Create feature branch
git checkout -b feature/discord-notifications

# 2. Update .github/workflows/validate-docs.yml
# (Add notify-discord job)

# 3. Commit and push
git add .github/workflows/validate-docs.yml
git commit -m "feat: Add Discord notifications for documentation updates"
git push origin feature/discord-notifications

# 4. Create PR and merge to main
# 5. Verify notification sent to Discord
```

#### 4. Deploy Release Script Updates
```bash
# 1. Update release-template.sh
# (Add Discord notification function)

# 2. Commit to main
git add tools/deployment/release-template.sh
git commit -m "feat: Add Discord notifications for releases"
git push origin main

# 3. Test with next production deployment
```

#### 5. Post-Deployment Verification
- [ ] Test documentation update notification
- [ ] Test release notification with staging deployment
- [ ] Verify all notification phases work
- [ ] Confirm error handling works
- [ ] Check message formatting in Discord

### Rollback Procedures

**If Discord notifications cause issues:**

1. **Disable Documentation Notifications:**
```bash
# Remove notify-discord job from workflow
git checkout main
# Edit .github/workflows/validate-docs.yml
# Comment out notify-discord job
git commit -m "fix: Temporarily disable Discord notifications"
git push origin main
```

2. **Disable Release Notifications:**
```bash
# Comment out notification calls in deployment script
# Or unset DISCORD_RELEASE_WEBHOOK environment variable
unset DISCORD_RELEASE_WEBHOOK
# Run deployment as normal
```

3. **Complete Removal:**
```bash
# Remove webhook URLs from GitHub Secrets
# Revert workflow and script changes
git revert <commit-sha>
git push origin main
```

**Rollback Triggers:**
- Notification failures block deployments
- Excessive notification spam
- Security concerns with webhook exposure
- Discord outage affecting critical operations

---

## 📈 Monitoring & Observability

### Application Monitoring

**Metrics to Track:**
- Webhook HTTP response codes (200 = success)
- Notification delivery time (timestamp in webhook vs. Discord receipt)
- Failure rate (4xx/5xx responses from Discord API)

**Alerts:**
- Alert if webhook returns 4xx errors (bad request) → Review payload format
- Alert if webhook returns 5xx errors (Discord outage) → Monitor Discord status
- Alert if notifications fail 3+ times in a row → Investigate webhook configuration

### Error Tracking

**Error Types to Monitor:**
- **Network errors**: Cannot reach Discord webhook URL
- **Authentication errors**: Webhook URL invalid or expired
- **Rate limiting**: Too many notifications in short period (Discord rate limits)
- **Payload errors**: Malformed JSON in webhook request

**Response:**
- Network errors: Check internet connectivity, Discord status
- Auth errors: Regenerate webhook URL in Discord
- Rate limiting: Implement backoff/retry logic
- Payload errors: Fix JSON formatting in code

### Performance Monitoring

**Performance Targets:**
- Notification delivery: < 30 seconds from trigger
- Workflow execution: < 2 minutes total
- Deployment script overhead: < 10 seconds added

---

## 🐛 Troubleshooting Guide

### Common Issues

#### Issue 1: Notifications Not Being Sent
**Symptoms:** No Discord message appears after merge or deployment

**Possible Causes:**
- Webhook URL not configured
- Webhook URL expired/invalid
- GitHub Secret not set
- Network connectivity issue

**Solutions:**
1. Verify webhook URL in Discord (should be active)
2. Check GitHub Secret is set correctly
3. Test webhook manually with curl:
```bash
curl -H "Content-Type: application/json" \
  -d '{"content": "Test message"}' \
  "YOUR_WEBHOOK_URL"
```
4. Check GitHub Actions logs for errors
5. Verify deployment script has DISCORD_RELEASE_WEBHOOK set

**Prevention:**
- Set up monitoring to detect failed notifications
- Periodically test webhooks
- Document webhook rotation process

#### Issue 2: Duplicate Notifications
**Symptoms:** Multiple identical notifications for same event

**Possible Causes:**
- Workflow triggered multiple times
- Deployment script called notification function multiple times
- Retry logic causing duplicates

**Solutions:**
1. Check GitHub Actions logs for duplicate runs
2. Review workflow triggers (ensure only one trigger condition)
3. Add deduplication logic based on commit SHA or version
4. Verify deployment script flow control

**Prevention:**
- Use unique identifiers in notifications
- Implement idempotency in notification logic

#### Issue 3: Malformed Messages
**Symptoms:** Discord message appears broken or empty

**Possible Causes:**
- Invalid JSON in webhook payload
- Special characters not escaped
- Field values empty/null
- Discord embed limits exceeded

**Solutions:**
1. Validate JSON payload before sending:
```bash
echo "$payload" | jq .  # Validate JSON syntax
```
2. Escape special characters in variables:
```bash
COMMIT_MSG=$(echo "$COMMIT_MSG" | sed 's/"/\\"/g')
```
3. Check Discord embed limits (6000 chars title+description, 25 fields max)
4. Test with minimal payload first, then add fields

**Prevention:**
- Use JSON templating library or tool
- Validate payload in development
- Set maximum lengths for dynamic content

---

## 🔄 Rollback Procedures

### Workflow Rollback
**If documentation notifications cause issues:**
1. Navigate to repository settings → Actions
2. Disable "Validate Documentation" workflow temporarily
3. Revert workflow file changes:
```bash
git revert <commit-sha-of-discord-changes>
git push origin main
```
4. Document issue for future resolution
5. Update timeline for fix

### Deployment Script Rollback
**If release notifications break deployments:**
1. Immediately unset webhook environment variable:
```bash
unset DISCORD_RELEASE_WEBHOOK
```
2. Continue deployment without notifications
3. Revert script changes in next update:
```bash
git revert <commit-sha-of-discord-changes>
```
4. Post-mortem to identify root cause

### Full Rollback
**If widespread issues occur:**
1. Remove webhook URLs from GitHub Secrets
2. Revert all Discord notification code
3. Notify team of rollback
4. Plan revision with fixes

---

## 📢 Communication Plan

### Stakeholder Updates

#### Initial Announcement
**Audience:** All team members  
**Channel:** Discord (general or announcements channel)  
**Content:**
- New notification system being implemented
- Which channels to watch (#documentation-updates, #releases)
- What to expect (notification frequency, format)
- How to provide feedback

#### Weekly Status Reports
**Audience:** Team leads, project managers  
**Channel:** Email or team meeting  
**Content:**
- Implementation progress
- Number of notifications sent
- Team feedback summary
- Issues encountered and resolutions

#### Completion Announcement
**Audience:** All team members  
**Channel:** Discord  
**Content:**
- Feature is live and stable
- Reminder of which channels to watch
- How notifications improve workflow
- Feedback mechanism

---

## ⚠️ Risk Assessment

### High Risk Items
- **Webhook URL Exposure**: If webhook URL is leaked, anyone can send messages to Discord channel
- **Deployment Blocking**: If notification code has bugs, could prevent deployments
- **Notification Spam**: Too many notifications could overwhelm team

### Mitigation Strategies
1. **Webhook Security Mitigation**:
   - Store webhooks as secrets only
   - Rotate webhooks periodically
   - Monitor Discord channels for unauthorized messages
   - Use webhook-specific permissions in Discord

2. **Deployment Blocking Mitigation**:
   - All notification calls are non-blocking (failures don't stop deployment)
   - Wrap notification calls in try-catch or error handling
   - Test thoroughly before production rollout
   - Have rollback procedure ready

3. **Notification Spam Mitigation**:
   - Limit notifications to main branch merges only (not all commits)
   - Consolidate multiple commits in one notification when possible
   - Use Discord rate limiting awareness
   - Monitor team feedback on notification frequency

### Dependencies
- Discord API availability (99.9% uptime SLA)
- GitHub Actions functionality (core platform dependency)
- Internet connectivity during deployments
- curl utility for webhook calls

---

## 🚀 Future Enhancements

### Short Term (1-3 months)
1. **Notification Customization**
   - Allow configuration of which events trigger notifications
   - Add ability to @mention specific team members for critical changes
   - Custom notification templates for different document types
   - Estimated effort: 1-2 days

2. **Enhanced Release Notifications**
   - Include deployment duration in success message
   - Add health check results to notification
   - Link to deployment logs or monitoring dashboard
   - Estimated effort: 2-3 days

### Medium Term (3-6 months)
1. **Interactive Notifications**
   - Use Discord buttons for common actions
   - Quick rollback button on release notifications
   - React to approve/acknowledge deployment
   - Estimated effort: 1 week

2. **Notification Analytics**
   - Track notification engagement
   - Measure team response times
   - Identify most frequently updated documentation
   - Estimated effort: 3-5 days

### Long Term (6-12 months)
1. **Multi-Platform Support**
   - Also send to Slack if team uses it
   - Microsoft Teams integration
   - Email fallback option
   - Estimated effort: 2 weeks

2. **Advanced Discord Bot**
   - Query deployment status via Discord commands
   - Request rollbacks via Discord
   - View recent deployments and documentation changes
   - Estimated effort: 1 month

---

## 📝 Change Log

### Version 1.0 (November 18, 2025)
- Initial specification created
- Defined documentation and release notification requirements
- Outlined implementation plan
- Created technical specifications for GitHub workflows and deployment scripts

---

## 📚 Related Documentation

- [HOW_TO_RELEASE.md](../../deployment/HOW_TO_RELEASE.md) - Release process documentation
- [DEPLOYMENT_PLAN.md](../../deployment/DEPLOYMENT_PLAN.md) - Deployment procedures
- [validate-docs.yml](../../.github/workflows/validate-docs.yml) - Current documentation workflow
- [Discord Webhooks Documentation](https://discord.com/developers/docs/resources/webhook) - Official Discord API docs
- [GitHub Actions Documentation](https://docs.github.com/en/actions) - GitHub workflow reference

---

## ✅ Success Criteria

### Overall Success Criteria
- [ ] Discord webhooks configured and operational
- [ ] Documentation changes trigger notifications automatically
- [ ] Production releases send notifications at all phases
- [ ] Notifications are timely (< 30 seconds latency)
- [ ] Notifications are accurate and informative
- [ ] No deployment disruptions from notification system
- [ ] Team finds notifications useful (survey rating > 4/5)
- [ ] 90% reduction in manual deployment announcements

### Phase-Specific Success Criteria

#### Phase 1: Discord Setup
- [ ] Discord channels created (#documentation-updates, #releases)
- [ ] Webhooks generated and tested
- [ ] GitHub Secrets configured in both repositories
- [ ] Test messages successfully delivered

#### Phase 2: Documentation Notifications
- [ ] GitHub workflow updated with notification job
- [ ] Notifications trigger on main branch merges only
- [ ] Message formatting is clear and readable
- [ ] Changed files listed accurately
- [ ] Links to commits functional
- [ ] No workflow failures due to notification code

#### Phase 3: Release Notifications
- [ ] Deployment script updated with notification function
- [ ] Notifications sent at deployment start, backend complete, frontend complete, and final success
- [ ] Error notifications sent on deployment failures
- [ ] Notifications include all relevant information (version, changes, URLs)
- [ ] Deployment not blocked by notification failures

---

## 📞 Support & Contacts

### Development Team
- **DevOps Engineer:** TBD
- **Platform Team:** TBD
- **Documentation Maintainer:** TBD

### Stakeholders
- **Product Owner:** TBD
- **Project Manager:** TBD
- **Team Lead:** TBD

### Support Channels
- **Implementation Questions:** Discord #dev-discussion
- **Notification Issues:** Create issue in respective repository
- **Discord Admin:** Contact server administrator

---

## 📊 Timeline Summary

| Phase | Duration | Components | Features | Dependencies |
|-------|----------|------------|----------|--------------|
| **Phase 1: Setup** | 1 day | Discord webhooks, GitHub Secrets | Configuration | Discord access |
| **Phase 2: Docs** | 2-3 days | GitHub workflow | Documentation notifications | Phase 1 |
| **Phase 3: Releases** | 2-3 days | Deployment script | Release notifications | Phase 1 |
| **Phase 4: Testing** | 1 day | All components | End-to-end validation | Phases 1-3 |
| **Total** | **6-8 days** | **3 components** | **2 notification types** | Discord, GitHub |

---

## 🎯 Business Impact

### Why This Is Important
1. **Improved Team Communication**
   - Reduces time spent manually announcing updates
   - Centralizes notifications in team's primary tool (Discord)
   - Ensures all team members are aware of changes

2. **Increased Transparency**
   - Creates public record of all documentation changes
   - Provides visibility into release cadence
   - Builds trust through open communication

3. **Better Coordination**
   - Team can react to releases in real-time
   - Documentation changes are immediately visible
   - Supports distributed/remote team workflows

### Expected Outcomes
- 90% reduction in manual announcements (measured by tracking manual messages in Discord)
- 100% team awareness of production releases (measured by survey)
- 80% team awareness of documentation updates (measured by survey)
- Faster response to issues (team sees deployment notifications immediately)

### ROI Estimation
**Investment:** 6-8 days of development time  
**Expected Return:** 
- 30 minutes saved per deployment (no manual announcement) × 20 deployments/month = 10 hours/month saved
- Improved team coordination (qualitative benefit)
- Better incident response (faster awareness)

**Payback Period:** Less than 2 months based on time savings alone

---

**Document Owner:** Platform/DevOps Team  
**Created:** November 18, 2025  
**Last Updated:** November 18, 2025  
**Next Review:** After implementation completion  
**Status:** Ready for Implementation

---

## 📝 Document Usage Guidelines

This specification follows the standard template and includes:
- ✅ Complete overview with business justification
- ✅ Detailed functional and non-functional requirements
- ✅ Technical implementation specifications
- ✅ GitHub Actions workflow code
- ✅ Bash script implementation for deployment
- ✅ Discord webhook integration details
- ✅ Message templates and formatting
- ✅ Security and error handling considerations
- ✅ Comprehensive testing strategy
- ✅ Week-by-week implementation plan
- ✅ Success criteria and metrics
- ✅ Troubleshooting guide
- ✅ Risk assessment and mitigation
- ✅ Future enhancement roadmap

**Ready for:** Development team to begin implementation following the detailed plan.
