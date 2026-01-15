# Project Brief: OpenHorizon

**Created:** 2026-01-15 (Stockholm time)
**Last Updated:** 2026-01-15
**Status:** Active
**Repository:** https://github.com/gpt153/openhorizon.cc
**Implementation Workspace:** `/home/samuel/.archon/workspaces/openhorizon.cc/`
**Planning Workspace:** `/home/samuel/supervisor/openhorizon/`

---

## Vision

OpenHorizon is an AI-powered platform that democratizes the creation of high-quality Erasmus+ youth exchange project applications. By combining intelligent brainstorming, conversational elaboration, and automated project generation with comprehensive budget and logistics planning, OpenHorizon empowers youth organizations to design impactful international programs without requiring deep expertise in Erasmus+ application processes.

---

## Goals

### Primary Goals
1. **Simplify Erasmus+ Application Creation:** Reduce the time and expertise required to create competitive Erasmus+ Youth Exchange applications from weeks to days through AI-assisted planning and document generation.

2. **Support Real-World Deployments:** Successfully support 3-5 organizations in submitting real Erasmus+ applications by February 2026, validating the platform with actual users and projects.

3. **Deliver Complete Project Management:** Provide end-to-end support from initial brainstorming through program design, budget planning, vendor coordination, and final application document generation.

### Success Criteria
- [x] Criterion 1: Core feature set complete (Seed generation, elaboration, project conversion, programme builder, budget tracking, document export)
- [ ] Criterion 2: Platform successfully generates 3-5 real Erasmus+ application documents by February 2026
- [ ] Criterion 3: User feedback validates that OpenHorizon reduces application creation time by >60% compared to manual processes
- [ ] Criterion 4: System achieves <5% error rate on AI-generated content requiring manual correction
- [ ] Criterion 5: Production deployment stable with <1% downtime

---

## Stakeholders

### Primary Stakeholders
- **Youth Organization Coordinators:** Project managers at NGOs and educational institutions who need to create Erasmus+ applications but lack deep expertise in EU funding processes. Need intuitive tools that guide them through complex requirements.
- **Youth Workers:** Practitioners designing educational programs who need help structuring activities, learning objectives, and programme schedules. Need creative support and validation of pedagogical approaches.
- **Financial Administrators:** Budget managers who need accurate cost planning, vendor management, and compliance with Erasmus+ financial rules. Need transparent calculations and export-ready documents.

### Decision Makers
- **Owner:** Samuel (gpt153)
- **Technical Lead:** SCAR (AI implementation agent)
- **Supervisor:** Claude Supervisor (planning & orchestration)

---

## Scope

### In Scope (Completed ~91%)
- **Seed Management:** AI-powered brainstorming, seed garden organization, seed versioning
- **Conversational Elaboration:** Interactive chat-based refinement of project ideas with AI guidance, metadata extraction, progress tracking
- **Project Generation:** Automated creation of project DNA, objectives, participant profiles, budgets from elaborated seeds
- **Programme Builder:** Multi-day activity planning with session-level detail, morning/afternoon/evening focus areas, learning objectives integration
- **Project Pipeline Management:**
  - AI-powered Travel, Food, and Accommodation vendor research
  - Budget calculator with vendor quote tracking
  - Communication template generation
  - Timeline/Gantt visualization
- **Document Export:** PDF and DOCX generation for application forms and project reports
- **Multi-tenant Architecture:** Organization-based tenancy with role-based access control (OWNER, ADMIN, STAFF, PARTNER, PARTICIPANT, GUARDIAN)
- **Authentication:** Clerk-based user management
- **Dual-mode Content:** Working language (internal collaboration) and formal language (application documents)

### In Scope (Remaining ~9%)
- **Bug Fixes:**
  - API timeout issues on Food/Accommodation agent searches (30-second limit)
  - Authentication endpoint 500 errors on registration
  - Database migration application in production
- **Production Readiness:**
  - Performance optimization for AI agent timeouts
  - Comprehensive E2E test coverage
  - Monitoring and error tracking
  - User onboarding flow refinement

