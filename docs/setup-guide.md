# Comprehensive Setup Guide

## Overview

This guide provides step-by-step instructions for setting up a cohesive project management system integrating GitHub, Linear, and Notion.

## Prerequisites

### Required Accounts
- **GitHub**: Repository admin access
- **Linear**: Workspace admin permissions
- **Notion**: Workspace with integration capabilities

### Development Environment
- Node.js 18+ 
- Git 2.30+
- Text editor or IDE
- API testing tool (Postman, curl)

## Step 1: GitHub Repository Setup

### 1.1 Create Repository
```bash
# Create new repository
gh repo create your-project-name --public --clone
cd your-project-name

# Initialize with README
echo "# Your Project Name" > README.md
git add README.md
git commit -m "Initial commit"
git push origin main
```

### 1.2 Configure Repository Settings
1. Navigate to Settings > General
2. Enable "Issues" and "Projects"
3. Set up branch protection rules
4. Configure merge settings

### 1.3 Set up GitHub Secrets
```bash
# Add API tokens to GitHub Secrets
gh secret set LINEAR_API_KEY
gh secret set NOTION_API_KEY
gh secret set GITHUB_TOKEN
```

## Step 2: Linear Workspace Configuration

### 2.1 Create Team and Project
1. Go to Linear Settings > Teams
2. Create new team with appropriate key (e.g., "PROJ")
3. Navigate to Projects and create new project
4. Link project to your team

### 2.2 Configure Workflow States
Set up the following states:
- **Backlog** (type: backlog)
- **Todo** (type: unstarted)
- **In Progress** (type: started)
- **In Review** (type: started)
- **Done** (type: completed)
- **Canceled** (type: canceled)

### 2.3 GitHub Integration Setup
1. Go to Settings > Integrations > GitHub
2. Click "Add GitHub integration"
3. Authorize Linear to access your GitHub account
4. Select the repository you want to connect
5. Configure integration settings:
   - Enable "Create Linear issues from GitHub issues"
   - Enable "Update Linear issues from GitHub PRs"
   - Set branch naming convention: `feature/PROJ-123-description`

## Step 3: Notion Workspace Setup

### 3.1 Create Integration
1. Go to https://developers.notion.com/
2. Click "Create new integration"
3. Name your integration (e.g., "Project Management Bot")
4. Select your workspace
5. Copy the integration token

### 3.2 Set up Project Hub Page
```markdown
# Create main project page
1. Create new page in Notion
2. Title: "🚀 Your Project Name"
3. Add sections for:
   - Project overview
   - Quick links
   - Status dashboard
   - Team information
```

### 3.3 Create Databases

#### Project Tasks Database
```
Properties:
- Task Name (Title)
- Status (Select: Not Started, In Progress, In Review, Done, Blocked)
- Priority (Select: Urgent, High, Medium, Low)
- Platform (Multi-select: GitHub, Linear, Notion)
- Assignee (Person)
- Due Date (Date)
- Linear Issue (URL)
- GitHub Link (URL)
```

#### Meeting Notes Database
```
Properties:
- Meeting Title (Title)
- Date (Date)
- Attendees (Person)
- Meeting Type (Select: Standup, Planning, Review, Retrospective)
- Action Items (Rich Text)
- Decisions Made (Rich Text)
```

### 3.4 GitHub Integration
1. In Notion, go to Settings > Connections
2. Connect to GitHub
3. Select repositories to sync
4. Configure which data to import

## Step 4: Automation Setup

### 4.1 GitHub Actions Workflow

Create `.github/workflows/linear-sync.yml`:

```yaml
name: Linear Sync

on:
  issues:
    types: [opened, edited, closed]
  pull_request:
    types: [opened, edited, closed, merged]

jobs:
  sync-linear:
    runs-on: ubuntu-latest
    steps:
      - name: Sync with Linear
        uses: linear/action-sync@v1
        with:
          linear-api-key: ${{ secrets.LINEAR_API_KEY }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### 4.2 Webhook Configuration

#### Linear Webhook
1. Go to Linear Settings > API > Webhooks
2. Create new webhook
3. URL: Your automation service endpoint
4. Select events: Issue created, updated, completed

#### GitHub Webhook
1. Repository Settings > Webhooks
2. Add webhook with your automation endpoint
3. Select events: Issues, Pull requests, Push

### 4.3 Automation Platform (Zapier/Make)

#### Zapier Setup
1. Create new Zap
2. Trigger: Linear "Issue Updated"
3. Action: Notion "Update Database Item"
4. Map fields appropriately
5. Test and activate

#### Alternative: Custom Node.js Script
```javascript
// webhook-handler.js
const express = require('express');
const { Linear } = require('@linear/sdk');
const { Client } = require('@notionhq/client');

