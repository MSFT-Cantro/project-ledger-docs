# How to Release a New Version

> **Quick Guide:** Step-by-step instructions for releasing a new version to production

---

## 🚀 Quick Start

```bash
# 1. Copy templates (update X.Y.Z to your version)
cp docs/deployment/RELEASE_TEMPLATE.md docs/deployment/releases/vX.Y.Z/RELEASE_NOTES_vX.Y.Z.md
cp docs/deployment/DEPLOYMENT_CHECKLIST_TEMPLATE.md docs/deployment/releases/vX.Y.Z/DEPLOYMENT_CHECKLIST_vX.Y.Z.md
cp tools/deployment/release-template.sh tools/deployment/release-vX.Y.Z.sh
chmod +x tools/deployment/release-vX.Y.Z.sh

# 2. Update version in all files
# Edit: RELEASE_NOTES_vX.Y.Z.md
# Edit: DEPLOYMENT_CHECKLIST_vX.Y.Z.md
# Edit: release-vX.Y.Z.sh

# 3. Create marketing post (see previous releases for examples)
# Create: docs/deployment/releases/vX.Y.Z/MARKETING_POST_vX.Y.Z.md
# Reference: docs/deployment/releases/v1.7.0/MARKETING_POST_v1.7.0.md

# 4. Commit templates to git
git add docs/deployment/releases/vX.Y.Z/
git add tools/deployment/release-vX.Y.Z.sh
git commit -m "docs: Add release templates for vX.Y.Z"
git push origin main

# 5. Run deployment
bash tools/deployment/release-vX.Y.Z.sh
```

---

## 📋 Detailed Steps

### Step 1: Pre-Release Preparation

**1.1 Complete Development**
- [ ] All features tested locally
- [ ] All tests passing (`npm test`)
- [ ] Code reviewed and merged to `main`
- [ ] No TypeScript errors
- [ ] No console errors

**1.2 Update Documentation**
- [ ] Update `docs/_bugs.md` (mark items as completed)
- [ ] Update `docs/_features.md` (mark items as completed)
- [ ] Update `CHANGELOG.md` with changes
- [ ] Update version in `package.json` files (optional)
- [ ] Create marketing post for the release (see Step 2.2)

**1.3 Check Database Migrations**
- [ ] Review new migrations in `apps/backend/prisma/migrations/`
- [ ] Ensure migrations are tested locally
- [ ] Verify seed data migrations (e.g., terms templates) work correctly
- [ ] Check migration history with `npx prisma migrate status`

**Note:** Database migrations run automatically during deployment via `docker-entrypoint-production.sh`. This includes:
- Schema changes
- Data seeding (e.g., terms templates auto-seeded by `20251015120000_seed_terms_templates`)
- See `docs/deployment/TERMS_TEMPLATES_MIGRATION.md` for details

**1.4 Commit All Changes**
```bash
git add .
git commit -m "chore: Prepare for v1.2.0 release"
git push origin main
```

---

### Step 2: Copy Templates

**2.1 Create Release Documents**
```bash
# Replace 1.2.0 with your actual version
VERSION="1.2.0"

# Create release directory
mkdir -p "docs/deployment/releases/v${VERSION}"

# Copy release notes template
cp docs/deployment/RELEASE_TEMPLATE.md "docs/deployment/releases/v${VERSION}/RELEASE_NOTES_v${VERSION}.md"

# Copy checklist template
cp docs/deployment/DEPLOYMENT_CHECKLIST_TEMPLATE.md "docs/deployment/releases/v${VERSION}/DEPLOYMENT_CHECKLIST_v${VERSION}.md"

# Copy deployment script
cp tools/deployment/release-template.sh "tools/deployment/release-v${VERSION}.sh"
chmod +x "tools/deployment/release-v${VERSION}.sh"
```

**2.2 Create Marketing Post**

Create a marketing announcement based on previous releases:

```bash
# Reference the v1.7.0 marketing post as a template
# File: docs/deployment/releases/v1.7.0/MARKETING_POST_v1.7.0.md

# Create your marketing post
touch "docs/deployment/releases/v${VERSION}/MARKETING_POST_v${VERSION}.md"
```

The marketing post should include:
- **Main announcement** with key features and benefits
- **Social media posts** (Twitter/X, LinkedIn, Facebook/Instagram)
- **Email announcement template**
- **Support team quick reference** with FAQs
- **Distribution checklist**

