# Troubleshooting

This guide provides solutions for common issues you might encounter when working with the IBCOL platform.

## Development Environment Issues

### Node.js Version Mismatch

**Problem**: Errors related to Node.js version compatibility.

**Solution**:
1. Check your Node.js version:
   ```bash
   node --version
   ```
2. Ensure you're using Node.js 12.x as specified in `package.json`.
3. If needed, install and use the correct version with nvm:
   ```bash
   nvm install 12
   nvm use 12
   ```

### Package Installation Failures

**Problem**: `npm install` or `yarn install` fails with errors.

**Solution**:
1. Clear npm cache:
   ```bash
   npm cache clean --force
   ```
2. Delete node_modules and package-lock.json:
   ```bash
   rm -rf node_modules package-lock.json
   ```
3. Reinstall dependencies:
   ```bash
   npm install
   ```

### Development Server Won't Start

**Problem**: `npm run local` fails to start the development server.

**Solution**:
1. Check for port conflicts:
   ```bash
   lsof -i :3000
   ```
2. Kill any processes using the port:
   ```bash
   kill -9 <PID>
   ```
3. Check environment variables in `.env` file.
4. Clear Next.js cache:
   ```bash
   rm -rf .next
   ```
5. Restart the server:
   ```bash
   npm run local
   ```

## Application Issues

### Hot Reloading Not Working

**Problem**: Changes to code don't trigger hot reloading.

**Solution**:
1. Stop the development server.
2. Clear the Next.js cache:
   ```bash
   rm -rf .next
   ```
3. Restart the server:
   ```bash
   npm run local
   ```

### Styling Issues

**Problem**: Styles aren't applying correctly or styled-components not working.

**Solution**:
1. Check that styled-components is properly imported:
   ```jsx
   import styled from 'styled-components';
   ```
2. Verify that global styles are loaded in `_app.js`.
3. Check for CSS conflicts in browser dev tools.
4. Ensure CSS files are being imported correctly.

### Routing Issues

**Problem**: Links not working or pages not rendering correctly.

**Solution**:
1. Check route definitions in `routes.js`.
2. Ensure you're using the `Link` component from `/routes`:
   ```jsx
   import { Link } from '/routes';
   ```
3. Verify locale parameter is included in all routes.
4. Check for console errors related to routing.

### GraphQL Connection Issues

**Problem**: Can't connect to the GraphQL API.

**Solution**:
1. Verify the GraphQL API is running.
2. Check the `GRAPHQL_URL` environment variable.
3. Look for CORS issues in the browser console.
4. Test the GraphQL endpoint directly:
   ```bash
   curl -X POST -H "Content-Type: application/json" --data '{"query": "{ __schema { types { name } } }"}' http://localhost:4000/graphql
   ```

### Authentication Issues

**Problem**: Login not working or authentication errors.

**Solution**:
1. Check browser localStorage for token:
   ```javascript
   // In browser console
   localStorage.getItem('token')
   ```
2. Verify token format and expiration.
3. Check authentication headers in GraphQL requests.
4. Clear localStorage and try logging in again:
   ```javascript
   localStorage.clear()
   ```

## File Upload Issues

### FilePond Not Working

**Problem**: File uploads with FilePond fail.

**Solution**:
1. Ensure the FilePond microservice is running:
   ```bash
   npm run local-filepond
   ```
2. Check environment variables:
   ```
   FILEPOND_API_URL
   FILEPOND_API_ENDPOINT
   ```
3. Verify Google Cloud Storage credentials.
4. Check browser console for CORS errors.
5. Verify file size and type restrictions.

### Google Cloud Storage Issues

**Problem**: Files not uploading to Google Cloud Storage.

**Solution**:
1. Check Google Cloud credentials:
   ```
   GOOGLE_APPLICATION_CREDENTIALS
   GOOGLE_APPLICATION_CREDENTIALS_DATA
   ```
2. Verify bucket permissions:
   ```bash
   gsutil ls -L gs://your-bucket-name
   ```
3. Check bucket name in environment variables:
   ```
   GCS_BUCKET_NAME
   ```
4. Test direct upload to Google Cloud Storage:
   ```bash
   gsutil cp test-file.txt gs://your-bucket-name/
   ```

## Internationalization Issues

### Missing Translations

**Problem**: Text appears as translation keys instead of translated text.

**Solution**:
1. Check if the translation key exists in the appropriate JSON file.
2. Verify the locale parameter in the URL.
3. Check the translation helper usage:
   ```jsx
   translate('keyName', 'namespace', locale)
   ```
4. Ensure the locale is supported in `translations/index.js`.

### Language Switching Issues

**Problem**: Language switching not working correctly.

**Solution**:
1. Check the `LanguageSelectorComponent` implementation.
2. Verify URL structure includes locale parameter.
3. Check for console errors during language switching.
4. Ensure all routes include the locale parameter.

## Deployment Issues

### Now (Vercel) Deployment Failures

