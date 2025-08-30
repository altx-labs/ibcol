# IBCOL Documentation

Welcome to the International Blockchain Olympiad (IBCOL) documentation. This wiki provides comprehensive information about the codebase, architecture, and development processes.

## Table of Contents

1. [Architecture Overview](./Architecture.md)
2. [Component Structure](./Components.md)
3. [Internationalization](./Internationalization.md)
4. [Routing System](./Routing.md)
5. [File Upload System](./FileUpload.md)
6. [GraphQL Integration](./GraphQL.md)
7. [Deployment Process](./Deployment.md)
8. [Configuration Management](./Configuration.md)

## Quick Links

- [Getting Started](./GettingStarted.md)
- [Development Workflow](./DevelopmentWorkflow.md)
- [Troubleshooting](./Troubleshooting.md)

## Project Overview

The International Blockchain Olympiad (IBCOL) is a multidisciplinary design and building competition focused on blockchain technology. The platform serves as both the public-facing website for the competition and the registration/submission system for participants.

### Key Features

- **Multi-language Support**: The platform supports over 20 languages with a comprehensive translation system.
- **Registration System**: Teams can register, add members, and submit project details.
- **File Upload**: Secure file upload system for whitepapers and presentations using Google Cloud Storage.
- **Admin Dashboard**: Administrative interface for managing applications and competition data.
- **Responsive Design**: Mobile-friendly interface that works across all devices.

### Target Audience

1. **Students**: University students participating in the competition
2. **Advisors**: Faculty members supporting student teams
3. **Organizers**: Competition administrators and judges
4. **General Public**: Visitors interested in learning about the competition

## Technology Stack

- **Frontend Framework**: React with Next.js for server-side rendering
- **Styling**: Styled Components for component-based styling
- **State Management**: Apollo Client for GraphQL data fetching
- **Routing**: Custom routing with next-routes
- **Deployment**: Zeit Now (Vercel) for serverless deployment
- **CI/CD**: Travis CI for continuous integration and deployment
- **Storage**: Google Cloud Storage for file uploads
- **Authentication**: Custom token-based authentication system

## Repository Structure

The repository follows a component-based architecture with the following main directories:

- `/components`: Reusable UI components
- `/pages`: Next.js page components
- `/helpers`: Utility functions and helpers
- `/translations`: Translation files for all supported languages
- `/configs`: Environment-specific configuration files
- `/static`: Static assets like images and fonts
- `/styles`: Global CSS styles
- `/node-routes`: Server-side route handlers

For more detailed information about each aspect of the codebase, please refer to the specific documentation pages linked in the Table of Contents.

