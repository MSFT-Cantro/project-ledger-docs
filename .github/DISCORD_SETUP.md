# Discord Notifications Setup Guide

This guide explains how to configure Discord webhooks for documentation update notifications.

---

## Prerequisites

- Admin access to Discord server
- Admin access to GitHub repository settings

---

## Step 1: Create Discord Webhook

1. **Navigate to Discord Channel**
   - Open Discord and go to your server
   - Navigate to the `#documentation-updates` channel (or create it if it doesn't exist)

2. **Access Channel Settings**
   - Right-click on the channel name → **Edit Channel**
   - Or click the gear icon ⚙️ next to the channel name

3. **Create Webhook**
   - Go to **Integrations** tab
   - Click **Webhooks** → **Create Webhook**
   - Name: `Project Ledger Docs` (or any name you prefer)
   - Channel: `#documentation-updates`
   - (Optional) Upload an avatar image for the webhook

4. **Copy Webhook URL**
   - Click **Copy Webhook URL**
   - **IMPORTANT:** Keep this URL secret! Anyone with this URL can send messages to your channel
   - The URL will look like: `https://discord.com/api/webhooks/123456789/AbCdEfGhIjKlMnOpQrStUvWxYz`

---

## Step 2: Add Webhook to GitHub Secrets

1. **Navigate to Repository Settings**
   - Go to: https://github.com/MSFT-Cantro/project-ledger/settings
   - Click **Secrets and variables** → **Actions**

2. **Create New Secret**
   - Click **New repository secret**
   - Name: `DISCORD_DOCS_WEBHOOK`
   - Value: Paste the Discord webhook URL you copied
   - Click **Add secret**

3. **Verify Secret**
   - You should see `DISCORD_DOCS_WEBHOOK` in the list of secrets
   - The value will be hidden (shows as `***`)

---

## Step 3: Test the Workflow

1. **Make a Documentation Change**
   ```bash
   # Create a test branch
   git checkout -b test/discord-notification
   
   # Make a small change to any markdown file
   echo "\n<!-- Test Discord notification -->" >> README.md
   
   # Commit and push
   git add README.md
   git commit -m "test: Verify Discord notification works"
   git push origin test/discord-notification
   ```

2. **Create and Merge Pull Request**
   - Go to GitHub and create a pull request
   - Merge the PR to `main`

3. **Verify Notification**
   - Check your Discord `#documentation-updates` channel
   - You should see a notification within 30 seconds with:
     - 📝 Documentation Updated
     - Commit message
     - Author name
     - Commit link
     - List of changed files

---

## Expected Notification Format

When working correctly, you'll see a Discord embed like this:

```
📝 Documentation Updated
test: Verify Discord notification works

Author: Your Name
Commit: abc1234
Files Changed: 1 file(s)

Changed Files:
README.md
```

Different file types trigger different emojis:
- 📋 Specification files (SPEC_*.md)
- 🚀 Deployment documentation (deployment/*.md)
- 📖 Guides (guides/*.md)
- 📝 Other documentation

---

## Troubleshooting

### No notification appears

1. **Check GitHub Actions**
   - Go to: https://github.com/MSFT-Cantro/project-ledger/actions
   - Look for the latest "Validate Documentation" workflow run
   - Click on it and check the "Notify Discord of Documentation Update" job
   - Review the logs for any errors

2. **Verify Secret is Set**
   - Go to repository Settings → Secrets and variables → Actions
   - Confirm `DISCORD_DOCS_WEBHOOK` exists
   - If not, add it following Step 2 above

3. **Test Webhook Manually**
   ```bash
   # Replace with your actual webhook URL
   curl -H "Content-Type: application/json" \
     -d '{"content": "Test message from command line"}' \
     "https://discord.com/api/webhooks/YOUR_WEBHOOK_URL"
   ```
   
   If this doesn't work, the webhook URL might be invalid:
   - Regenerate the webhook in Discord
   - Update the GitHub Secret

4. **Check Discord Permissions**
   - Ensure the webhook has permission to post in the channel
   - Check Discord server settings if messages are being filtered

### Workflow fails

1. **Check the error message in GitHub Actions logs**
2. **Common issues:**
   - Malformed JSON (usually auto-fixed in the workflow)
   - Network timeout (retry the merge)
   - Rate limiting (wait a few minutes)

### Duplicate notifications

- This is normal if you force-push or re-run the workflow
- Each successful merge should only trigger one notification

---

## Optional: Create Multiple Notification Channels

If you want separate channels for different types of updates:

1. **Create Additional Channels**
   - `#spec-updates` - For specification changes
   - `#guide-updates` - For guide changes
   - `#deployment-docs` - For deployment documentation

2. **Create Separate Webhooks**
   - Follow Step 1 for each channel
   - Add as separate GitHub Secrets:
     - `DISCORD_SPEC_WEBHOOK`
     - `DISCORD_GUIDE_WEBHOOK`
     - `DISCORD_DEPLOYMENT_WEBHOOK`

3. **Update Workflow**
   - Modify `.github/workflows/validate-docs.yml`
   - Add conditional logic to send to different webhooks based on changed files

---

## Security Best Practices

1. **Never commit webhook URLs to git**
   - Always use GitHub Secrets
   - Add webhook URLs to `.gitignore` if stored locally

2. **Rotate webhooks periodically**
   - Every 6-12 months, regenerate the webhook
   - Update the GitHub Secret immediately

3. **Monitor for abuse**
   - Watch for unexpected messages in Discord
   - If compromised, delete the webhook in Discord and create a new one

4. **Limit webhook permissions**
   - Webhooks can only post to their assigned channel
   - They cannot read messages or access other channels
   - They cannot perform admin actions

---

## Maintenance

### Updating the Webhook

If you need to change the webhook URL:

1. **Discord Side:**
   - Go to channel settings → Integrations → Webhooks
   - Click on existing webhook → **Copy Webhook URL**
   - Or delete and create a new webhook

2. **GitHub Side:**
   - Go to repository Settings → Secrets and variables → Actions
   - Click on `DISCORD_DOCS_WEBHOOK`
   - Click **Update secret**
   - Paste new webhook URL → **Update secret**

### Disabling Notifications

To temporarily disable notifications without removing the code:

1. **Option A: Delete the GitHub Secret**
   - Remove `DISCORD_DOCS_WEBHOOK` from GitHub Secrets
   - Workflow will skip notification (check `if: env.DISCORD_WEBHOOK != ''`)

2. **Option B: Disable the Workflow**
   - Go to repository Settings → Actions → General
   - Disable "Validate Documentation" workflow
   - Re-enable when ready

3. **Option C: Comment Out the Job**
   - Edit `.github/workflows/validate-docs.yml`
   - Comment out the `notify-discord` job
   - Commit to main

---

## Support

If you encounter issues:

1. **Check GitHub Actions logs** - Most errors are logged there
2. **Test webhook manually** - Verify webhook URL works
3. **Review Discord audit log** - Check if webhook is posting
4. **Create an issue** - If the problem persists

---

**Last Updated:** November 18, 2025  
**Workflow Version:** 1.0