const app = express();
const linear = new Linear(process.env.LINEAR_API_KEY);
const notion = new Client({ auth: process.env.NOTION_API_KEY });

app.post('/webhook/linear', async (req, res) => {
  const { data, type } = req.body;
  
  if (type === 'Issue' && data.action === 'update') {
    // Update Notion database
    await notion.pages.update({
      page_id: data.issue.notionPageId,
      properties: {
        'Status': {
          select: { name: mapLinearStatusToNotion(data.issue.state.name) }
        }
      }
    });
  }
  
  res.status(200).send('OK');
});

app.listen(3000);
```

## Step 5: Testing and Validation

### 5.1 Integration Testing

1. **GitHub → Linear Test**
   ```bash
   # Create branch with Linear issue ID
   git checkout -b feature/PROJ-123-test-integration
   
   # Make commit referencing Linear issue
   git commit -m "PROJ-123: Add test integration"
   
   # Push and create PR
   git push origin feature/PROJ-123-test-integration
   gh pr create --title "Test integration" --body "Tests PROJ-123"
   ```

2. **Linear → Notion Test**
   - Create issue in Linear
   - Update issue status
   - Verify Notion database updates

3. **GitHub → Notion Test**
   - Push code changes
   - Create/update issues
   - Check Notion page updates

### 5.2 Validation Checklist
- [ ] GitHub commits link to Linear issues
- [ ] PR creation updates Linear issue status
- [ ] Linear issue updates sync to Notion
- [ ] Notion database reflects current project status
- [ ] Webhooks deliver events reliably
- [ ] Error handling works correctly

## Step 6: Security Configuration

### 6.1 API Token Management

```bash
# Rotate tokens regularly (recommended: every 90 days)
# Store in secure location (GitHub Secrets, environment variables)
# Use minimal required permissions

# GitHub Token Permissions:
# - repo (for repository access)
# - workflow (for GitHub Actions)

# Linear API Key Permissions:
# - read:issues
# - write:issues
# - read:projects

# Notion Integration Permissions:
# - Read content
# - Update content
# - Insert content
```

### 6.2 Access Control

1. **GitHub Repository**
   - Limit collaborator access
   - Enable branch protection
   - Require PR reviews
   - Enable status checks

2. **Linear Team**
   - Team-based permissions
   - Project visibility settings
   - Guest user restrictions

3. **Notion Workspace**
   - Page-level permissions
   - Database access control
   - Integration permissions

## Step 7: Monitoring and Maintenance

### 7.1 Health Monitoring

Create monitoring script:
```javascript
// health-check.js
const checkIntegrations = async () => {
  try {
    // Test GitHub API
    const githubResponse = await fetch('https://api.github.com/user', {
      headers: { 'Authorization': `token ${process.env.GITHUB_TOKEN}` }
    });
    
    // Test Linear API
    const linearResponse = await linear.viewer;
    
    // Test Notion API
    const notionResponse = await notion.users.me();
    
    console.log('All integrations healthy');
  } catch (error) {
    console.error('Integration health check failed:', error);
  }
};

// Run every 5 minutes
setInterval(checkIntegrations, 5 * 60 * 1000);
```

### 7.2 Regular Maintenance

**Weekly Tasks:**
- Review sync accuracy
- Check for failed webhooks
- Monitor API rate limits
- Update documentation

**Monthly Tasks:**
- Rotate API tokens
- Review access permissions
- Update integration settings
- Archive completed projects

**Quarterly Tasks:**
- Security audit
- Performance review
- Tool evaluation
- Process improvements

## Troubleshooting

### Common Issues

1. **Sync Delays**
   - Check webhook delivery logs
   - Verify API rate limits
   - Review error logs

2. **Authentication Errors**
   - Validate API tokens
   - Check token permissions
   - Verify integration settings

3. **Data Inconsistencies**
   - Run manual sync scripts
   - Check field mappings
   - Review automation logic

### Support Resources

- **GitHub**: [API Documentation](https://docs.github.com/en/rest)
- **Linear**: [API Docs](https://developers.linear.app/docs)
- **Notion**: [Developer Docs](https://developers.notion.com/)
- **Community**: Project discussion boards and forums

## Conclusion

This setup creates a robust, automated project management system that keeps all stakeholders informed across platforms. Regular monitoring and maintenance ensure continued reliability and performance.

For additional support or questions, refer to the project documentation or reach out to the development team.