**Problem**: Deployment to Now (Vercel) fails.

**Solution**:
1. Check the Now CLI version:
   ```bash
   now --version
   ```
2. Verify Now configuration files:
   ```
   now-dev.json
   now-stage.json
   now-production.json
   ```
3. Check for build errors in the Now logs:
   ```bash
   now logs your-deployment-url
   ```
4. Verify environment variables are set correctly.

### Travis CI Build Failures

**Problem**: Travis CI builds fail.

**Solution**:
1. Check the `.travis.yml` configuration.
2. Verify Node.js version in Travis configuration.
3. Check for linting or build errors in the Travis logs.
4. Ensure all dependencies are properly installed.
5. Verify environment variables are set in Travis settings.

## Performance Issues

### Slow Page Loading

**Problem**: Pages load slowly.

**Solution**:
1. Check network requests in browser dev tools.
2. Optimize image sizes and formats.
3. Implement code splitting for large components:
   ```jsx
   import dynamic from 'next/dynamic';
   
   const DynamicComponent = dynamic(() => import('../components/HeavyComponent'));
   ```
4. Use React.memo or PureComponent for frequently re-rendered components.
5. Analyze bundle size:
   ```bash
   npm run build
   # Check .next/stats.json
   ```

### Memory Leaks

**Problem**: Application memory usage grows over time.

**Solution**:
1. Check for event listeners not being removed in `componentWillUnmount`.
2. Look for subscriptions not being unsubscribed.
3. Use React DevTools to profile component renders.
4. Check for large objects being stored in state or context.

## Browser Compatibility Issues

### Cross-Browser Issues

**Problem**: Application works in some browsers but not others.

**Solution**:
1. Check for browser-specific CSS issues.
2. Verify polyfills are included for older browsers.
3. Test in multiple browsers:
   - Chrome
   - Firefox
   - Safari
   - Edge
4. Use feature detection instead of browser detection.

### Mobile Responsiveness Issues

**Problem**: Application doesn't display correctly on mobile devices.

**Solution**:
1. Use browser dev tools to simulate mobile devices.
2. Check media queries in styled-components:
   ```jsx
   const ResponsiveComponent = styled.div`
     width: 100%;
     
     ${media.smallDown`
       width: 90%;
     `}
   `;
   ```
3. Test on actual mobile devices.
4. Ensure viewport meta tag is set correctly in `_document.js`.

## Common Error Messages

### "Module not found"

**Problem**: `Error: Cannot find module 'some-module'`

**Solution**:
1. Check if the module is installed:
   ```bash
   npm list some-module
   ```
2. Install the missing module:
   ```bash
   npm install some-module
   ```
3. Check import paths for typos.
4. Verify module name casing (case-sensitive in some environments).

### "Unexpected token"

**Problem**: `SyntaxError: Unexpected token`

**Solution**:
1. Check for syntax errors in the indicated file.
2. Verify Babel configuration for proper transpilation.
3. Check for unsupported JavaScript features.
4. Ensure all JSX is properly closed and formatted.

### "Cannot read property of undefined"

**Problem**: `TypeError: Cannot read property 'x' of undefined`

**Solution**:
1. Use optional chaining for potentially undefined values:
   ```jsx
   const value = obj?.prop?.nestedProp
   ```
2. Add null checks before accessing properties.
3. Provide default values:
   ```jsx
   const { prop = 'default' } = obj || {}
   ```
4. Check component props and state initialization.

## Debugging Techniques

### React DevTools

Use React DevTools to inspect component hierarchy, props, and state:

1. Install the [React DevTools browser extension](https://reactjs.org/blog/2019/08/15/new-react-devtools.html).
2. Open browser dev tools and navigate to the "Components" or "React" tab.
3. Inspect component props, state, and hooks.
4. Use the profiler to identify performance issues.

### Network Debugging

Use browser dev tools to debug network requests:

1. Open browser dev tools and navigate to the "Network" tab.
2. Filter for XHR/Fetch requests.
3. Inspect request and response headers, payloads, and status codes.
4. Check for CORS issues and authentication problems.

### Console Logging

Strategic console logging can help identify issues:

```jsx
console.log('Component rendered:', { props, state });
console.log('Before API call:', data);
console.log('After API call:', response);
console.error('Error occurred:', error);
```

### Source Maps

Enable source maps for better debugging:

```javascript
// next.config.js
module.exports = {
  webpack: (config, { dev, isServer }) => {
    if (dev) {
      config.devtool = 'eval-source-map';
    }
    return config;
  }
};
```

## Getting Help

If you're still experiencing issues:

1. **Check existing documentation**: Review all wiki pages for relevant information.
2. **Search issue tracker**: Check if the issue has been reported before.
3. **Ask team members**: Reach out to other developers who have worked on the project.
4. **Create a detailed report**: If creating a new issue, include:
   - Steps to reproduce
   - Expected behavior
   - Actual behavior
   - Environment details (browser, OS, Node.js version)
   - Screenshots or error logs

