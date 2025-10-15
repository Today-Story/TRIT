# TRIT Project Constitution for AI Agents

This document provides comprehensive guidelines for AI agents working on the TRIT project, which consists of a frontend monorepo (TRIT-FE) and a backend monorepo (TRIT-BE).

---

## 📋 Table of Contents
- [Project Overview](#project-overview)
- [Frontend (TRIT-FE) Constitution](#frontend-trit-fe-constitution)
- [Backend (TRIT-BE) Constitution](#backend-trit-be-constitution)
- [Cross-Project Guidelines](#cross-project-guidelines)
- [Agent Task Protocol](#agent-task-protocol)

---

## Project Overview

### System Architecture
**TRIT** is a travel recommendation and reservation platform consisting of:

#### Frontend Stack
- **Framework**: Next.js (App Router)
- **Monorepo**: Turborepo + PNPM workspaces
- **Applications**: 
  - `platform` - Main user-facing application
  - `admin` - Admin management interface
  - `backoffice` - Internal operations interface
- **Shared Packages**: UI components, hooks, utils, types, configurations
- **Styling**: TailwindCSS + design system
- **Type Safety**: TypeScript with strict mode

#### Backend Stack
- **Framework**: Spring Boot 3.3.9 (Java 17)
- **Build Tool**: Gradle
- **Database**: PostgreSQL (AWS RDS)
- **Cache**: Redis
- **ORM**: JPA + QueryDSL
- **Migration**: Liquibase
- **Deployment**: Docker + AWS EC2 + Nginx
- **Monitoring**: Grafana + Prometheus + Loki + Promtail
- **Performance Testing**: k6

---

## Frontend (TRIT-FE) Constitution

### 1. Project Structure

#### Workspace Organization
```
trit/TRIT-FE/
├── apps/
│   ├── platform/     # Main user application
│   ├── admin/        # Admin dashboard
│   └── backoffice/   # Internal operations
├── packages/
│   ├── ui/           # Shared component library (Storybook)
│   ├── types/        # Shared TypeScript types & schemas
│   ├── hooks/        # Shared React hooks
│   ├── utils/        # Shared utility functions
│   ├── next-config/  # Shared Next.js configuration
│   └── tailwind-config/ # Shared Tailwind configuration
├── turbo/            # Turborepo generators
└── turbo.json        # Turborepo pipeline configuration
```

#### Key Architectural Patterns
- **Monorepo Structure**: PNPM workspaces with Turborepo for build orchestration
- **Feature-First Organization**: Each app organizes features under `src/` with colocated components, hooks, and utilities
- **Design System**: Centralized in `@repo/ui` package with Storybook documentation
- **Type Safety**: Shared types in `@repo/types`, imported via workspace protocol

### 2. Development Guidelines

#### Code Style & Conventions
- **Language**: TypeScript (`.tsx` for components, `.ts` for utilities)
- **Formatting**: Prettier (2-space indent, single quotes, trailing commas)
- **Linting**: ESLint with `simple-import-sort` plugin
- **Component Style**: Functional components with TypeScript interfaces
- **Naming Conventions**:
  - Components: PascalCase (e.g., `UserProfile.tsx`)
  - Hooks: camelCase with `use` prefix (e.g., `useAuth.ts`)
  - Utils: camelCase (e.g., `formatDate.ts`)
  - Constants: UPPER_SNAKE_CASE in `packages/utils/src/constants`

#### Import Strategy
```typescript
// Internal app imports (within same app)
import { Component } from '@/components/Component'

// Workspace package imports
import { Button } from '@repo/ui'
import { UserSchema } from '@repo/types'

// Avoid climbing more than two directory levels with relative imports
// ❌ Bad: import { util } from '../../../utils/util'
// ✅ Good: import { util } from '@/utils/util'
```

#### Component Structure
```typescript
// Colocate related files
ComponentName/
├── ComponentName.tsx        # Component implementation
├── ComponentName.test.tsx   # Unit tests
├── ComponentName.stories.tsx # Storybook story
└── index.ts                 # Re-export
```

### 3. Build & Development Commands

#### Development
```bash
# Start all apps
pnpm dev

# Start specific app
pnpm dev:platform
pnpm dev:admin
pnpm dev:backoffice
```

#### Build
```bash
# Build all packages/apps
pnpm build

# Build specific app
pnpm build:platform
pnpm build:admin
pnpm build:backoffice
```

#### Quality Checks
```bash
# Run before committing (MANDATORY)
pnpm lint          # ESLint check
pnpm format        # Prettier format
pnpm check-types   # TypeScript type check
```

#### UI Library Development
```bash
# Start Storybook
pnpm --filter @repo/ui storybook

# Build UI library
pnpm --filter @repo/ui build

# Run UI tests
pnpm --filter @repo/ui vitest run --coverage
```

#### Code Generation
```bash
# Generate new component
pnpm turbo:gen "Generate Component"
```

### 4. Testing Guidelines

#### Testing Stack
- **Unit Testing**: Vitest + @testing-library/react
- **Component Testing**: Storybook + @storybook/test
- **Visual Testing**: Chromatic

#### Testing Patterns
```typescript
// Test file structure
describe('<ComponentName>', () => {
  it('should render correctly', () => {
    // Arrange
    const props = { /* ... */ }
    
    // Act
    render(<Component {...props} />)
    
    // Assert
    expect(screen.getByText('...')).toBeInTheDocument()
  })
})
```

#### Testing Requirements
- All new components in `@repo/ui` must have tests
- Mock network requests for deterministic tests
- Colocate tests with components (`*.test.tsx`)
- Maintain test coverage above 70%

### 5. Commit & PR Guidelines

#### Commit Message Format
```
[TEAM](type) - short summary (#issue)

Examples:
[TRIPLE](feat) - add playlist bookmark feature (#862)
[TRIPLE](fix) - resolve pagination bug in content list (#901)
[TRIPLE](refactor) - improve performance of search component (#922)
[TACOS](chore) - update dependencies to latest versions (#945)
```

#### Commit Types
- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code refactoring without behavior change
- `chore`: Maintenance tasks (dependencies, configs)
- `docs`: Documentation changes
- `test`: Test additions or modifications
- `style`: Code style/formatting changes

#### Pull Request Requirements
- **Base Branch**: Follow team branching strategy
- **PR Description Must Include**:
  1. Summary of changes and reasoning
  2. Related issue/ticket number
  3. Screenshots or recordings for UI changes
  4. Testing instructions
  5. Build/lint/type-check outputs
  6. Customer/user impact explanation
- **Checks**:
  - All CI checks must pass (build, lint, type-check, tests)
  - At least one approval from relevant team (`@TRIPLE`, `@TACOS`)
  - No merge conflicts

### 6. API Integration Guidelines

#### API Client Patterns
- Use centralized API client configuration
- Implement proper error handling and retry logic
- Type all API responses with `@repo/types` schemas
- Use React Query or SWR for data fetching
- Implement loading and error states consistently

#### Environment Variables
```typescript
// Access environment variables safely
const apiUrl = process.env.NEXT_PUBLIC_API_URL
if (!apiUrl) {
  throw new Error('NEXT_PUBLIC_API_URL is not defined')
}
```

### 7. Performance Guidelines

- **Code Splitting**: Use dynamic imports for large components
- **Image Optimization**: Use Next.js Image component
- **Bundle Analysis**: Regular bundle size checks
- **Lazy Loading**: Implement for below-the-fold content
- **Memoization**: Use React.memo, useMemo, useCallback appropriately

### 8. Accessibility Guidelines

- Semantic HTML elements
- ARIA labels where necessary
- Keyboard navigation support
- Color contrast compliance (WCAG AA minimum)
- Screen reader testing for critical flows

---

## Backend (TRIT-BE) Constitution

### 1. Project Structure

#### Domain Organization
```
trit/TRIT-BE/backend/src/main/java/today/story/backend/
├── admin/           # Admin features (user/system management, roles)
├── auth/            # Authentication & Authorization (JWT)
├── campaign/        # Campaign management (CRUD, applications, approvals)
├── comments/        # Comment features (CRUD)
├── common/          # Common utilities, constants, base entities
├── company/         # Company information & holiday management
├── config/          # Application configuration (S3, JPA, Redis, Security, Swagger)
├── contents/        # Content management
├── creator/         # Creator information management
├── facade/          # Integrated APIs (combining multiple services)
├── hashtags/        # Hashtag management
├── history/         # Like/view history tracking
├── holiday/         # Holiday scheduler & external API integration
├── location/        # Location management (ChatGPT place name correction)
├── payment/         # Payment processing (Eromnet PG)
├── place/           # Place management
├── playlist/        # Playlist features
├── product/         # Product & schedule management
├── report/          # Inquiry data access
├── reservation/     # Reservation management
├── theme/           # Themed content grouping
└── users/           # User management (signup, login, mypage)
```

#### Resource Structure
```
src/main/resources/
├── db/
│   └── changelog/   # Liquibase database migrations
├── static/
│   └── mail-template/ # Email templates (welcome, approve-company, approve-creator)
├── application.yml
├── application-local.yml
├── application-dev.yml
├── application-prod.yml
└── log4j2-spring.xml
```

### 2. Architectural Principles

#### Layered Architecture (STRICT)
```
Controller Layer
    ↓
Service Layer (Business Logic)
    ↓
Repository Layer (Data Access)
```

#### Key Patterns
- **Controllers**: REST endpoints at `/api/v1/...`, return `ResultResponse` or `PageResponse`
- **Services**: Business logic, DTO ↔ Entity mapping (MapStruct), transaction management
- **Repositories**: JPA repositories, custom QueryDSL queries
- **DTOs**: Validation via `@Valid`, `@NotNull`, `@NotBlank`
- **Validation**: Custom validators in `validate` package
- **Exception Handling**: Centralized in `common` exception handlers

### 3. Coding Standards

#### Transaction Management
```java
// Default: read-only transactions for query methods
@Transactional(readOnly = true)
public UserResponse getUser(Long userId) {
    // ...
}

// Explicit write transactions
@Transactional
public UserResponse createUser(CreateUserRequest request) {
    // ...
}
```

#### DTO Validation
```java
public class CreateProductRequest {
    @NotBlank(message = "Product name is required")
    private String name;
    
    @NotNull(message = "Price is required")
    @Positive(message = "Price must be positive")
    private BigDecimal price;
}
```

#### Response Patterns
```java
// Success response
return ResultResponse.success(data);

// Paginated response
return PageResponse.of(page, content);
```

#### Logging Standards
- Use Log4j2
- **NEVER** log secrets, tokens, or PII
- Log levels: ERROR (failures), WARN (degraded), INFO (key events), DEBUG (detailed)
- Include correlation IDs for tracing

### 4. Database Guidelines

#### Liquibase Migrations
```yaml
# Enable Liquibase
./gradlew -PenableLiquibase=true :backend:bootRun

# ChangeSet structure
- changeSet:
    id: 2025-10-15-add-column-example
    author: agent-name
    changes:
      - addColumn:
          tableName: products
          columns:
            - column:
                name: featured
                type: boolean
                defaultValueBoolean: false
    rollback:
      - dropColumn:
          tableName: products
          columnName: featured
```

#### Migration Rules
- One changeSet per logical feature
- Always provide rollback when possible
- Test migrations on staging before production
- Document destructive changes
- Two-step migration for breaking changes:
  1. Add new column (nullable)
  2. Dual-write to both columns
  3. Migrate data
  4. Switch reads to new column
  5. Drop old column

#### Query Optimization
- **N+1 Prevention**: Use fetch joins or two-step pagination
- **Pagination**: Use `page` and `size` parameters, sometimes `SortOption` enum
- **Indexing**: Add indexes for frequently queried columns
- **Query Analysis**: Check query count and latency (p95, p99)

```java
// Two-step pagination example
// Step 1: Get IDs
List<Long> ids = repository.findIdsByCondition(pageable);

// Step 2: Fetch with associations
List<Entity> entities = repository.findByIdInWithFetchJoin(ids);
```

### 5. API Development Rules

#### Critical Constraints (NEVER VIOLATE)
- **NEVER** change `ProductResponse` structure (backward compatibility)
- **NEVER** convert Enums to Strings (maintain type safety)
- **NEVER** omit DTO validation
- **NEVER** expose secrets/keys/PII in logs or responses
- Avoid destructive schema changes without approval

#### Standard Response Format
```java
@GetMapping("/api/v1/products/{id}")
public ResultResponse<ProductResponse> getProduct(@PathVariable Long id) {
    ProductResponse product = productService.getProduct(id);
    return ResultResponse.success(product);
}
```

#### Error Handling
```java
// Use domain-specific exceptions
throw new ProductNotFoundException("Product not found: " + id);

// Global exception handler catches and formats
@ExceptionHandler(ProductNotFoundException.class)
public ResultResponse<Void> handleProductNotFound(ProductNotFoundException e) {
    return ResultResponse.error(ErrorCode.PRODUCT_NOT_FOUND, e.getMessage());
}
```

### 6. Testing Guidelines

#### Test Structure
```bash
# Run all tests
./gradlew test

# Run package tests
./gradlew test --tests 'today.story.backend.product.*'

# Run single test
./gradlew test --tests 'ProductServiceTest.shouldCreateProduct'
```

#### Test Requirements
- Unit tests for business logic
- Integration tests for API endpoints
- Mock external dependencies
- Test edge cases and error scenarios
- Maintain test coverage above 70%

### 7. Performance Testing (k6)

#### Running Performance Tests
```bash
# Run k6 script
k6 run scripts/performance-test-scripts.js

# With Docker Compose
docker compose -f docker-compose.k6.yml run k6 run /scripts/load-test.js
```

#### Performance Requirements
- Document baseline metrics (req/s, error %, p95/p99 latency)
- Include performance test results in PRs for performance-related changes
- Monitor for regressions in query count and response times

### 8. Deployment & CI/CD

#### Local Development
```bash
# Gradle (local profile)
cd backend
./gradlew clean build
./gradlew bootRun

# With specific profile
./gradlew bootRun --args='--spring.profiles.active=dev'

# Docker Compose (development)
docker compose -f docker-compose.develop.yml up -d

# View logs
docker compose -f docker-compose.develop.yml logs -f

# Stop containers
docker compose -f docker-compose.develop.yml down
```

#### Deployment
```bash
# Staging
./deploy-script.sh develop

# Production
./deploy-script.sh prod
```

#### CI/CD Pipeline
- **Trigger**: PR, Push, Git Tag
- **Steps**: Build → Test → Docker Image → Deploy
- **Notifications**: Slack notifications for success/failure
- **Rollback**: Immediate rollback on failure with documented cause

#### Branch Strategy
- Branch naming: `codex/<feature-name>` (English, hyphen-separated)
- PR base: `codex-bot` branch
- Merge requirements: CI green, tests pass, one approval

### 9. Security Guidelines

#### Secrets Management
- Use `.env.example` for local development templates
- **NEVER** commit real keys to repository
- Secrets stored in CI/CD secrets or server keychain
- No secrets/PII in logs, commits, or PR descriptions

#### External API Integration
- Always set timeouts and retry policies
- Implement circuit breakers for critical services
- Classify errors (transient vs. permanent)
- Log integration failures with context

### 10. Monitoring & Observability

#### Stack
- **Logs**: Promtail → Loki
- **Metrics**: Prometheus + cAdvisor
- **Dashboards**: Grafana
- **Performance**: k6 + InfluxDB

#### Common Issues & Runbook
- **Liquibase Lock**: Unlock via manual SQL, retry migration
- **External API Errors**: Check timeout/retry/circuit breaker config
- **Cache Mismatch**: Invalidate Redis keys, adjust TTL
- **Database Connection Pool**: Monitor active connections, adjust pool size

---

## Cross-Project Guidelines

### 1. API Contract Management

#### Frontend ↔ Backend Integration
- Backend maintains API contract stability
- Frontend uses typed API clients (generated from OpenAPI/Swagger)
- Version APIs appropriately (`/api/v1/`, `/api/v2/`)
- Deprecate endpoints gracefully with migration period

#### Type Synchronization
```typescript
// Frontend (@repo/types)
export interface ProductResponse {
  id: number
  name: string
  price: number
  // Must match backend ProductResponse exactly
}
```

```java
// Backend
public class ProductResponse {
    private Long id;
    private String name;
    private BigDecimal price;
    // Must match frontend interface
}
```

### 2. Development Workflow

#### Feature Development Flow
1. **Planning**: Design API contract, frontend mockups
2. **Backend**: Implement API endpoints, tests, documentation
3. **Frontend**: Implement UI using mocked API
4. **Integration**: Connect frontend to backend staging environment
5. **Testing**: E2E tests, performance tests
6. **Review**: Cross-team PR review
7. **Deploy**: Coordinate deployment (backend first, then frontend)

#### Local Development Setup
```bash
# Backend
cd TRIT-BE
docker compose -f docker-compose.develop.yml up -d

# Frontend
cd TRIT-FE
pnpm dev:platform
# Frontend will connect to local backend at localhost:8080
```

### 3. Git Strategy

#### Repository Structure
```
trit/                      # Main repository
├── TRIT-FE/              # Frontend submodule
├── TRIT-BE/              # Backend submodule
├── CONSTITUTION.md       # This document
└── README.md             # Project overview
```

#### Submodule Management
```bash
# Update submodules
git submodule update --remote

# Commit submodule changes
git add TRIT-FE TRIT-BE
git commit -m "chore: update submodules to latest"
git push
```

### 4. Communication & Documentation

#### Required Documentation
- API changes: Update Swagger/OpenAPI specs
- Database changes: Document in Liquibase changesets
- UI changes: Update Storybook stories
- Breaking changes: Create migration guide

#### Team Communication
- Slack: Real-time communication
- Notion: Design docs, architecture decisions
- GitHub: Code reviews, technical discussions
- Jira: Task tracking, sprint planning

---

## Agent Task Protocol

### 1. Analysis Phase
**Before making any changes, agents must:**

1. **Identify Scope**
   - Which submodule(s) are affected? (FE, BE, both)
   - Which domain/package is primary?
   - Does this require API contract changes?
   - Are there database migrations involved?

2. **Review Constraints**
   - Frontend: Check AGENTS.md, team conventions, component library guidelines
   - Backend: Check AGENTS.md guardrails, API rules, DB constraints
   - Cross-cutting: Review this constitution, particularly API contract section

3. **Check Dependencies**
   - Will changes in one submodule require changes in the other?
   - Are there shared types that need updating?
   - Will this affect other teams' work?

### 2. Planning Phase

1. **Break Down Work**
   - Create separate PRs for frontend and backend when possible
   - Plan deployment order (typically backend first)
   - Identify rollback points

2. **Design Changes**
   - API contract: Design request/response formats
   - Database: Plan migrations with rollback strategy
   - Frontend: Plan component structure and state management
   - Tests: Plan test coverage for new functionality

3. **Document Plan**
   - Create detailed PR description templates
   - Document API changes for both teams
   - Create migration guide if breaking changes

### 3. Implementation Phase

#### For Frontend Changes
```bash
cd TRIT-FE

# 1. Create feature branch
git checkout -b feat/add-product-filters

# 2. Make changes following TRIT-FE constitution
# - Update components in @repo/ui if shared
# - Add types to @repo/types
# - Implement feature in target app (platform/admin/backoffice)

# 3. Run quality checks
pnpm lint
pnpm format
pnpm check-types
pnpm build

# 4. Test changes
pnpm --filter @repo/ui vitest run
pnpm --filter platform test  # if applicable

# 5. Commit with proper format
git add .
git commit -m "[TRIPLE](feat) - add product filter components (#1234)"

# 6. Push and create PR
git push origin feat/add-product-filters
```

#### For Backend Changes
```bash
cd TRIT-BE

# 1. Create feature branch
git checkout -b codex/add-product-filters

# 2. Make changes following TRIT-BE constitution
# - Add/modify entities, DTOs, services, controllers
# - Create Liquibase changeset if needed
# - Follow layered architecture

# 3. Run tests
./gradlew test

# 4. Test with Docker
docker compose -f docker-compose.develop.yml up -d
# Verify endpoints with curl or Postman

# 5. Run performance tests if applicable
k6 run scripts/performance-test-scripts.js

# 6. Commit with proper format
git add .
git commit -m "feat(product): add filtering endpoint"

# 7. Push and create PR to codex-bot branch
git push origin codex/add-product-filters
```

#### For Full-Stack Features
```bash
# 1. Implement backend first
cd TRIT-BE
# ... implement backend API ...
git commit -m "feat(product): add filtering endpoint"
git push

# 2. Wait for backend PR review and merge

# 3. Implement frontend
cd ../TRIT-FE
# ... implement frontend UI ...
git commit -m "[TRIPLE](feat) - add product filters UI (#1234)"
git push

# 4. Update main repository submodule references
cd ..
git add TRIT-BE TRIT-FE
git commit -m "feat: add product filtering feature"
git push
```

### 4. Testing Phase

#### Frontend Testing Checklist
- [ ] Components render correctly
- [ ] UI matches design specifications
- [ ] API integration works with backend
- [ ] Loading and error states display properly
- [ ] Responsive design works on all breakpoints
- [ ] Accessibility requirements met
- [ ] Browser compatibility verified
- [ ] Performance benchmarks met

#### Backend Testing Checklist
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] API contract matches documentation
- [ ] Database migrations run successfully
- [ ] Rollback script works
- [ ] Performance tests show acceptable metrics
- [ ] No N+1 query issues
- [ ] Error handling works correctly
- [ ] Logs contain no sensitive data

### 5. PR Creation Guidelines

#### Frontend PR Template
```markdown
## Summary
Brief description of changes

## Related Issue
Closes #1234

## Changes
- Added ProductFilter component
- Updated ProductList to support filtering
- Added filter types to @repo/types

## Screenshots/Videos
[Attach screenshots or recordings]

## Testing
- [ ] Lint passed: `pnpm lint`
- [ ] Types checked: `pnpm check-types`
- [ ] Build succeeded: `pnpm build:platform`
- [ ] Tests passed: `pnpm test`

## API Changes
Integrates with `/api/v1/products?filter=...` endpoint

## Team
@TRIPLE
```

#### Backend PR Template
```markdown
## Summary
Brief description of changes

## Related Issue
Closes #5678

## Changes
- Added filtering endpoint to ProductController
- Implemented ProductFilterService
- Added database indexes for filter columns
- Created Liquibase changeset 2025-10-15-add-product-indexes

## API Changes
### New Endpoint
\`\`\`
GET /api/v1/products?filter={filterType}&value={value}
\`\`\`

### Response Format
\`\`\`json
{
  "success": true,
  "data": [...],
  "pagination": {...}
}
\`\`\`

## Database Changes
- Added indexes: idx_product_category, idx_product_price
- Rollback available in changeset

## Testing
\`\`\`bash
./gradlew test --tests 'ProductControllerTest'
# All tests passed
\`\`\`

## Performance
- k6 test results: 500 req/s, p95 < 200ms, 0% error rate
- No N+1 queries detected

## Deployment
- Deploy order: Run migrations → Deploy backend → Deploy frontend
- Rollback: Revert deployment → Run rollback changeset
```

### 6. Review Phase

#### As a Reviewer
- Check adherence to constitution guidelines
- Verify test coverage
- Review security implications
- Check performance impact
- Ensure documentation is updated
- Verify rollback plan exists

#### As an Agent
- Address all review comments
- Update tests if requested
- Improve documentation if unclear
- Re-run all checks after changes

### 7. Deployment Phase

#### Pre-Deployment Checklist
- [ ] All tests pass
- [ ] PR approved by required reviewers
- [ ] Database migrations tested on staging
- [ ] Performance benchmarks acceptable
- [ ] Rollback plan documented
- [ ] Related teams notified

#### Deployment Order
1. **Backend**:
   - Run database migrations
   - Deploy backend service
   - Verify health checks
   - Monitor error rates

2. **Frontend**:
   - Build and deploy frontend
   - Verify API integration
   - Monitor client-side errors

3. **Monitoring**:
   - Watch dashboards for 15-30 minutes
   - Check logs for unexpected errors
   - Verify metrics are normal

#### Post-Deployment
- Update ticket status
- Document any issues encountered
- Share deployment summary with team

### 8. Rollback Protocol

#### If Issues Detected
1. **Immediate**: Notify team in Slack
2. **Assess**: Determine severity (critical → rollback, minor → hotfix)
3. **Rollback** (if needed):
   ```bash
   # Backend
   cd TRIT-BE
   ./deploy-script.sh rollback
   
   # Database (if needed)
   cd backend
   ./gradlew -PenableLiquibase=true liquibaseRollback
   
   # Frontend
   cd TRIT-FE
   # Redeploy previous version via CI/CD
   ```
4. **Document**: Log issue, root cause, resolution in incident log

---

## Best Practices Summary

### For AI Agents Working on TRIT

#### ✅ DO
- Read and follow both AGENTS.md files in submodules
- Follow commit message conventions strictly
- Run all quality checks before committing
- Create small, focused PRs
- Document API changes comprehensively
- Test across both frontend and backend
- Coordinate with other teams
- Provide rollback plans
- Monitor after deployment

#### ❌ DON'T
- Make changes without understanding impact
- Commit secrets or PII
- Skip tests or quality checks
- Create large, multi-purpose PRs
- Change API contracts without documentation
- Make destructive database changes without approval
- Deploy without proper testing
- Ignore performance implications
- Leave code commented out (remove or document why)

---

## Appendix

### Useful Commands Reference

#### Frontend Commands
```bash
# Development
pnpm dev                          # All apps
pnpm dev:platform                 # Platform only

# Building
pnpm build                        # All apps
pnpm build:platform              # Platform only

# Testing
pnpm lint                        # ESLint
pnpm format                      # Prettier
pnpm check-types                 # TypeScript
pnpm --filter @repo/ui test      # UI tests

# Storybook
pnpm --filter @repo/ui storybook # Start Storybook
```

#### Backend Commands
```bash
# Development
./gradlew clean build            # Build
./gradlew bootRun               # Run (local profile)
./gradlew bootRun --args='--spring.profiles.active=dev'  # Dev profile

# Testing
./gradlew test                           # All tests
./gradlew test --tests 'package.*'       # Package tests

# Docker
docker compose -f docker-compose.develop.yml up -d      # Start
docker compose -f docker-compose.develop.yml logs -f    # Logs
docker compose -f docker-compose.develop.yml down       # Stop

# Database
./gradlew -PenableLiquibase=true :backend:bootRun       # With migrations
./gradlew liquibaseUpdate                               # Run migrations

# Performance
k6 run scripts/performance-test-scripts.js              # Load test

# Deployment
./deploy-script.sh develop                              # Staging
./deploy-script.sh prod                                 # Production
```

#### Git Commands
```bash
# Submodule management
git submodule update --init --recursive                 # Initialize
git submodule update --remote                           # Update
git add TRIT-FE TRIT-BE                                # Stage submodules
git commit -m "chore: update submodules"               # Commit changes
```

### Environment Setup

#### Prerequisites
- **Frontend**: Node.js 18+, PNPM 8+
- **Backend**: JDK 17, Gradle 8+, Docker, Docker Compose
- **Tools**: Git, k6 (for performance testing)

#### Initial Setup
```bash
# Clone main repository with submodules
git clone --recursive https://github.com/Today-Story/TRIT.git
cd TRIT

# Frontend setup
cd TRIT-FE
pnpm install

# Backend setup
cd ../TRIT-BE
cd backend
./gradlew build

# Start local environment
docker compose -f docker-compose.develop.yml up -d
```

---

## Document Maintenance

This constitution should be updated when:
- Major architectural changes occur
- New tools or frameworks are introduced
- Team conventions evolve
- Common issues are identified and resolved

**Last Updated**: 2025-10-15
**Version**: 1.0.0
**Maintained By**: Development Team

---

**For questions or clarifications, contact the team leads or refer to project documentation in Notion.**
