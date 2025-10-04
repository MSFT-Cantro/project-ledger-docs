# ✅ Production Deployment Checklist - v1.1.0

**Release:** v1.1.0 - Beamer Integration + UserMenu Updates  
**Date:** ________________  
**Deployer:** ________________  
**Start Time:** ________________  
**End Time:** ________________

---

## Pre-Deployment (5 min)

- [ ] All changes committed and pushed to GitHub
- [ ] Git tag v1.1.0 created
- [ ] Azure CLI logged in (`az login`)
- [ ] Docker Desktop running
- [ ] Backup current revisions saved
- [ ] Team notified of deployment
- [ ] Deployment window confirmed (low-traffic time)

---

## Build & Push (10-15 min)

- [ ] Logged into ACR (`az acr login --name projectledgerregistry`)
- [ ] Backend image built successfully
- [ ] Backend image pushed to registry (latest + v1.1.0)
- [ ] Frontend image built successfully
- [ ] Frontend image pushed to registry (latest + v1.1.0)

**Build Errors?** ________________

---

## Deployment (5-10 min)

- [ ] Backend deployed to Azure (revision: v1-1-0)
- [ ] Backend stable (waited 30 seconds)
- [ ] Frontend deployed to Azure (revision: v1-1-0)
- [ ] Frontend stable (waited 30 seconds)

**Deployment Errors?** ________________

---

## Verification & Testing (10 min)

### Health Checks
- [ ] Frontend returns HTTP 200 (https://app.projectledger.ca)
- [ ] Backend health check passes (/api/health)
- [ ] Beamer script detected in HTML (curl + grep)

### Functional Testing
- [ ] Page loads without errors
- [ ] User can login
- [ ] Dashboard loads correctly

### Beamer Integration
- [ ] Notification bell icon visible (desktop)
- [ ] Notification bell icon visible (mobile)
- [ ] Click opens Beamer panel
- [ ] Click again closes panel
- [ ] Panel has correct styling

### Theme Synchronization
- [ ] Toggle to dark mode → Beamer updates to dark
- [ ] Toggle to light mode → Beamer updates to light
- [ ] Dark mode top bar color: #1e293b
- [ ] Light mode top bar color: #2563eb

### UserMenu Updates
- [ ] Desktop menu shows "Quick Settings"
- [ ] Desktop menu has NO "Profile" option
- [ ] Desktop menu has NO "Notifications" option
- [ ] Mobile menu shows "Account Settings"
- [ ] Mobile menu has NO "Profile" option
- [ ] Mobile menu has NO "Notifications" option

### MAU Tracking
- [ ] Login and check localStorage for 'userEmail'
- [ ] Logout and verify 'userEmail' removed
- [ ] Beamer dashboard shows user activity (check later)

### Regression Testing
- [ ] Clients page works
- [ ] Projects page works
- [ ] Quotes page works
- [ ] Invoices page works
- [ ] Reports page works
- [ ] Settings page works

---

## Log Monitoring (First 30 min)

- [ ] Frontend logs checked (no errors)
- [ ] Backend logs checked (no errors)
- [ ] No increase in error rates
- [ ] Response times acceptable (<2s)

**Errors Found?** ________________

---

## Rollback (If Needed)

- [ ] Previous revision identified: ________________
- [ ] Rollback command executed
- [ ] Previous version active and verified
- [ ] Issue documented: ________________

---

## Post-Deployment

### Immediate
- [ ] Deployment log updated
- [ ] Team notified of completion
- [ ] Monitoring dashboard open
- [ ] Issues documented (if any)

### First Hour
- [ ] Application monitored continuously
- [ ] User feedback collected
- [ ] Performance metrics reviewed
- [ ] Beamer dashboard checked

### First Day
- [ ] MAU tracking verified in Beamer
- [ ] Error rates reviewed
- [ ] User engagement with notifications monitored
- [ ] No major issues reported

---

## Success Criteria

- [ ] Zero downtime during deployment
- [ ] All functional tests passed
- [ ] No increase in error rates
- [ ] Response times within acceptable range
- [ ] Beamer integration working correctly
- [ ] UserMenu updates applied correctly
- [ ] No user complaints

---

## Sign-Off

**Deployment Status:** ☐ Success  ☐ Failed  ☐ Rolled Back

**Notes:**
_____________________________________________________________________________
_____________________________________________________________________________
_____________________________________________________________________________
_____________________________________________________________________________

**Deployer Signature:** ________________  
**Date/Time:** ________________

---

## Quick Reference Commands

### Check Revisions
```bash
az containerapp revision list --name projectledger-frontend --resource-group projectledger-poc -o table
```

### View Logs
```bash
az containerapp logs show --name projectledger-frontend --resource-group projectledger-poc --tail 50 --follow
```

### Rollback
```bash
az containerapp revision activate --name projectledger-frontend --resource-group projectledger-poc --revision <previous-revision>
```

### Health Check
```bash
curl -I https://app.projectledger.ca
curl https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io/api/health
```

---

**Emergency Contact:** ________________  
**Escalation Contact:** ________________
