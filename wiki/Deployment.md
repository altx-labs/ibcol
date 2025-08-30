# Deployment Process

The IBCOL platform uses a modern deployment process based on Zeit Now (Vercel) and Travis CI. This document explains how the deployment process works and how to deploy the application to different environments.

## Deployment Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│  Git Repository │────▶│  Travis CI      │────▶│  Zeit Now      │
│                 │     │                 │     │  (Vercel)       │
└─────────────────┘     └─────────────────┘     └─────────────────┘
        │                        │                       │
        │                        │                       │
        │                        ▼                       ▼
        │                ┌─────────────────┐    ┌─────────────────┐
        │                │                 │    │                 │
        └───────────────▶│  Slack          │◀───│  CDN Edge       │
                         │  Notifications  │    │  Network        │
                         │                 │    │                 │
                         └─────────────────┘    └─────────────────┘
```

## Deployment Environments

The application supports multiple deployment environments:

| Environment | Configuration | URL | Purpose |
|-------------|--------------|-----|---------|
| Local | `now-local.json` | localhost | Local development |
| Development | `now-dev.json` | dev.ibcol.org | Development testing |
| Staging | `now-stage.json` | uat.ibcol.org | User acceptance testing |
| Production | `now-production.json` | ibcol.org | Live production site |

## Environment Configuration

Each environment has its own configuration file:

```javascript
// now-production.json
{
  "version": 2,
  "name": "ibcol-production",
  "alias": ["ibcol.org", "www.ibcol.org"],
  "builds": [
    { "src": "next.config.js", "use": "@now/next" },
    { "src": "node-routes/*.js", "use": "@now/node" }
  ],
  "routes": [
    { "src": "/api/filepond/(.*)", "dest": "/node-routes/filepondRoute.js" },
    { "src": "/(.*)", "dest": "/$1" }
  ],
  "env": {
    "ENV": "production",
    "GRAPHQL_URL": "https://api.ibcol.org/graphql",
    "FILEPOND_API_URL": "https://api.ibcol.org",
    "FILEPOND_API_ENDPOINT": "/api/filepond/",
    "GCS_BUCKET_NAME": "ibcol-uploads-production",
    "SALT": "@ibcol-salt"
  }
}
```

## Deployment Scripts

The deployment process is managed through npm scripts:

```json
"scripts": {
  "deploy": "npm run deploy-dev",
  "deploy-dev": "now --local-config now-dev.json --scope bbi --target production",
  "deploy-stage": "export NOW_URL=$(now --local-config now-stage.json --scope ibcol --target production) && ALIAS_PATH='https://www.uat.ibcol.org' node slack-notifications/now-alias.js",
  "deploy-production": "export NOW_URL=$(now --local-config=now-production.json --scope ibcol --target production) && ALIAS_PATH='https://www.ibcol.org' node slack-notifications/now-alias.js",
  "cleanup-production-now": "now rm $npm_package_name-production -s"
}
```

## Travis CI Integration

Travis CI is used for continuous integration and deployment:

```yaml
# .travis.yml
language: node_js
node_js:
  - "12"

cache:
  directories:
    - node_modules

before_script:
  - npm install -g now
  - node slack-notifications/travis-before_script.js

script:
  - npm run build

before_deploy:
  - node slack-notifications/travis-before_deploy.js

deploy:
  - provider: script
    script: npm run deploy-stage
    skip_cleanup: true
    on:
      branch: dev
  - provider: script
    script: npm run deploy-production
    skip_cleanup: true
    on:
      branch: master

notifications:
  email: false
```

## Deployment Process Flow

### Local Development

1. Run the local development server:
   ```bash
   npm run local
   ```

2. The server starts with the local configuration:
   ```bash
   now dev --local-config now-local.json
   ```

3. Access the application at `http://localhost:3000`

### Development Deployment

1. Push changes to the `dev` branch
2. Travis CI builds the application
3. Travis CI deploys to the development environment:
   ```bash
   npm run deploy-dev
   ```
4. The application is deployed to `https://dev.ibcol.org`
5. Slack notifications are sent

### Staging Deployment

1. Push changes to the `dev` branch
2. Travis CI builds the application
3. Travis CI deploys to the staging environment:
   ```bash
   npm run deploy-stage
   ```
4. The application is deployed to `https://uat.ibcol.org`
5. Slack notifications are sent

### Production Deployment

1. Push changes to the `master` branch
2. Travis CI builds the application
3. Travis CI deploys to the production environment:
   ```bash
   npm run deploy-production
   ```
4. The application is deployed to `https://ibcol.org`
5. Slack notifications are sent

## Slack Notifications

The deployment process includes Slack notifications:

```javascript
// slack-notifications/now-deployed.js
const Slack = require('slack-node');

const slack = new Slack();
slack.setWebhook(process.env.SLACK_WEBHOOK_URL);

slack.webhook({
  channel: "#deployments",
  username: "Now Deployment",
  icon_emoji: ":rocket:",
  text: `Deployment to ${process.env.ALIAS_PATH} completed successfully!`,
  attachments: [
    {
      fallback: `Deployment to ${process.env.ALIAS_PATH} completed successfully!`,
      color: "#36a64f",
      fields: [
        {
          title: "Environment",
          value: process.env.ENV,
          short: true
        },
        {
          title: "URL",
          value: process.env.ALIAS_PATH,
          short: true
        },
        {
          title: "Deployment URL",
          value: process.env.NOW_URL,
          short: false
        }
      ]
    }
  ]
}, function(err, response) {
  if (err) {
    console.error('Error sending Slack notification:', err);
  } else {
    console.log('Slack notification sent successfully');
  }
});
```

## Zeit Now (Vercel) Configuration

The application is deployed as a serverless application on Zeit Now (Vercel):

1. **Builds**: Specify how to build different parts of the application
2. **Routes**: Define routing rules for the application
3. **Environment Variables**: Set environment-specific variables
4. **Aliases**: Map domains to the deployment

## Deployment Secrets

Sensitive information is stored as Now secrets:

```bash
# Set a secret
now secrets add ibcol-salt "your-secret-salt-value"

# Reference the secret in configuration
{
  "env": {
    "SALT": "@ibcol-salt"
  }
}
```

## Google Cloud Storage Integration

The application integrates with Google Cloud Storage for file uploads:

1. **Service Account**: A service account is used for authentication
2. **Credentials**: Stored as environment variables or secrets
3. **Bucket**: Each environment has its own storage bucket

## Rollback Process

If a deployment needs to be rolled back:

1. Identify the previous working deployment:
   ```bash
   now ls ibcol-production
   ```

2. Alias the previous deployment:
   ```bash
   now alias [deployment-url] ibcol.org
   now alias [deployment-url] www.ibcol.org
   ```

3. Send a Slack notification about the rollback

## Deployment Monitoring

The deployment is monitored through:

1. **Travis CI Build Status**: Displayed in the README
2. **Slack Notifications**: Sent on deployment events
3. **Vercel Dashboard**: For deployment status and logs

## Best Practices

1. **Test Before Deployment**: Always test changes locally before deployment
2. **Environment Variables**: Use environment-specific variables
3. **Secrets Management**: Store sensitive information as secrets
4. **Deployment Notifications**: Monitor deployment status through notifications
5. **Rollback Plan**: Have a plan for rolling back problematic deployments
6. **Cleanup Old Deployments**: Regularly clean up old deployments

