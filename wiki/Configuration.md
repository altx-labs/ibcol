# Configuration Management

The IBCOL platform uses a flexible configuration system to manage environment-specific settings. This document explains how configuration is managed across different environments.

## Configuration Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│  Config Files   │────▶│  Config Module  │────▶│  Application    │
│  (.json)        │     │  (index.js)     │     │  Components     │
│                 │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

The configuration system consists of:

1. **Environment-specific Config Files**: JSON files for each environment
2. **Config Module**: JavaScript module that loads the appropriate config
3. **Now Config Files**: Zeit Now (Vercel) configuration for deployment
4. **Environment Variables**: Runtime environment variables

## Config Files

Configuration files are stored in the `/configs` directory:

```
configs/
├── common.json     # Common settings for all environments
├── dev.json        # Development environment settings
├── index.js        # Config loader module
├── local.json      # Local development settings
├── production.json # Production environment settings
└── stage.json      # Staging environment settings
```

### Common Config

The `common.json` file contains settings shared across all environments:

```json
{
  "app": {
    "name": "International Blockchain Olympiad",
    "shortName": "IBCOL",
    "description": "International Blockchain Olympiad is a multidisciplinary design and building competition."
  },
  "social": {
    "twitter": "https://twitter.com/ibcolorg",
    "facebook": "https://www.facebook.com/ibcol.org",
    "instagram": "https://www.instagram.com/ibcol_org",
    "linkedin": "https://www.linkedin.com/company/ibcol",
    "youtube": "https://www.youtube.com/channel/UCxN1ZN_bSlWlVWncT9QcTkg"
  }
}
```

### Environment-specific Config

Each environment has its own configuration file:

```json
// production.json
{
  "env": "production",
  "baseUrl": "https://www.ibcol.org",
  "apiUrl": "https://api.ibcol.org",
  "graphqlUrl": "https://api.ibcol.org/graphql",
  "analytics": {
    "googleAnalyticsId": "UA-113535301-3"
  }
}
```

## Config Loader

The `configs/index.js` file loads the appropriate configuration based on the environment:

```javascript
const _ = require('lodash');
const common = require('./common.json');

// Determine the environment
const env = process.env.ENV || 'local';

// Load environment-specific config
let envConfig = {};
try {
  envConfig = require(`./${env}.json`);
} catch (e) {
  console.warn(`No config found for environment: ${env}, using default config`);
  envConfig = require('./local.json');
}

// Merge common and environment-specific configs
const config = _.merge({}, common, envConfig);

module.exports = config;
```

## Using Configuration

Configuration is imported and used throughout the application:

```javascript
import configs from 'configs';

// In a component:
const apiUrl = configs.apiUrl;
const appName = configs.app.name;

// Example usage:
<title>{configs.app.name}</title>
<meta name="description" content={configs.app.description} />
```

## Zeit Now (Vercel) Configuration

Each environment has a corresponding Now configuration file:

```
/
├── now-dev.json       # Development environment
├── now-local.json     # Local development
├── now-production.json # Production environment
└── now-stage.json     # Staging environment
```

Example of a Now configuration file:

```json
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

## Environment Variables

Environment variables are used for runtime configuration:

1. **Now Environment Variables**: Defined in Now config files
2. **Now Secrets**: Sensitive information stored as Now secrets
3. **Runtime Environment Variables**: Available through `process.env`

Example of accessing environment variables:

```javascript
// In a server-side component:
const graphqlUrl = process.env.GRAPHQL_URL;
const salt = process.env.SALT;

// In a client-side component (via Next.js):
import getConfig from 'next/config';
const { publicRuntimeConfig } = getConfig();
const graphqlUrl = publicRuntimeConfig.GRAPHQL_URL;
```

## Next.js Configuration

The `next.config.js` file configures Next.js and exposes environment variables to the client:

```javascript
const withCSS = require('@zeit/next-css');
const configs = require('./configs');

module.exports = withCSS({
  target: 'serverless',
  
  // Environment variables available on the client
  publicRuntimeConfig: {
    ENV: process.env.ENV,
    GRAPHQL_URL: process.env.GRAPHQL_URL,
    FILEPOND_API_URL: process.env.FILEPOND_API_URL,
    FILEPOND_API_ENDPOINT: process.env.FILEPOND_API_ENDPOINT
  },
  
  // Environment variables only available on the server
  serverRuntimeConfig: {
    SALT: process.env.SALT,
    GCS_BUCKET_NAME: process.env.GCS_BUCKET_NAME,
    GOOGLE_APPLICATION_CREDENTIALS: process.env.GOOGLE_APPLICATION_CREDENTIALS
  }
});
```

## Secrets Management

Sensitive information is managed using Now secrets:

```bash
# Set a secret
now secrets add ibcol-salt "your-secret-salt-value"

# Reference the secret in Now config
{
  "env": {
    "SALT": "@ibcol-salt"
  }
}
```

## Google Cloud Credentials

Google Cloud credentials are managed securely:

1. **Service Account JSON**: Stored as a base64-encoded environment variable
2. **Runtime Decoding**: Decoded at runtime and saved to a temporary file
3. **Environment Variable**: Path to the credentials file set as an environment variable

```javascript
// Decode and save Google Cloud credentials
if (process.env.GOOGLE_APPLICATION_CREDENTIALS_DATA) {
  const fs = require('fs');
  const credentialsData = Buffer.from(
    process.env.GOOGLE_APPLICATION_CREDENTIALS_DATA,
    'base64'
  ).toString('utf8');
  
  fs.writeFileSync(
    process.env.GOOGLE_APPLICATION_CREDENTIALS,
    credentialsData
  );
}
```

## Configuration Hierarchy

Configuration is applied in the following order (later overrides earlier):

1. **Default Values**: Hardcoded defaults in the application
2. **Common Config**: Settings in `common.json`
3. **Environment Config**: Settings in environment-specific JSON files
4. **Environment Variables**: Runtime environment variables
5. **Now Config**: Settings in Now configuration files

## Feature Flags

Feature flags can be implemented using configuration:

```json
{
  "features": {
    "registration": true,
    "adminDashboard": true,
    "fileUpload": true
  }
}
```

Usage in components:

```javascript
import configs from 'configs';

// In a component:
{configs.features.registration && (
  <Link route="registration" params={{ locale }}>
    <a className="btn">Register Now</a>
  </Link>
)}
```

## Environment Detection

The current environment can be detected using:

```javascript
import configs from 'configs';

// Check if in production
const isProduction = configs.env === 'production';

// Conditional logic based on environment
if (isProduction) {
  // Production-only code
} else {
  // Non-production code
}
```

## Best Practices

1. **Environment Separation**: Keep environment-specific settings separate
2. **Secrets Management**: Never commit sensitive information to the repository
3. **Default Values**: Provide sensible defaults for all configuration options
4. **Documentation**: Document all configuration options
5. **Validation**: Validate configuration at startup
6. **Minimal Client Exposure**: Only expose necessary configuration to the client

