# Contributing to aws-next-edge-bedrock-repl

👍 First off, thanks for taking the time to contribute! 🎉

## Code of Conduct

This project and everyone participating in it is governed by our [Code of Conduct](CODE_OF_CONDUCT.md). By participating, you are expected to uphold this code.

## Development Process

1. **Fork & Clone**
   ```bash
   git clone https://github.com/BjornMelin/aws-next-edge-bedrock-repl.git
   cd aws-next-edge-bedrock-repl
   ```

2. **Set Up Development Environment**
   ```bash
   # Install dependencies
   npm install

   # Copy environment variables
   cp .env.example .env.local

   # Run development server
   npm run dev
   ```

3. **Create Feature Branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```

## Pull Request Process

1. **Before Submitting a PR**
   - Update documentation
   - Add/update tests
   - Run linting and type checking
   ```bash
   npm run lint
   npm run type-check
   ```
   - Ensure tests pass
   ```bash
   npm run test
   ```

2. **PR Guidelines**
   - Clear, descriptive title
   - Detailed description of changes
   - Reference any related issues
   - Update CHANGELOG.md
   - Include screenshots for UI changes

3. **PR Template**
   ```markdown
   ## Description
   [Describe your changes]

   ## Type of Change
   - [ ] Bug fix
   - [ ] New feature
   - [ ] Breaking change
   - [ ] Documentation update

   ## Testing
   [Describe testing steps]

   ## Screenshots
   [If applicable]

   ## Checklist
   - [ ] Tests added/updated
   - [ ] Documentation updated
   - [ ] Types updated
   - [ ] Code follows style guidelines
   ```

## Development Guidelines

### Code Style

1. **TypeScript**
   - Use strict mode
   - Proper type definitions
   - No any types
   - Document complex types

2. **React**
   - Functional components
   - Hooks for state/effects
   - Proper prop typing
   - Avoid prop drilling

3. **Testing**
   - Unit tests for utilities
   - Component tests
   - Integration tests
   - E2E tests for critical paths

### Commit Guidelines

1. **Commit Message Format**
   ```
   type(scope): description

   [optional body]

   [optional footer]
   ```

2. **Types**
   - feat: New feature
   - fix: Bug fix
   - docs: Documentation
   - style: Formatting
   - refactor: Code restructuring
   - test: Adding tests
   - chore: Maintenance

### Directory Structure

```
src/
├── app/             # Next.js app router files
├── components/      # React components
├── lib/            # Utility functions
├── server/         # Server-side code
├── styles/         # Global styles
├── types/          # TypeScript types
└── utils/          # Helper functions
```

## Setting Up Development Environment

1. **Required Tools**
   - Node.js (v18+)
   - npm or yarn
   - Git
   - AWS CLI

2. **Editor Setup**
   - VSCode recommended
   - Required extensions:
     - ESLint
     - Prettier
     - TypeScript
     - Tailwind CSS

3. **Environment Variables**
   ```env
   # Required
   AWS_REGION=
   AWS_ACCESS_KEY_ID=
   AWS_SECRET_ACCESS_KEY=

   # Optional
   NODE_ENV=development
   DEBUG=true
   ```

## Testing

1. **Running Tests**
   ```bash
   # Unit tests
   npm run test

   # Integration tests
   npm run test:integration

   # E2E tests
   npm run test:e2e

   # Coverage
   npm run test:coverage
   ```

2. **Writing Tests**
   ```typescript
   describe('Component/Feature', () => {
     beforeEach(() => {
       // Setup
     });

     it('should do something', () => {
       // Test
     });
   });
   ```

## Documentation

1. **Code Documentation**
   - JSDoc for functions
   - Component prop documentation
   - Type documentation
   - README updates

2. **Architecture Documentation**
   - Update diagrams
   - Document new features
   - Update setup guides

## Review Process

1. **Code Review Checklist**
   - [ ] Code follows style guide
   - [ ] Tests included
   - [ ] Documentation updated
   - [ ] Types properly defined
   - [ ] No unnecessary dependencies
   - [ ] Performance considered
   - [ ] Security implications reviewed

2. **Review Response Time**
   - Initial response: 2 business days
   - Follow-up reviews: 1 business day
   - Final approval: 1 business day

## Getting Help

- Create an issue
- Join discussions
- Review existing PRs
- Check documentation

## Recognition

Contributors will be:
- Added to CONTRIBUTORS.md
- Mentioned in release notes
- Credited in documentation

Thank you for contributing! 🙏