See `docs/deployment/releases/v1.7.0/MARKETING_POST_v1.7.0.md` for a complete example.

---

### Step 3: Customize Templates

**3.1 Edit Release Notes (`RELEASE_NOTES_vX.Y.Z.md`)**

Find and replace:
- `vX.Y.Z` → Your version (e.g., `v1.2.0`)
- `X.Y.Z` → Version without 'v' (e.g., `1.2.0`)
- `vX-Y-Z` → Revision suffix (e.g., `v1-2-0`)

Add your changes in the "Release Information" section:
```markdown
## Release Information

**Version:** `v1.2.0`
**Release Date:** `October 15, 2025`
**Deployer:** `Your Name`
**Revision Suffix:** `v1-2-0`

**Changes in this release:**
- Added user notifications feature
- Fixed login timeout bug
- Improved mobile responsiveness
```

**3.2 Edit Deployment Checklist (`DEPLOYMENT_CHECKLIST_vX.Y.Z.md`)**

Find and replace:
- `vX.Y.Z` → Your version (e.g., `v1.2.0`)
- `X.Y.Z` → Version without 'v'
- `vX-Y-Z` → Revision suffix

Add your specific features to test:
```markdown
### New Features
- [ ] User notifications display correctly
- [ ] Notification bell icon works
- [ ] Email notifications sent
```

**3.3 Edit Deployment Script (`release-vX.Y.Z.sh`)**

Update the configuration section at the top:
```bash
# ============================================================================
# CONFIGURATION - UPDATE THESE FOR EACH RELEASE
# ============================================================================

VERSION="1.2.0"                    # Your version
REVISION_SUFFIX="v1-2-0"           # Revision suffix for Azure
CHANGES_DESCRIPTION="
  ✅ Added user notifications feature
  ✅ Fixed login timeout bug  
  ✅ Improved mobile responsiveness
"
```

---

### Step 4: Review and Commit Templates

**4.1 Review Files**
- Open each file and verify all placeholders replaced
- Check that version numbers are consistent
- Verify changes description is accurate

**4.2 Commit Templates**
```bash
git add docs/deployment/RELEASE_NOTES_v1.2.0.md
git add docs/deployment/DEPLOYMENT_CHECKLIST_v1.2.0.md
git add tools/deployment/release-v1.2.0.sh
git commit -m "docs: Add release templates for v1.2.0"
git push origin main
```

---

### Step 5: Pre-Deployment Checks

**5.1 Verify Prerequisites**
```bash
# Check git status (should be clean)
git status

# Check Docker is running
docker ps

# Check Azure login
az account show

# Run tests one more time
cd apps/backend && npm test
cd ../frontend && npm test
```

**5.2 Review Checklist**
- Print or open `DEPLOYMENT_CHECKLIST_v1.2.0.md`
- Keep it open during deployment to check off items

---

### Step 6: Execute Deployment

**6.1 Run the Script**
```bash
bash tools/deployment/release-v1.2.0.sh
```

**6.2 Follow Prompts**
- Review changes when prompted
- Type `y` to confirm production deployment
- Wait for script to complete (5-10 minutes)

**6.3 Script Will:**
1. ✅ Check prerequisites (Docker, Azure, Git)
2. ✅ Backup current production revisions
3. ✅ Create and push git tag `v1.2.0`
4. ✅ Build Docker images (backend + frontend)
5. ✅ Push images to Azure Container Registry
6. ✅ Deploy backend to Azure
7. ✅ Deploy frontend to Azure
8. ✅ Run basic health checks

---

### Step 7: Verification

**7.1 Automated Checks**
The script automatically verifies:
- ✅ Frontend responding (200 OK)
- ✅ Backend health endpoint
- ✅ Basic connectivity

**7.2 Manual Testing**
Use your checklist (`DEPLOYMENT_CHECKLIST_v1.2.0.md`):

```bash
# Open the application
open https://app.projectledger.ca
```

Test:
- [ ] Login/logout works
- [ ] Dashboard loads
- [ ] All navigation items work
- [ ] New features work
- [ ] Mobile view works
- [ ] No console errors
- [ ] No 404/500 errors

**7.3 Monitor Logs**
```bash
# Backend logs
az containerapp logs show \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --tail 50 \
  --follow

# Frontend logs (in another terminal)
az containerapp logs show \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --tail 50 \
  --follow
```

