# Component Structure

The IBCOL platform follows a component-based architecture using React. This document outlines the main components and their relationships.

## Component Hierarchy

```
App
├── PageContainerComponent
│   ├── MenuComponent / IndexMenuComponent
│   ├── Page Content (varies by route)
│   └── Footer
├── LanguageSelectorComponent
└── LoaderComponent
```

## Base Components

The application includes several base components that serve as building blocks for more complex UI elements:

| Component | Description |
|-----------|-------------|
| `Button.jsx` | Styled button component with various states |
| `H1.jsx` | Primary heading component |
| `H2.jsx` | Secondary heading component |
| `H3.jsx` | Tertiary heading component |

## Key Components

### Page Structure Components

| Component | Purpose |
|-----------|---------|
| `PageContainerComponent` | Wrapper for all pages, provides consistent layout |
| `PageSectionComponent` | Container for page sections with consistent styling |
| `SectionBlockComponent` | Block-level container within sections |
| `MenuComponent` | Main navigation menu for standard pages |
| `IndexMenuComponent` | Special navigation menu for the landing page |

### Form Components

| Component | Purpose |
|-----------|---------|
| `RegistrationFormComponent` | Complex form for team registration |
| `CountryInputSelectComponent` | Country selection dropdown with search |
| `FilePondComponent` | File upload component using FilePond library |

### Navigation Components

| Component | Purpose |
|-----------|---------|
| `NavLinkComponent` | Navigation link with active state handling |
| `ScrollButtonComponent` | Button for scrolling to different page sections |
| `LocaleSwitcherComponent` | Language selection component |

### Utility Components

| Component | Purpose |
|-----------|---------|
| `LoaderComponent` | Loading indicator for async operations |
| `SVGComponent` | SVG rendering component |
| `MailChimpComponent` | Newsletter signup integration |

## Component Patterns

### Styled Components Pattern

Most components use styled-components for styling:

```jsx
import styled from 'styled-components';

const StyledComponent = styled.div`
  /* CSS styles here */
`;

const MyComponent = ({ children }) => (
  <StyledComponent>
    {children}
  </StyledComponent>
);
```

### Container/Presentation Pattern

Components are often split into container and presentation components:

- **Container Components**: Handle data fetching, state management
- **Presentation Components**: Focus on rendering UI based on props

### Component Export Pattern

Components are typically organized with an index.js file for easier imports:

```
ComponentName/
├── ComponentName.jsx  // Main component implementation
└── index.js           // Re-exports the component
```

This allows importing with:

```jsx
import ComponentName from 'components/ComponentName';
```

## Component Diagram

Below is a diagram of the main component relationships:

```
┌─────────────────────────────────────────────────────────────┐
│ _app.js (MyApp)                                             │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ ApolloProvider                                      │    │
│  │                                                     │    │
│  │  ┌─────────────────────────────────────────────┐   │    │
│  │  │ StickyContainer                             │   │    │
│  │  │                                             │   │    │
│  │  │  ┌─────────────────────────────────────┐   │   │    │
│  │  │  │ MenuComponent / IndexMenuComponent  │   │   │    │
│  │  │  └─────────────────────────────────────┘   │   │    │
│  │  │                                             │   │    │
│  │  │  ┌─────────────────────────────────────┐   │   │    │
│  │  │  │ Page Component (e.g., home.js)      │   │   │    │
│  │  │  │                                     │   │   │    │
│  │  │  │  ┌─────────────────────────────┐   │   │   │    │
│  │  │  │  │ PageContainerComponent      │   │   │   │    │
│  │  │  │  │                             │   │   │   │    │
│  │  │  │  │  ┌─────────────────────┐   │   │   │   │    │
│  │  │  │  │  │ Page-specific       │   │   │   │   │    │
│  │  │  │  │  │ Components          │   │   │   │   │    │
│  │  │  │  │  └─────────────────────┘   │   │   │   │    │
│  │  │  │  │                             │   │   │   │    │
│  │  │  │  └─────────────────────────────┘   │   │   │    │
│  │  │  │                                     │   │   │    │
│  │  │  └─────────────────────────────────────┘   │   │    │
│  │  │                                             │   │    │
│  │  │  ┌─────────────────────────────────────┐   │   │    │
│  │  │  │ Footer                              │   │   │    │
│  │  │  └─────────────────────────────────────┘   │   │    │
│  │  │                                             │   │    │
│  │  └─────────────────────────────────────────────┘   │    │
│  │                                                     │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Component Communication

Components communicate through several mechanisms:

1. **Props**: Parent-to-child data flow
2. **Context**: For global state like locale
3. **Apollo Client**: For GraphQL data fetching
4. **Event Handlers**: For user interactions

## Reusable Component Examples

### Button Component

```jsx
// components/BaseComponents/Button.jsx
import styled from 'styled-components';

const Button = styled.button`
  /* Button styles */
`;

export default Button;
```

### Registration Form Component

The `RegistrationFormComponent` is one of the most complex components in the application:

- Handles form state management
- Validates user input
- Manages file uploads
- Submits data to GraphQL API
- Shows success/error messages

Key features:

- Dynamic form fields for team members and projects
- File upload integration with Google Cloud Storage
- Form validation with error messages
- GraphQL mutation for form submission

## Best Practices

The component structure follows these best practices:

1. **Single Responsibility**: Each component has a clear, focused purpose
2. **Reusability**: Components are designed to be reused across the application
3. **Composition**: Complex UIs are built by composing smaller components
4. **Separation of Concerns**: Styling, logic, and markup are separated
5. **Consistent Naming**: Components follow a consistent naming convention

