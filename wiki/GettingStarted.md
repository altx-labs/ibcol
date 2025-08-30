# Getting Started

This guide will help you set up the IBCOL platform for local development and understand the basic workflow.

## Prerequisites

Before you begin, ensure you have the following installed:

- **Node.js**: Version 12.x (as specified in `package.json`)
- **npm** or **yarn**: For package management
- **Git**: For version control

## Installation

1. **Clone the repository**:
   ```bash
   git clone git@github.com:altx-labs/ibcol.git
   cd ibcol
   ```

2. **Install dependencies**:
   ```bash
   npm install
   # or
   yarn install
   ```

3. **Set up environment variables**:
   
   Create a `.env` file in the root directory with the following variables:
   ```
   ENV=local
   GRAPHQL_URL=http://localhost:4000/graphql
   FILEPOND_API_URL=http://localhost:4004
   FILEPOND_API_ENDPOINT=/api/filepond/
   SALT=your-secret-salt-value
   ```

   For Google Cloud Storage integration (optional):
   ```
   GOOGLE_APPLICATION_CREDENTIALS=/tmp/gc-service-account.json
   GOOGLE_APPLICATION_CREDENTIALS_DATA=base64-encoded-service-account-json
   GCS_BUCKET_NAME=your-gcs-bucket-name
   ```

4. **Start the development server**:
   ```bash
   npm run local
   # or
   yarn local
   ```

5. **Access the application**:
   
   Open your browser and navigate to:
   ```
   http://localhost:3000
   ```

## Development Workflow

### Project Structure

The project follows this structure:

```
/
├── components/           # Reusable UI components
├── configs/              # Environment configurations
├── helpers/              # Utility functions
├── node-routes/          # Server-side route handlers
├── pages/                # Next.js page components
├── static/               # Static assets
├── styles/               # Global CSS styles
├── translations/         # Translation files
└── slack-notifications/  # Deployment notifications
```

### Key Development Tasks

#### Running the Application

```bash
# Local development
npm run local

# Development environment
npm run _dev

# Staging environment
npm run stage

# Production environment
npm run production
```

#### Building the Application

```bash
npm run build
```

#### Deploying the Application

```bash
# Deploy to development
npm run deploy-dev

# Deploy to staging
npm run deploy-stage

# Deploy to production
npm run deploy-production
```

### Working with Components

Components are organized in directories with their own index.js file:

```
components/
├── ComponentName/
│   ├── ComponentName.jsx  # Main component implementation
│   └── index.js           # Re-exports the component
```

To create a new component:

1. Create a new directory in `components/`
2. Create the component file (e.g., `MyComponent.jsx`)
3. Create an `index.js` file that exports the component
4. Import and use the component in your pages

Example:

```jsx
// components/MyComponent/MyComponent.jsx
import React from 'react';
import styled from 'styled-components';

const StyledComponent = styled.div`
  /* CSS styles here */
`;

const MyComponent = ({ children }) => (
  <StyledComponent>
    {children}
  </StyledComponent>
);

export default MyComponent;

// components/MyComponent/index.js
export { default } from './MyComponent';

// Usage in a page
import MyComponent from 'components/MyComponent';

const Page = () => (
  <MyComponent>
    Content goes here
  </MyComponent>
);
```

### Working with Pages

Pages are defined in the `pages/next/` directory:

```
pages/
├── _app.js              # Custom App component
├── _document.js         # Custom Document component
└── next/
    ├── home.js          # Home page
    ├── about.js         # About page
    ├── registration/
    │   ├── index.js     # Registration page
    │   └── login.js     # Registration login page
    └── admin/
        ├── index.js     # Admin login page
        └── dashboard.js # Admin dashboard page
```

To create a new page:

1. Create a new file in `pages/next/`
2. Define the page component
3. Add the route to `routes.js`

Example:

```jsx
// pages/next/my-page.js
import React from 'react';
import { translate } from 'helpers/translate.js';
import PageContainerComponent from 'components/PageContainerComponent';

export default class extends React.Component {
  static async getInitialProps({ req, res, query }) {
    return { query }
  }

  translate = (t) => translate(t, 'my-page', this.props.query.locale);

  render() {
    return (
      <PageContainerComponent
        title={this.translate('pageTitle')}
        description={this.translate('seoDescription')}
        locale={this.props.query.locale}
      >
        <h1>{this.translate('mainHeading')}</h1>
        <p>{this.translate('content')}</p>
      </PageContainerComponent>
    );
  }
}

// Add to routes.js
routes.add({ name: 'my-page', pattern: '/:locale/my-page/', page: 'next/my-page' })
```

### Working with Translations

To add translations for a new page:

1. Create a new JSON file in each locale directory:
   ```
   translations/
   ├── en-us/
   │   └── my-page.json
   ├── zh-hk/
   │   └── my-page.json
   └── ...
   ```

2. Add the translations to each file:
   ```json
   {
     "pageTitle": "My Page Title",
     "seoDescription": "Description for search engines",
     "mainHeading": "Welcome to My Page",
     "content": "This is the content of my page."
   }
   ```

3. Use the translations in your page:
   ```jsx
   this.translate('pageTitle')
   ```

### Working with GraphQL

To fetch data from the GraphQL API:

1. Define your query or mutation:
   ```jsx
   import gql from 'graphql-tag';

   const GET_DATA = gql`
     query GetData {
       someData {
         id
         name
       }
     }
   `;
   ```

2. Use the Query or Mutation component:
   ```jsx
   import { Query } from 'react-apollo';

   <Query query={GET_DATA}>
     {({ loading, error, data }) => {
       if (loading) return <p>Loading...</p>;
       if (error) return <p>Error: {error.message}</p>;
       
       return (
         <ul>
           {data.someData.map(item => (
             <li key={item.id}>{item.name}</li>
           ))}
         </ul>
       );
     }}
   </Query>
   ```

## Troubleshooting

### Common Issues

#### Node.js Version

If you encounter errors related to Node.js, ensure you're using version 12.x:

```bash
node --version
# Should output v12.x.x
```

If you need to switch Node.js versions, consider using [nvm](https://github.com/nvm-sh/nvm).

#### GraphQL Connection

If you can't connect to the GraphQL API:

1. Check that the API is running
2. Verify the `GRAPHQL_URL` environment variable
3. Check for CORS issues in the browser console

#### File Upload Issues

If file uploads aren't working:

1. Ensure the FilePond microservice is running:
   ```bash
   npm run local-filepond
   ```
2. Check the Google Cloud Storage credentials
3. Verify the bucket permissions

#### Styling Issues

If styles aren't applying correctly:

1. Check that styled-components is working
2. Verify that CSS files are being imported
3. Check for CSS conflicts

## Next Steps

Now that you have the application running locally, you can:

1. Explore the codebase to understand its structure
2. Make changes to components and pages
3. Add new features or fix bugs
4. Run tests to ensure everything works
5. Deploy your changes to the development environment

For more detailed information, refer to the other documentation pages:

- [Architecture Overview](./Architecture.md)
- [Component Structure](./Components.md)
- [Internationalization](./Internationalization.md)
- [Routing System](./Routing.md)
- [File Upload System](./FileUpload.md)
- [GraphQL Integration](./GraphQL.md)
- [Deployment Process](./Deployment.md)
- [Configuration Management](./Configuration.md)

