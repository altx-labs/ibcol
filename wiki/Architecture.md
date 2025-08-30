# Architecture Overview

This document provides a high-level overview of the IBCOL platform architecture.

## System Architecture

The IBCOL platform follows a modern web application architecture with the following key components:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│  Client Browser │────▶│  Next.js Server │────▶│  GraphQL API   │
│                 │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                                │                        │
                                │                        │
                                ▼                        ▼
                        ┌─────────────────┐     ┌─────────────────┐
                        │                 │     │                 │
                        │  Google Cloud   │     │  Database       │
                        │  Storage        │     │  (MongoDB)      │
                        │                 │     │                 │
                        └─────────────────┘     └─────────────────┘
```

### Key Components

1. **Client Browser**: The user interface rendered in the browser, built with React components.

2. **Next.js Server**: Server-side rendering and API routes, handling:
   - Page rendering
   - Routing
   - API endpoints for file uploads
   - Authentication

3. **GraphQL API**: External microservice ([ibcol-micro-graphql-api](https://github.com/altx-labs/ibcol-micro-graphql-api)) that provides:
   - Data persistence
   - User authentication
   - Application management
   - Competition data

4. **Google Cloud Storage**: Cloud storage for user-uploaded files:
   - Whitepapers
   - Presentation files
   - Student ID cards
   - Transcripts

5. **Database**: MongoDB database (managed by the GraphQL API service) storing:
   - User accounts
   - Team registrations
   - Project submissions
   - Competition data

## Application Flow

### Page Rendering Flow

1. User requests a page (e.g., `/en-us/registration`)
2. Next.js server renders the page with initial data
3. React components are hydrated on the client
4. Apollo Client manages subsequent data fetching

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│          │     │          │     │          │     │          │
│  Request │────▶│  Next.js │────▶│  React   │────▶│  Apollo  │
│          │     │  Server  │     │  Hydrate │     │  Client  │
│          │     │          │     │          │     │          │
└──────────┘     └──────────┘     └──────────┘     └──────────┘
```

### Registration Flow

1. User fills out registration form
2. Files are uploaded to Google Cloud Storage via FilePond
3. Form data and file references are submitted to GraphQL API
4. Confirmation email is sent to the user
5. User can log in to view/edit their submission

```
┌──────────┐     ┌──────────┐     ┌──────────┐
│          │     │          │     │          │
│  Form    │────▶│  File    │────▶│  GraphQL │
│  Data    │     │  Upload  │     │  Submit  │
│          │     │          │     │          │
└──────────┘     └──────────┘     └──────────┘
      │                                 │
      │                                 │
      ▼                                 ▼
┌──────────┐                     ┌──────────┐
│          │                     │          │
│  GCS     │                     │  Email   │
│  Storage │                     │  Confirm │
│          │                     │          │
└──────────┘                     └──────────┘
```

## Code Architecture

The codebase follows a component-based architecture with the following principles:

1. **Component Reusability**: UI elements are built as reusable components
2. **Container/Presentation Pattern**: Separation of data fetching and presentation
3. **Server-Side Rendering**: Initial page load is rendered on the server
4. **Client-Side Navigation**: Subsequent navigation happens on the client
5. **Internationalization**: All text is externalized in translation files

### Directory Structure

```
/
├── components/           # Reusable UI components
├── pages/                # Next.js page components
├── helpers/              # Utility functions
├── translations/         # Translation files
├── configs/              # Environment configurations
├── static/               # Static assets
├── styles/               # Global CSS
├── node-routes/          # Server-side routes
└── slack-notifications/  # Deployment notifications
```

## Deployment Architecture

The application is deployed on Zeit Now (Vercel) as a serverless application:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│  Git Repository │────▶│  Travis CI      │────▶│  Zeit Now      │
│                 │     │                 │     │  (Vercel)       │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                                                        │
                                                        │
                                                        ▼
                                                ┌─────────────────┐
                                                │                 │
                                                │  CDN Edge       │
                                                │  Network        │
                                                │                 │
                                                └─────────────────┘
```

1. Code is pushed to the Git repository
2. Travis CI builds and tests the application
3. On successful build, the application is deployed to Zeit Now
4. Zeit Now distributes the application to its global CDN
5. Slack notifications are sent on successful deployment

## Security Architecture

The application implements several security measures:

1. **Authentication**: Token-based authentication for user sessions
2. **File Validation**: File type and size validation before upload
3. **CORS**: Cross-Origin Resource Sharing restrictions
4. **Content Security Policy**: Restrictions on script execution
5. **HTTPS**: All traffic is encrypted using TLS
6. **Input Validation**: Form inputs are validated on both client and server

## Performance Optimization

The application includes several performance optimizations:

1. **Server-Side Rendering**: Faster initial page loads
2. **Code Splitting**: Only load JavaScript needed for each page
3. **Image Optimization**: Optimized images for faster loading
4. **Caching**: Browser and CDN caching for static assets
5. **Lazy Loading**: Components and translations are loaded on demand

