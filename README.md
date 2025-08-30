# IBCOL - International Blockchain Olympiad

[![Build Status](https://travis-ci.com/ibcol/ibcol.svg?branch=master)](https://travis-ci.com/ibcol/ibcol)

The International Blockchain Olympiad (IBCOL) is a multidisciplinary design and building competition focused on blockchain technology. This repository contains the official website and registration platform for the competition.

## Overview

IBCOL invites students from around the world to solve real-world challenges through decentralized applications. Unlike typical hackathons, IBCOL provides:

- Sufficient time to design complete solutions
- A multidisciplinary approach to blockchain applications
- Opportunities for matching with investors and incubators

## Environments

| Environment | URL | Status |
|-------------|-----|--------|
| Production | [https://www.ibcol.org/](https://www.ibcol.org/) | [![Build Status](https://travis-ci.com/ibcol/ibcol.svg?branch=master)](https://travis-ci.com/ibcol/ibcol) |
| UAT | [https://www.uat.ibcol.org/](https://www.uat.ibcol.org/) | [![Build Status](https://travis-ci.com/ibcol/ibcol.svg?branch=dev)](https://travis-ci.com/ibcol/ibcol) |

## Technology Stack

- **Frontend**: React, Next.js, Styled Components
- **State Management**: Apollo Client (GraphQL)
- **Deployment**: Zeit Now (Vercel)
- **CI/CD**: Travis CI
- **File Storage**: Google Cloud Storage
- **Internationalization**: Custom translation system with support for multiple languages

## Key Features

- Multi-language support with 20+ languages
- Team registration and project submission system
- Whitepaper and presentation file uploads
- Admin dashboard for application management
- Responsive design for all devices

## Documentation

For detailed documentation about the codebase, architecture, and development processes, please refer to the [Wiki](./wiki/README.md).

## Getting Started

### Prerequisites

- Node.js 12.x
- npm or yarn

### Installation

1. Clone the repository:
   ```bash
   git clone git@github.com:altx-labs/ibcol.git
   cd ibcol
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Set up environment variables:
   - Create a `.env` file based on the configuration examples in the `configs` directory
   - For Google Cloud Storage integration, set up the appropriate credentials

4. Run the development server:
   ```bash
   npm run local
   ```

### Deployment

The application is deployed using Zeit Now (Vercel). Different deployment environments are configured in the respective `now-*.json` files:

- `now-local.json` - Local development
- `now-dev.json` - Development environment
- `now-stage.json` - Staging environment
- `now-production.json` - Production environment

To deploy to a specific environment:

```bash
npm run deploy-dev     # Deploy to development
npm run deploy-stage   # Deploy to staging
npm run deploy-production  # Deploy to production
```

## Related Projects

- [ibcol-micro-graphql-api](https://github.com/altx-labs/ibcol-micro-graphql-api) - GraphQL API for IBCOL
- [ibcol-towww](https://github.com/altx-labs/ibcol-towww) - Redirect service for IBCOL

## License

This project is proprietary and not open for public use or modification without explicit permission.

## Contributors

- [William Li](https://github.com/wiiiimm) - Lead Developer
- Breaking Bad Interactive (BBI) - Development Team

