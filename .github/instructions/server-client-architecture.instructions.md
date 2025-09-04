---
applyTo: '**'
---

# Server-Client Component Architecture Guidelines

## Component Structure Pattern

### Server-Client Component Separation
- Components are divided into **Server Components** and **Client Components**
- Server components: `ComponentNameSC.tsx` with component name ending in "SC"
- Client components: `ComponentName.tsx` (regular name)
- Both files live in the same folder for logical grouping

### File Organization Rules
- Reserved Next.js files (`layout.tsx`, `page.tsx`, `loading.tsx`, `error.tsx`) should NOT have SC/Client pairs
- These reserved files are server components by default unless marked with `"use client"`
- Only child components should use the SC/Client file pair pattern
- Shared code should be placed closest to where it's used:
  - Page-specific: Include in the page folder
  - Multi-page: Include in common folder at project root

## Data Loading Strategy

### Server Component Responsibilities
- **ALL initial data loading** using server-side Prisma queries
- Static content rendering that doesn't require client-side logic
- User authentication checks using Clerk's `auth()` server function
- Role-based data fetching and filtering
- Initial table data loading (first page of results)
- Profile and user information preloading

### Client Component Responsibilities
- State management and React hooks
- Interactive UI elements and event handlers
- Form submissions and data mutations
- Real-time updates (will be implemented with Upstash later)
- Dynamic user interactions

### Data Communication
- **Props only** - No React Context unless explicitly requested
- Server components pass initial data to client components via props
- Client components can fetch/mutate data only for:
  - Form submissions
  - State updates
  - Real-time data updates
  - User interactions requiring fresh data

## Authentication & Authorization

### Clerk Integration
- Use Clerk as the official authentication provider
- Server components: Use `auth()` from `@clerk/nextjs/server`
- Client components: Use `useUser()` hook when session info is needed
- Favor server component props for passing user data

### Role-Based Access Control
- **First layer**: Middleware handles initial route protection and redirects
- **Second layer**: Role-specific layout components (`/doctor/*`, `/patient/*`) verify user role
- Users without profiles should be redirected to `/register`
- Cross-role access should be prevented at the layout level

## Loading & Error Handling

### Loading States
- Use **MUI Skeleton components** for loading indicators
- Implement `loading.tsx` file for every page route
- Wrap server components in Suspense boundaries with skeleton fallbacks
- Loading states should be handled at the parent component level

### Error Handling
- Error boundaries should be placed at the parent component where Server components are used
- Configure error boundaries at the root layout of each page route
- Server component data fetch failures should be caught by error boundaries
- Client-side errors should have appropriate error states

## Performance & Caching

### Next.js App Router Benefits
- Leverage Next.js data fetch caching for performance
- Fetch data separately in each component - caching will optimize duplicate requests
- Server components provide better initial page load performance
- Reduced client-side API calls improve user experience

## Implementation Guidelines

### Initial Data Loading
- **Preload all necessary data** in server components
- Dashboard pages should load initial metrics, appointments, and user-specific data
- Table components should load the first page of results
- Profile components should load complete user profile information
- Avoid API endpoints for initial data - use direct Prisma queries

### Component Communication
- Pass data down through props from server to client components
- Avoid prop drilling by structuring components appropriately
- Keep component hierarchy flat when possible
- Use TypeScript interfaces to ensure type safety between components

### Code Organization
- Group related server and client components in the same folder
- Create shared types and utilities at appropriate levels
- Remove unused API endpoints and hooks when refactoring to server components
- Maintain clear separation of concerns between server and client logic

## Migration Strategy

### From API-based to Server Components
1. Remove API endpoints that provide initial data
2. Replace client hooks with server component data loading
3. Convert layout and page components to server components
4. Create SC/Client pairs for interactive child components
5. Update TypeScript interfaces for prop-based data flow
6. Add appropriate loading and error boundaries

### Backwards Compatibility
- Remove stale API endpoints completely during refactor
- Update imports and dependencies
- Ensure all components have proper TypeScript types
- Test role-based redirects and access control
