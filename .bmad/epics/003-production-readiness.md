# Epic: Production Readiness & Testing

**Epic ID:** 003
**Created:** 2026-01-15
**Status:** Draft
**Complexity Level:** 2

## Project Context

- **Project:** openhorizon
- **Repository:** https://github.com/gpt153/openhorizon.cc
- **Tech Stack:** Next.js 16, React 19, TypeScript, Fastify, Playwright, Vitest, PostgreSQL, Cloud Run
- **Related Epics:** Blocked by Epic #001 (API Timeouts) and Epic #002 (Auth Stability)
- **Workspace:** `/home/samuel/.archon/workspaces/openhorizon.cc/`
- **Worktree Base:** `/home/samuel/.archon/worktrees/openhorizon.cc/`

## Business Context

### Problem Statement
OpenHorizon is ~91% feature-complete but lacks comprehensive testing, production validation, and operational readiness needed to support real users. The February 2026 deadline to support 3-5 Erasmus+ applications requires a stable, well-tested platform that won't fail during critical application submission periods.

Without thorough testing and production validation, we risk:
- Data loss or corruption during user sessions
- Critical bugs discovered during live usage
- Performance issues under real-world load
- Poor user experience that damages credibility

### User Value
Users need to:
- Trust that their project data is safe and won't be lost
- Experience consistent performance without crashes or freezes
- Complete application workflows without encountering bugs
- Receive responsive support when issues occur
- Successfully submit Erasmus+ applications with confidence

Production readiness ensures OpenHorizon meets professional standards expected by organizations submitting real grant applications worth €20,000-€60,000.

### Success Metrics
- Metric 1: E2E test coverage >80% for critical user journeys
- Metric 2: Zero P0 (critical) bugs in production during beta period
- Metric 3: System uptime >99.5% during February 2026
- Metric 4: User-reported bug rate <2% of completed application workflows
- Metric 5: All 3-5 beta organizations successfully submit applications

## Requirements

### Functional Requirements (MoSCoW)

**MUST HAVE:**
- [ ] Complete E2E test coverage for critical paths (seed creation → elaboration → project generation → programme builder → budget planning → document export)
- [ ] Verify database migrations applied correctly in production
- [ ] Implement comprehensive error boundaries in React app
- [ ] Add monitoring/logging for all critical operations
- [ ] Create runbook for common operational issues
- [ ] Perform load testing (simulate 50 concurrent users)
- [ ] Validate all document exports (PDF/DOCX) format correctly
- [ ] Test multi-tenant isolation (verify org data properly segregated)

**SHOULD HAVE:**
- [ ] Add performance monitoring (track page load times, API latency)
- [ ] Implement automated backup verification
- [ ] Create user onboarding flow with tooltips/guides
- [ ] Add usage analytics to understand user behavior
- [ ] Set up error tracking (Sentry or similar)
- [ ] Create admin dashboard for monitoring system health

**COULD HAVE:**
- [ ] Add feature flags for gradual rollout
- [ ] Implement A/B testing framework
- [ ] Add user feedback widget
- [ ] Create changelog/release notes system

**WON'T HAVE (this iteration):**
- Chaos engineering / fault injection testing - Deferred to post-launch
- Multi-region deployment - Single region sufficient for MVP
- CDN for static assets - Cloud Run serves adequately for beta
- Advanced observability (distributed tracing) - Basic logging sufficient for now

### Non-Functional Requirements

**Reliability:**
- Availability: 99.5% uptime during February 2026
- Data durability: Zero data loss (database backups + point-in-time recovery)
- Failure recovery: Automatic restarts for crashed services
- Error handling: All errors logged with sufficient context for debugging

**Performance:**
- Page load time: < 2 seconds (p95)
- API response time: < 500ms (p95)
- AI generation time: < 30 seconds for project generation
- Database query time: < 100ms (p95)
- Document export time: < 5 seconds for PDF generation

**Security:**
- Vulnerability scanning: Zero high/critical vulnerabilities (npm audit)
- Dependency updates: All critical security patches applied
- Data encryption: At rest (database) and in transit (HTTPS)
- Access control: Role-based permissions properly enforced
- Audit logging: All sensitive operations logged