### Out of Scope (Explicitly)
- **Non-Erasmus+ Programs:** Initial release focused exclusively on Erasmus+ Youth Exchanges (Action Key 1). Other program types (EVS, adult education) excluded to maintain focus.
- **Automatic Application Submission:** System generates documents but does not directly submit to EU portals due to liability and compliance complexity.
- **Real-time Collaboration:** Multi-user live editing deferred to Phase 2 to reduce technical complexity.
- **Mobile Native Apps:** Web-first approach; native iOS/Android apps excluded to focus on core functionality.
- **Multi-language UI:** English-only interface initially (though projects support multiple working/formal languages).

---

## Technical Context

### Technology Stack
- **Language:** TypeScript 5.9.3
- **Frontend Framework:** Next.js 16.0.10 with React 19.2.1
- **Backend Framework:** Next.js API Routes (tRPC) + Fastify 5.1.0 (separate project-pipeline service)
- **Database:** PostgreSQL via Supabase (pooler mode) + Prisma ORM 6.0.0
- **Styling:** Tailwind CSS 4 with Radix UI component library (20+ primitives)
- **State Management:** Zustand (local) + React Query v5 (server state)
- **Authentication:** Clerk
- **AI/LLM:** LangChain with Anthropic Claude (claude-sonnet-4-5-20250929) and OpenAI (fallback)
- **Background Jobs:** Inngest (event-driven job queue)
- **Infrastructure:** Docker + Google Cloud Run
- **CI/CD:** Git-based deployment
- **Testing:** Playwright (E2E), Vitest (unit tests)
- **Email:** SendGrid

### Architecture Patterns
- **Monorepo Structure:** Single repository with workspaces (landing + app + project-pipeline)
- **tRPC for Type-Safe APIs:** End-to-end TypeScript type safety between frontend and backend
- **Orchestrator Component Pattern:** Complex UIs (ConversationalElaboration) coordinate multiple pure sub-components
- **AI Agent Pattern:** Specialized LangChain agents for domain-specific searches (Travel, Food, Accommodation)
- **Multi-tenant with Row-Level Security:** Organization-based data isolation via Prisma middleware
- **Event-Driven Background Processing:** Long-running AI generation tasks handled via Inngest
- **Dual-mode Content Storage:** Separate columns for working vs. formal language in database

### Integrations
- **Anthropic Claude API:** Primary LLM for project generation and conversational elaboration
- **OpenAI API:** Fallback LLM and embeddings
- **Clerk Authentication:** User management and SSO
- **Supabase PostgreSQL:** Managed database hosting
- **SendGrid:** Transactional email delivery
- **Inngest:** Background job orchestration
- **Google Cloud Run:** Container deployment platform

---

## Constraints

### Technical Constraints
- **AI API Rate Limits:** Anthropic and OpenAI rate limits require careful queue management for batch operations
- **30-Second Cloud Run Timeout:** HTTP requests limited to 30 seconds; long-running AI operations must use background jobs (current issue with Food/Accommodation agents)
- **Database Connection Pooling:** Supabase connection limits require pooler mode and careful connection management
- **Cold Start Latency:** Cloud Run cold starts can add 2-5 seconds to initial requests

### Business Constraints
- **February 2026 Launch Deadline:** Hard deadline to support real Erasmus+ applications for spring 2026 submission round
- **Bootstrap Budget:** Minimal external funding; infrastructure costs kept under €100/month
- **Erasmus+ Compliance:** Generated documents must meet official Erasmus+ Youth Exchange application requirements

### Resource Constraints
- **Team Size:** Solo developer (Samuel) + AI assistants (SCAR for implementation, Supervisor for planning)
- **Time:** Part-time development alongside other commitments
- **Support Capacity:** Limited ability to provide hands-on support to users; platform must be self-service

---

## Current Status

### Phase
**Implementation (91% complete) → Testing & Deployment**

