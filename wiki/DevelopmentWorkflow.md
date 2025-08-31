# Development Workflow

This document outlines the recommended development workflow for the IBCOL platform, including branching strategy, code standards, and review process.

## Development Lifecycle

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│  Feature Branch │────▶│  Dev Branch     │────▶│  Master Branch  │
│  (feature/*)    │     │  (dev)          │     │  (master)       │
│                 │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
        │                        │                       │
        │                        │                       │
        ▼                        ▼                       ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│  Local Testing  │────▶│  UAT Testing    │────▶│  Production     │
│                 │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

## Branching Strategy

The project follows a simplified Git Flow branching strategy:

1. **master**: Production branch, always stable and deployable
2. **dev**: Development branch, integration branch for features
3. **feature/\***: Feature branches for new development

### Branch Naming Conventions

- **Feature branches**: `feature/feature-name`
- **Bug fix branches**: `bugfix/issue-description`
- **Hotfix branches**: `hotfix/issue-description`

## Development Process

### Starting a New Feature

1. **Create a feature branch from dev**:
   ```bash
   git checkout dev
   git pull origin dev
   git checkout -b feature/my-new-feature
   ```

2. **Implement the feature**:
   - Write code following the project's coding standards
   - Add or update translations as needed
   - Test locally using `npm run local`

3. **Commit changes**:
   ```bash
   git add .
   git commit -m "feat: add my new feature"
   ```

4. **Push to remote**:
   ```bash
   git push origin feature/my-new-feature
   ```

5. **Create a pull request**:
   - Create a PR from your feature branch to `dev`
   - Add a description of the changes
   - Request a code review

### Code Review Process

1. **Reviewer checks**:
   - Code follows project standards
   - Functionality works as expected
   - No regression issues
   - Translations are complete
   - Performance considerations

2. **Address feedback**:
   - Make requested changes
   - Push additional commits
   - Request re-review if needed

3. **Merge to dev**:
   - Once approved, merge the PR to `dev`
   - Delete the feature branch after merging

### Deployment to Staging

1. **Automated deployment**:
   - Travis CI automatically deploys the `dev` branch to staging
   - The application is available at `https://uat.ibcol.org`

2. **UAT testing**:
   - Test the feature in the staging environment
   - Verify functionality across different browsers
   - Check mobile responsiveness

### Deployment to Production

1. **Create a PR from dev to master**:
   - Create a PR when ready to release to production
   - Include a summary of all changes

2. **Final review**:
   - Conduct a final review of the changes
   - Verify all tests pass

3. **Merge to master**:
   - Once approved, merge the PR to `master`
   - Travis CI automatically deploys to production
   - The application is available at `https://ibcol.org`

## Commit Message Conventions

The project follows the [Conventional Commits](https://www.conventionalcommits.org/) specification:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Types

- **feat**: A new feature
- **fix**: A bug fix
- **docs**: Documentation changes
- **style**: Changes that don't affect the code's meaning (formatting, etc.)
- **refactor**: Code changes that neither fix a bug nor add a feature
- **perf**: Performance improvements
- **test**: Adding or correcting tests
- **chore**: Changes to the build process or auxiliary tools

### Examples

```
feat(registration): add file upload for student ID cards

- Added FilePond component for file uploads
- Integrated with Google Cloud Storage
- Added validation for file types and sizes

Closes #123
```

```
fix(i18n): correct translation key in Chinese locale

The 'submit' button text was missing in the zh-hk locale.
```

## Testing

### Local Testing

1. **Run the local development server**:
   ```bash
   npm run local
   ```

2. **Test in different browsers**:
   - Chrome
   - Firefox
   - Safari
   - Edge

3. **Test responsive design**:
   - Desktop
   - Tablet
   - Mobile

### Automated Testing

Currently, the project does not have automated tests. This is an area for future improvement.

## Code Standards

### JavaScript/React

- Use ES6+ features
- Follow React best practices
- Use PropTypes for component props
- Use destructuring for props and state
- Keep components focused and reusable

### Styling

- Use styled-components for component styling
- Follow the existing style patterns
- Use the media query helpers for responsive design
- Avoid inline styles

### File Organization

- Keep related files together
- Use index.js files for cleaner imports
- Follow the established project structure

## Internationalization

- Always use the translation system for text
- Add translations for all supported languages
- Test with different languages

## Performance Considerations

- Optimize images and assets
- Use React.PureComponent or React.memo when appropriate
- Avoid unnecessary re-renders
- Lazy load components when possible

## Security Best Practices

- Validate all user inputs
- Sanitize data before rendering
- Use HTTPS for all API calls
- Don't store sensitive information in client-side code

## Troubleshooting

### Common Development Issues

#### Hot Reloading Not Working

If hot reloading stops working:

1. Stop the development server
2. Clear the Next.js cache:
   ```bash
   rm -rf .next
   ```
3. Restart the server:
   ```bash
   npm run local
   ```

#### GraphQL Connection Issues

If you can't connect to the GraphQL API:

1. Check that the API is running
2. Verify the `GRAPHQL_URL` environment variable
3. Check for CORS issues in the browser console

#### Styling Issues

If styles aren't applying correctly:

1. Check that styled-components is working
2. Verify that CSS files are being imported
3. Check for CSS conflicts

## Continuous Integration

The project uses Travis CI for continuous integration:

1. **Build**: Builds the application
2. **Deploy**: Deploys to the appropriate environment based on the branch

## Release Process

### Release Checklist

Before releasing to production:

1. **Verify all features**:
   - Test all new features in staging
   - Check for regression issues

2. **Review translations**:
   - Ensure all translations are complete
   - Test with different languages

3. **Performance check**:
   - Run Lighthouse audits
   - Check page load times

4. **Security review**:
   - Review for security issues
   - Check for exposed secrets

5. **Documentation**:
   - Update documentation if needed
   - Document any API changes

### Creating a Release

1. **Create a release PR**:
   - Create a PR from `dev` to `master`
   - Include a summary of all changes

2. **Tag the release**:
   ```bash
   git checkout master
   git pull origin master
   git tag v1.2.3
   git push origin v1.2.3
   ```

3. **Monitor the deployment**:
   - Watch the Travis CI build
   - Check Slack notifications
   - Verify the production site

## Hotfix Process

For critical issues in production:

1. **Create a hotfix branch from master**:
   ```bash
   git checkout master
   git pull origin master
   git checkout -b hotfix/critical-issue
   ```

2. **Fix the issue**:
   - Make the minimal changes needed
   - Test thoroughly

3. **Create a PR to master**:
   - Create a PR from the hotfix branch to `master`
   - Get an expedited review

4. **Merge to master**:
   - Once approved, merge to `master`
   - Travis CI will deploy to production

5. **Backport to dev**:
   - Create a PR from the hotfix branch to `dev`
   - Merge to keep `dev` up to date