**Scalability:**
- Concurrent users: Support 50+ simultaneous users
- Database connections: Properly pooled and managed
- Background jobs: Queue system handles spikes gracefully
- Storage: Sufficient capacity for 500+ projects

## Architecture

### Technical Approach
**Pattern:** Comprehensive testing pyramid (unit → integration → E2E) + production monitoring + incident response
**Testing Stack:** Vitest (unit/integration), Playwright (E2E), manual testing checklist
**Monitoring Stack:** Cloud Run logs, database metrics, error tracking, uptime monitoring

### Integration Points
- **Testing:** Playwright tests against staging environment (oh.153.se)
- **Monitoring:** Cloud Run metrics + custom logging + error tracking service
- **Database:** Verify migrations, indexes, backup schedule
- **Deployment:** Validate Docker builds, environment variables, service configuration

### Data Flow (Testing)
```
Local development
  ↓
Unit tests (Vitest) - validate business logic
  ↓
Integration tests (Vitest) - validate API + DB
  ↓
E2E tests (Playwright) - validate user journeys
  ↓
Staging deployment (oh.153.se)
  ↓
Manual testing with beta users
  ↓
Production deployment
  ↓
Continuous monitoring
```

### Key Technical Decisions
- **Decision 1:** Use Playwright (not Cypress) for E2E tests due to better TypeScript support and parallel execution (see ADR-008)
- **Decision 2:** Test against staging environment (not production) to avoid polluting real user data (see ADR-008)
- **Decision 3:** Manual testing required for complex AI-generated content (automated tests can't validate semantic quality) (see ADR-008)

### Files to Create/Modify
```
app/
├── tests/
│   ├── e2e/
│   │   ├── seed-creation.spec.ts         # NEW - Test seed brainstorming
│   │   ├── seed-elaboration.spec.ts      # NEW - Test conversational elaboration
│   │   ├── project-generation.spec.ts    # NEW - Test seed → project conversion
│   │   ├── programme-builder.spec.ts     # NEW - Test multi-day programme creation
│   │   ├── budget-planning.spec.ts       # NEW - Test vendor searches + budget calc
│   │   ├── document-export.spec.ts       # NEW - Test PDF/DOCX generation
│   │   └── multi-tenant.spec.ts          # NEW - Test org data isolation
│   ├── integration/
│   │   ├── auth.test.ts                  # ADD - Clerk integration tests
│   │   ├── database.test.ts              # ADD - Prisma CRUD tests
│   │   └── ai-agents.test.ts             # ADD - LangChain agent tests
│   └── playwright.config.ts              # REVIEW - Verify configuration
├── src/
│   ├── app/
│   │   ├── error.tsx                     # IMPROVE - Global error boundary
│   │   └── layout.tsx                    # IMPROVE - Add error boundary
│   └── lib/
│       └── monitoring.ts                 # NEW - Logging/monitoring utilities
├── docs/
│   ├── runbook.md                        # NEW - Operational procedures
│   ├── testing-guide.md                  # NEW - Testing documentation
│   └── troubleshooting.md                # NEW - Common issues + solutions
└── scripts/
    ├── verify-migrations.sh              # NEW - Check DB schema vs. Prisma
    └── load-test.sh                      # NEW - Load testing script

project-pipeline/
└── frontend/
    └── tests/
        └── e2e/
            └── pipeline-workflow.spec.ts # NEW - Test project pipeline
```

## Implementation Tasks

### Breakdown into GitHub Issues

**Issue #1: Testing - E2E tests for seed management**
- Test seed creation flow (brainstorming → seed garden)
- Test seed editing and versioning
- Test seed deletion
- Acceptance: All seed management E2E tests pass

**Issue #2: Testing - E2E tests for conversational elaboration**
- Test starting elaboration session
- Test chat interaction with AI
- Test metadata extraction during elaboration
- Test progress tracking (completeness %)
- Test conversion to project at 80% complete
- Acceptance: All elaboration E2E tests pass

**Issue #3: Testing - E2E tests for project generation**
- Test seed → project conversion
- Test project DNA generation
- Test objectives generation
- Test participant profiles generation
- Test budget generation
- Acceptance: All project generation E2E tests pass

**Issue #4: Testing - E2E tests for programme builder**
- Test multi-day programme creation
- Test session-level activity planning
- Test learning objectives integration
- Test working/formal mode toggle
- Acceptance: All programme builder E2E tests pass

**Issue #5: Testing - E2E tests for budget planning**
- Test Food agent search (background job)
- Test Accommodation agent search (background job)
- Test Travel agent search
- Test budget calculator
- Test vendor quote management
- Acceptance: All budget planning E2E tests pass

**Issue #6: Testing - E2E tests for document export**
- Test PDF export (application form)
- Test DOCX export (project report)
- Verify document formatting is correct
- Test multi-language support (working vs. formal)
- Acceptance: All document export E2E tests pass

**Issue #7: Testing - Multi-tenant isolation tests**
- Test user A cannot access user B's projects (different orgs)
- Test organization member roles enforce permissions
- Test database queries properly filter by organizationId
- Acceptance: All multi-tenant tests pass, no data leaks

**Issue #8: Database - Verify production migrations**
- Connect to production database (read-only)
- Run `npx prisma migrate status`
- Compare schema with Prisma schema.prisma
- Apply any missing migrations
- Acceptance: Production DB schema matches codebase

**Issue #9: Monitoring - Add comprehensive logging**
- Log all AI generation requests (seed, project, programme)
- Log all search jobs (Food, Accommodation, Travel)
- Log all authentication events
- Log all document exports
- Add request ID for tracing
- Acceptance: All critical operations logged

**Issue #10: Monitoring - Set up error tracking**
- Integrate Sentry or similar error tracking service
- Add error boundaries to React components
- Configure source maps for production debugging
- Set up alerts for error spikes
- Acceptance: Errors automatically reported with stack traces

**Issue #11: Performance - Load testing**
- Create load test script (50 concurrent users)
- Test seed creation under load
- Test project generation under load
- Test document export under load
- Measure database connection pooling
- Acceptance: System handles 50 users without crashes or timeouts

**Issue #12: Operations - Create runbook**
- Document how to deploy new versions
- Document how to rollback deployments
- Document how to check system health
- Document how to investigate errors
- Document how to restore from backup
- Acceptance: Runbook complete and reviewed

**Issue #13: User Experience - Onboarding flow**
- Add welcome modal for first-time users
- Add tooltips for key features
- Add sample project template (optional)
- Add help documentation links
- Acceptance: New users can complete first project without confusion

**Issue #14: Security - Vulnerability audit**
- Run `npm audit` and fix all high/critical issues
- Review all environment variables (no secrets in git)
- Verify HTTPS enforced on all endpoints
- Verify CORS configured correctly
- Verify rate limiting on public endpoints
- Acceptance: Zero high/critical vulnerabilities

### Estimated Effort
- E2E tests (6 test suites): 18 hours
- Multi-tenant isolation tests: 3 hours
- Database migration verification: 2 hours
- Logging and monitoring: 6 hours
- Error tracking setup: 3 hours
- Load testing: 4 hours
- Runbook and documentation: 4 hours
- Onboarding flow: 6 hours
- Security audit: 4 hours
- Manual testing and bug fixes: 10 hours
- Total: 60 hours (~7-8 days)

## Acceptance Criteria

**Feature-Level Acceptance:**
- [ ] All E2E tests pass (seed, elaboration, project, programme, budget, export)
- [ ] Multi-tenant isolation verified (no data leaks)
- [ ] Production database migrations verified and applied
- [ ] Load testing passed (50 concurrent users)
- [ ] Security audit completed (zero high/critical vulnerabilities)
- [ ] Monitoring and logging operational
- [ ] Error tracking service integrated
- [ ] Runbook documented and reviewed
- [ ] User onboarding flow implemented
- [ ] All builds succeed with zero TypeScript errors
- [ ] No console errors in production

**Code Quality:**
- [ ] E2E tests have clear descriptions and good coverage
- [ ] Error boundaries handle all error scenarios
- [ ] Logging does not expose sensitive data
- [ ] Tests are deterministic (no flaky tests)

**Performance:**
- [ ] Page load time < 2 seconds (p95)
- [ ] API response time < 500ms (p95)
- [ ] Load test confirms system scales to 50+ users

**Documentation:**
- [ ] Runbook complete (deploy, rollback, troubleshoot)
- [ ] Testing guide complete (how to run tests locally)
- [ ] Troubleshooting guide complete (common issues + solutions)
- [ ] ADR-008: Testing Strategy documented

## Dependencies

**Blocked By:**
- Epic #001: API Timeouts (must be fixed before budget planning tests)
- Epic #002: Auth Stability (must be fixed before auth/onboarding tests)

**Blocks:**
- None (final epic before production launch)

**External Dependencies:**
- Staging environment (oh.153.se) must be accessible
- Production database access (for migration verification)
- Error tracking service account (Sentry or equivalent)
- Load testing infrastructure (can use local machine or CI)

## Risks & Mitigation

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| E2E tests are flaky due to AI non-determinism | High | Medium | Use mock responses for AI in tests; test against consistent seed data |
| Load testing reveals performance bottlenecks | Medium | High | Identify bottlenecks early; optimize database queries; add caching |
| Production database schema out of sync | Low | Critical | Run migration verification immediately; test migrations on staging first |
| Beta users discover critical bugs during testing | Medium | High | Reserve 1 week buffer before deadline for bug fixes; prioritize ruthlessly |
| Onboarding flow too complex, users confused | Medium | Medium | User testing with 1-2 friendly users first; iterate on feedback |

## Testing Strategy

### Unit Tests (Vitest)
- Business logic functions (budget calculations, date formatting)
- Utility functions (data transformations, validators)
- React hooks (custom hooks isolated from components)

### Integration Tests (Vitest)
- tRPC procedures (API + database)
- Clerk authentication flow (mock Clerk API)
- Prisma database operations (use test database)
- LangChain agent logic (mock LLM responses)
- Inngest functions (background jobs)

### E2E Tests (Playwright)
- Complete user journeys (seed → project → document export)
- Multi-tenant scenarios (multiple users, multiple orgs)
- Error handling (network failures, API timeouts)
- Authentication flows (signup, login, logout)
- Document generation (verify PDF/DOCX output)

### Load Tests
- Use `k6` or `artillery` to simulate 50 concurrent users
- Test scenarios: seed creation, project generation, document export
- Measure: throughput, latency (p50/p95/p99), error rate
- Monitor: database connections, memory usage, CPU usage

### Manual Testing Checklist
- [ ] Complete full application workflow (seed → document) as new user
- [ ] Test on different browsers (Chrome, Firefox, Safari)
- [ ] Test on different screen sizes (desktop, tablet, mobile)
- [ ] Test slow network conditions (throttle to 3G)
- [ ] Test with ad blockers enabled
- [ ] Test with screen reader (basic accessibility)
- [ ] Review generated documents for formatting quality
- [ ] Test multi-user collaboration (two users in same org)
- [ ] Verify email notifications work (if applicable)
- [ ] Test error scenarios (invalid input, API failures)

## Notes

### Design Decisions
**Why Playwright over Cypress?**
Playwright offers better TypeScript support, faster parallel execution, and more reliable auto-waiting. It's actively maintained by Microsoft and has excellent documentation.

**Why staging environment for E2E tests?**
Testing against production would pollute real user data and risk breaking live services. Staging provides production-like environment for safe testing.

**Why manual testing still required?**
AI-generated content (project descriptions, learning objectives) requires human evaluation for semantic quality. Automated tests verify technical correctness but not educational value.

### Known Limitations
- E2E tests may be slower than unit tests (trade-off for coverage)
- AI non-determinism makes some tests harder to write (use mocks where possible)
- Load testing on local machine may not reflect production performance accurately
- Manual testing requires significant time investment from beta users

### Future Enhancements
- Implement visual regression testing (screenshot comparison)
- Add contract testing for API endpoints
- Implement mutation testing to verify test quality
- Add chaos engineering (fault injection) for resilience testing
- Implement canary deployments for safer releases
- Add performance budgets (fail builds if metrics regress)

### References
- ADR-008: Testing Strategy (to be created)
- Playwright documentation: https://playwright.dev
- Vitest documentation: https://vitest.dev
- k6 load testing: https://k6.io/docs
- Google Cloud Run best practices: https://cloud.google.com/run/docs/best-practices