Look for:
- No error messages
- Successful API calls
- Database connections working

---

### Step 8: Post-Deployment

**8.1 Complete Checklist**
- Check off all items in `DEPLOYMENT_CHECKLIST_v1.2.0.md`
- Note any issues encountered
- Sign off at the bottom

**8.2 Marketing and Communications**
```bash
# Publish marketing post
# Review: docs/deployment/releases/v1.2.0/MARKETING_POST_v1.2.0.md

# Post to social media (use content from marketing post)
# - Twitter/X
# - LinkedIn
# - Facebook/Instagram

# Send email announcement to customers
# Use email template from marketing post

# Brief support team
# Review FAQ section from marketing post
```

**8.3 Update Documentation**
```bash
# Update bug list
# Mark completed items in docs/_bugs.md

# Update feature list  
# Mark completed items in docs/_features.md

# Update marketing post distribution checklist
# Mark completed items in MARKETING_POST_v1.2.0.md

# Commit changes
git add docs/_bugs.md docs/_features.md
git add docs/deployment/releases/v1.2.0/MARKETING_POST_v1.2.0.md
git commit -m "docs: Update bug/feature lists and marketing post v1.2.0"
git push origin main
```

**8.4 Monitor Application**
- Watch logs for first hour
- Check for errors at end of day
- Review metrics next morning

---

## 🔙 If Something Goes Wrong

### Rollback Immediately

**1. List Current Revisions**
```bash
az containerapp revision list \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  -o table
```

**2. Activate Previous Revision**
```bash
# Use revision name from backup file or list above
az containerapp revision activate \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --revision projectledger-frontend--PREVIOUS-REVISION

# Rollback backend too if needed
az containerapp revision activate \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --revision projectledger-backend--PREVIOUS-REVISION
```

**3. Verify Rollback**
```bash
curl -I https://app.projectledger.ca
```

**4. Document Issue**
- Create incident report in `docs/deployment/incidents/`
- Include: timestamp, symptoms, cause, resolution
- Plan fix and re-release

---

## 📊 Example Timeline

**Typical release timeline:**
- Pre-release prep: 15 minutes
- Copy & customize templates: 10 minutes
- Review & commit: 5 minutes
- Run deployment script: 5-10 minutes
- Manual verification: 10-15 minutes
- Post-deployment tasks: 10 minutes

**Total: 55-75 minutes**

---

## ✅ Checklist Summary

### Before Deployment
- [ ] All code changes committed and pushed
- [ ] Tests passing
- [ ] Documentation updated
- [ ] Templates copied and customized
- [ ] Templates committed to git
- [ ] Docker running
- [ ] Azure logged in

### During Deployment
- [ ] Deployment script executed
- [ ] Health checks passed
- [ ] Manual testing completed
- [ ] Logs monitored
- [ ] Checklist completed

### After Deployment
- [ ] Bug/feature lists updated
- [ ] Documentation updated
- [ ] Team notified
- [ ] Monitoring active
- [ ] Templates archived

---

## 📚 Additional Resources

- [DEPLOYMENT_CHECKLIST_TEMPLATE.md](./DEPLOYMENT_CHECKLIST_TEMPLATE.md) - Checklist template
- [Production Release Plan](./archive/PRODUCTION_RELEASE_PLAN.md) - v1.1.0 example (archived)
- [Azure Deployment Archive](./archive/) - Historical infrastructure details

---

## 💡 Tips

1. **Always test locally first** - Never deploy untested code
2. **Use the checklist** - Don't skip verification steps
3. **Monitor logs** - Watch for errors in the first hour
4. **Document issues** - Help future releases go smoother
5. **Keep templates updated** - Improve process over time
6. **Have rollback ready** - Know how to rollback before deploying
7. **Deploy during low-traffic** - When possible (not required yet)
8. **Communicate** - Let team know about deployments

---

## 🎓 Learning From v1.1.0

**What worked well:**
- Automated script saved time and reduced errors
- Health checks caught issues immediately
- Templates ensured nothing was forgotten
- Documentation made process repeatable

**Best practices established:**
- Always backup revisions before deployment
- Use revision suffixes matching version
- Tag git commits with release versions
- Complete manual testing even with automation
- Monitor logs for at least 1 hour post-deployment

---

**Need Help?** Review [PRODUCTION_RELEASE_PLAN.md](./PRODUCTION_RELEASE_PLAN.md) for detailed procedures or ask for assistance.
