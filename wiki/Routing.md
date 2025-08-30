# Routing System

The IBCOL platform uses a custom routing system built on top of Next.js and the `next-routes` package. This document explains how routing works and how to manage routes.

## Routing Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│  routes.js      │────▶│  Next.js Router │────▶│  Page Component │
│  Configuration  │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

The routing system consists of:

1. **Route Definitions**: Defined in `routes.js`
2. **Link Component**: Custom `Link` component for client-side navigation
3. **Router Object**: For programmatic navigation
4. **Hash-Aware Routing**: Support for URL hash fragments

## Route Definitions

Routes are defined in `routes.js` using the `next-routes` package:

```javascript
const routes = require('next-routes')({
  Link: HashAwareLink,
  Router: HashAwareRouter,
})

routes
  .add({ name: 'home', pattern: '/:locale/home', page: 'next/home' })
  .add({ name: 'landing', pattern: '/:locale/', page: 'next/landing' })
  .add({ name: '2018', pattern: '/:locale/ibcol/2018/', page: 'next/ibcol/2018' })
  .add({ name: '2019', pattern: '/:locale/ibcol/2019/', page: 'next/ibcol/2019' })
  .add({ name: 'program', pattern: '/:locale/program/', page: 'next/program' })
  .add({ name: 'rules', pattern: '/:locale/rules/', page: 'next/rules' })
  .add({ name: 'contact', pattern: '/:locale/contact/', page: 'next/contact' })
  .add({ name: 'registration', pattern: '/:locale/registration/', page: 'next/registration' })
  .add({ name: 'registrationLogin', pattern: '/:locale/registration/login/', page: 'next/registration/login' })
  .add({ name: 'registrationVerification', pattern: '/:locale/registration/verify/:verificationCode/:email/', page: 'next/registration/verify' })
  .add({ name: 'adminLogin', pattern: '/:locale/admin/', page: 'next/admin' })
  .add({ name: 'adminVerification', pattern: '/:locale/admin/verify/:verificationCode/:email/', page: 'next/admin/verify' })
  .add({ name: 'adminDashboard', pattern: '/:locale/admin/dashboard/', page: 'next/admin/dashboard' })
  .add({ name: 'join-us', pattern: '/:locale/join-us/', page: 'next/join-us' })
```

Each route has:

- **name**: Unique identifier for the route
- **pattern**: URL pattern with parameters (e.g., `:locale`)
- **page**: Path to the Next.js page component

## Locale-Based Routing

All routes include a `:locale` parameter that specifies the language:

- `/en-us/home` - English (US) home page
- `/zh-hk/home` - Chinese (Hong Kong) home page

This allows the application to:

1. Load the correct translations
2. Maintain language selection during navigation
3. Support deep linking to specific pages in specific languages

## Using the Link Component

The custom `Link` component is used for client-side navigation:

```jsx
import { Link } from '/routes';

// In a component:
<Link route="home" params={{ locale: 'en-us' }}>
  <a>Home</a>
</Link>
```

The `Link` component:

1. Handles client-side navigation
2. Preserves the locale parameter
3. Supports URL hash fragments

## Programmatic Navigation

For programmatic navigation, use the `Router` object:

```jsx
import { Router } from '/routes';

// Navigate to a route:
Router.pushRoute('home', { locale: 'en-us' });

// Navigate with hash:
Router.pushRoute('home', { locale: 'en-us', hash: 'section1' });
```

## Hash-Aware Routing

The routing system includes custom hash-aware components:

```javascript
const HashAwareRouter = ['push', 'replace', 'prefetch'].reduce((result, key) => {
  result[key] = (href, as = href, rest = {}) => {
    const updatedPaths = addHash(href, as, rest.hash)
    return NextRouter[key](updatedPaths.href, updatedPaths.as, rest)
  }
  result.nextRouter = NextRouter
  return result
}, {})

const HashAwareLink = (props) => {
  const newProps = Object.assign({}, props, addHash(props.href, props.as, props.hash))
  delete newProps.hash
  return React.createElement(NextLink, newProps)
}
```

This allows:

1. Adding hash fragments to URLs
2. Scrolling to specific sections on navigation
3. Deep linking to specific sections

## Route Parameters

Routes can include parameters in their patterns:

- **:locale**: Language code (required in all routes)
- **:verificationCode**: For email verification
- **:email**: User email for verification

Parameters are accessed in page components via `props.query`:

```jsx
class VerifyPage extends React.Component {
  static async getInitialProps({ query }) {
    return { query }
  }
  
  render() {
    const { locale, verificationCode, email } = this.props.query;
    // ...
  }
}
```

## Server-Side Routing

For server-side routing, the application uses:

1. **Next.js API Routes**: For API endpoints
2. **Micro**: For standalone microservices
3. **Express Middleware**: For custom server logic

Example of a server-side route handler:

```javascript
// node-routes/filepondRoute.js
const { json } = require('micro');
const cors = require('micro-cors')();

module.exports = cors(async (req, res) => {
  const body = await json(req);
  
  // Handle the request...
  
  return { success: true };
});
```

## Route Guards

The application implements route guards for protected routes:

1. **Admin Routes**: Require admin authentication
2. **User Routes**: Require user authentication

Example of a route guard in a page component:

```jsx
class AdminDashboard extends React.Component {
  static async getInitialProps({ req, res, query }) {
    // Check if user is authenticated
    if (!isAuthenticated(req)) {
      // Redirect to login page if not authenticated
      if (res) {
        res.writeHead(302, { Location: `/admin?locale=${query.locale}` });
        res.end();
      } else {
        Router.pushRoute('adminLogin', { locale: query.locale });
      }
      return {};
    }
    
    return { query };
  }
  
  // ...
}
```

## URL Structure

The application follows a consistent URL structure:

```
/:locale/:page/[additional-parameters]/
```

Examples:

- `/en-us/home/` - Home page in English
- `/zh-hk/registration/` - Registration page in Chinese
- `/en-us/registration/verify/ABC123/user@example.com/` - Verification page with parameters

## Adding New Routes

To add a new route:

1. Add the route definition to `routes.js`
2. Create the corresponding page component in `/pages/next/`
3. Add translations for the new page
4. Update navigation components to include the new route

Example of adding a new route:

```javascript
// In routes.js
routes.add({ name: 'newPage', pattern: '/:locale/new-page/', page: 'next/new-page' })

// Create /pages/next/new-page.js
// Add translations in /translations/*/new-page.json
```

## Best Practices

1. **Always Include Locale**: All routes should include the `:locale` parameter
2. **Use Named Routes**: Always use route names instead of hardcoded URLs
3. **Consistent Patterns**: Follow the established URL pattern structure
4. **Trailing Slashes**: Include trailing slashes in route patterns
5. **Descriptive Names**: Use clear, descriptive route names