### Recent Progress
- **2026-01-15:** Comprehensive codebase research completed, planning artifacts backfill initiated
- **2026-01:** Conversational seed elaboration UI (#104) implemented
- **2026-01:** Backend project generation engine (#103) completed
- **2026-01:** Database schema for intelligent seed elaboration (#102) deployed
- **2025-12:** Conversational seed elaboration agent (#101) implemented
- **2025-Q4:** Core feature set completed (seed management, project generation, programme builder, pipeline)

### Next Milestones
- [ ] **Milestone 1:** Fix API timeout issues on Food/Accommodation agents - Target: 2026-01-22
- [ ] **Milestone 2:** Resolve authentication endpoint 500 errors - Target: 2026-01-22
- [ ] **Milestone 3:** Complete E2E test coverage (>80% critical paths) - Target: 2026-01-29
- [ ] **Milestone 4:** Production deployment validation - Target: 2026-02-05
- [ ] **Milestone 5:** Onboard first 3-5 partner organizations - Target: 2026-02-15

---

## Risks

### High-Priority Risks
1. **AI Agent Timeout Failures**
   - **Impact:** Food and Accommodation agents hitting 30-second timeout prevent users from completing budget planning, blocking full application workflow
   - **Mitigation:** Convert synchronous API calls to Inngest background jobs with polling/webhook updates; implement caching for common searches; add user-friendly timeout handling with partial results

2. **February 2026 Deadline Pressure**
   - **Impact:** Insufficient time to properly test and validate with real users could result in buggy applications or missed submission deadlines for partner organizations
   - **Mitigation:** Prioritize ruthlessly using MoSCoW method; focus remaining development on critical path bugs only; establish early beta testing with 1-2 friendly users in January

3. **Authentication System Instability**
   - **Impact:** 500 errors on registration prevent new users from accessing the platform, blocking adoption and testing
   - **Mitigation:** Debug and fix Clerk integration issues immediately; implement comprehensive error logging; add fallback authentication method if Clerk proves unstable

4. **Database Migration Drift**
   - **Impact:** Production database schema may be out of sync with codebase, causing runtime errors or data corruption
   - **Mitigation:** Audit current production schema vs. Prisma migrations; create migration reconciliation plan; implement automated migration checks in CI/CD

5. **AI Cost Overruns**
   - **Impact:** High-volume AI generation could exceed budget constraints if not carefully monitored
   - **Mitigation:** Implement usage quotas per organization tier (FREE/BASIC/PRO); add cost monitoring and alerts; optimize prompts to reduce token usage; cache common AI responses

---

## Related Documents

- **PRDs:** `.bmad/prd/` (To be created)
- **Epics:** `.bmad/epics/` (See: 001-fix-api-timeouts.md, 002-auth-stability.md)
- **ADRs:** `.bmad/adr/` (See: 001-tech-stack.md, 002-ai-architecture.md, 003-multi-tenancy.md)
- **Architecture:** `.bmad/architecture/` (To be documented)
- **Workflow Status:** `.bmad/workflow-status.yaml`

---

## Notes

### Historical Context
OpenHorizon was conceived to address a recurring challenge observed in the youth work sector: highly motivated organizations with strong educational ideas struggle to navigate the complex Erasmus+ application process. The bureaucratic complexity, detailed budget requirements, and formal documentation standards create significant barriers to entry, particularly for smaller NGOs and grassroots initiatives.

### Implementation Approach
The project has followed a pragmatic "build then document" approach, with SCAR implementing features rapidly based on user needs. This planning artifacts backfill represents a strategic pause to formalize documentation, create epics for remaining work, and establish systematic oversight before the February 2026 production deadline.

### Deployment History
- **Domain:** oh.153.se (app.openhorizon.cc)
- **Hosting:** Google Cloud Run (frontend + backend services)
- **Database:** Supabase PostgreSQL (pooler mode for connection management)
- **Current Version:** Pre-1.0 (beta phase, ~91% feature complete)

### User Feedback Integration
Early informal testing has validated core assumptions:
- Users appreciate the conversational elaboration approach vs. blank forms
- AI-generated programme structures save significant time but require human review
- Budget planning is the most time-consuming step (hence urgency of fixing agent timeouts)
- Document export quality meets Erasmus+ application standards

---

**Template Version:** 1.0
**Template Source:** BMAD-inspired project brief for SCAR supervisor
**Document Status:** Complete and validated against implementation
