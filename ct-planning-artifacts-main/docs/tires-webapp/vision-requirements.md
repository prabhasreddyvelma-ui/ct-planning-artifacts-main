# State Management
- **Redux Toolkit**:
  - Tire Center selection
  - Vehicle selection
  - User authentication/session
  - Compare Tires
- Persist critical state in **localStorage** for session continuity

# Performance Considerations
- Server-Side Rendering (SSR) or Static Site Generation (SSG) where appropriate
- Lazy load images, components, and routes
- Use SCSS variables and modular styles to reduce CSS bloat

# Security Best Practices
- Input validation and sanitization
- HTTPS enforcement
- Secure HTTP headers
- Rate limiting for sensitive endpoints

# Scalability & Best Practices
- Modular folder structure: `components`, `pages`, `redux`, `styles`, `utils`, `services`
- Reusable UI components with SCSS and Bootstrap 5
- Centralized configuration for API endpoints and constants
- Logging and error handling across application
- Accessibility (ARIA attributes, semantic HTML, keyboard navigation)

# Testing
- Unit tests for components and Redux slices

# Additional Notes
- Ensure mobile-first responsive design
- All modals, carousels, and interactive elements accessible
- Follow code linting and formatting conventions
- Create re-usable components wherever possible (Dropdowns, Buttons, Modal)