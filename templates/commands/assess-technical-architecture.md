You are a senior software architect tasked with understanding the technical architecture and implementation details of an application. Your goal is to extract and document the technical structure, patterns, dependencies, and architectural decisions from the provided codebase to create a comprehensive technical reference.

## Instructions

Carefully analyze the provided codebase, focusing on the technical implementation, architecture patterns, and structural decisions, and create a comprehensive document (at `.specify/memory/technical-architecture.md`) that outlines the technical architecture and serves as a definitive technical reference for the solution.

### Key Areas to Focus On

1. **Application Architecture Pattern**: Identify the overall architectural pattern being used (MVC, Clean Architecture, Onion Architecture, etc.). How is the application structured and layered?
2. **Project Structure and Dependencies**: Document the solution structure, project dependencies, and how different projects/assemblies relate to each other. What is the dependency graph?
3. **Data Access Layer**: Analyze how data is accessed and managed. What ORM is used? How are database connections handled? What are the data access patterns? What are the key entities and their relationships?
4. **Service Layer Architecture**: Identify service classes, their responsibilities, and how they interact. What patterns are used for dependency injection and service registration?
5. **External Dependencies and Integrations**: Document third-party libraries, package dependencies, external APIs, and integration patterns. How are external systems accessed?
6. **Configuration Management**: How is configuration handled? What configuration sources are used (config files, environment variables, etc.)?
7. **Cross-Cutting Concerns**: How are logging, error handling, security, caching, and other cross-cutting concerns implemented?
8. **Communication Patterns**: How do different parts of the application communicate? Are there internal APIs, message queues, or event-driven patterns?
9. **Authentication and Authorization**: What security frameworks and patterns are implemented? How are users authenticated and authorized?
10. **Background Processing**: Are there background services, scheduled jobs, or async processing patterns? How are they implemented?
11. **Testing Architecture**: What testing strategies are in place? How is the code structured to support testing?
12. **Deployment and Infrastructure**: What deployment patterns are evident from the code? Are there containerization, cloud-specific implementations, or infrastructure as code patterns?

### Technical Analysis Focus

1. **Code Organization**: Analyze namespaces, folder structures, and how code is organized across projects. Look for:
   - Controllers and their dependencies
   - Shared utility classes and common libraries
   - Service class organization and responsibilities
   - Dependencies between projects and modules

2. **Design Patterns**: Identify design patterns used throughout the codebase (Repository, Factory, Strategy, etc.). Pay attention to:
   - Repository patterns and data access abstractions
   - Factory patterns and object creation strategies
   - Observer/Event patterns and messaging approaches

3. **Dependency Injection**: Document the DI container usage and service lifetimes. Look for:
   - Singleton services and their responsibilities
   - Scoped services and their transaction boundaries
   - Service dependency graphs and relationships

4. **Database Schema and Migrations**: Understand data models and how schema changes are managed. Examine:
   - Entity relationships and data model structure
   - Table relationships and foreign key dependencies
   - Aggregate roots and domain boundaries
   - Migration patterns and versioning strategies

5. **API Design**: Analyze REST API structure, routing patterns, and serialization approaches. Look for:
   - Controller groupings and API organization
   - DTOs and data transfer patterns
   - API versioning strategies
   - Endpoint structure and call patterns

6. **Security Implementation**: Analyze authentication, authorization, input validation, and security headers. Look for:
   - Middleware that handles security concerns
   - Authorization policies either in configuration or code
   - Claims-based authorization patterns
   - Input validation strategies and their impact on business logic

7. **Performance Considerations**: Identify caching strategies, async patterns, and performance optimizations
8. **Error Handling Strategies**: Document exception handling patterns and error propagation

### Architecture Analysis

1. **Domain Boundaries**: Identify logical boundaries within the application. Look for:
   - Domain aggregates and bounded contexts
   - Module and component separation
   - Feature organization and groupings
   - Business capability alignment

2. **Shared Components**: Document shared libraries, utilities, and common modules
3. **Data Relationships**: Analyze database dependencies and data organization:
   - Tables that naturally group together by domain
   - Shared data and cross-cutting data concerns
   - Master-detail relationships and data hierarchies
   - Reference data and lookup tables

4. **Communication Patterns**: Document how components communicate internally and externally
5. **Transaction Management**: Understand current transaction patterns. Analyze:
   - Transaction management patterns and boundaries
   - Database transactions and their scope
   - Consistency requirements and strategies
   - Error handling and rollback patterns

## Deliverables

Create a comprehensive document in the `.specify/memory/` directory with the following Markdown structure:

```markdown
# Technical Architecture Analysis: [Application Name]

## Executive Summary

High-level overview of the technical architecture, main technologies used, and architectural patterns employed.

## Technology Stack

### Core Technologies
- Programming Languages and versions
- Frameworks and runtime versions
- Database(s)
- Web Framework
- Key Libraries and Frameworks

### External Dependencies
- Third-party packages
- External APIs
- Infrastructure dependencies

## Architecture Overview

### High-Level Architecture Diagram
[Insert Mermaid diagram showing main architectural components and their relationships]

### Architecture Pattern
Description of the overall architectural approach (layered, clean architecture, etc.)

## Project Structure

### Solution Organization
- Project/module breakdown and responsibilities
- Inter-project/module dependencies
- Package/module boundaries

### Dependency Graph
[Insert Mermaid diagram showing project dependencies]

## Data Architecture

### Data Access Patterns
- ORM/Data access technology
- Repository patterns
- Database connection management
- Entity models and relationships

### Database Design
- Schema overview
- Migration strategies
- Data partitioning considerations

## Service Architecture

### Service Layer Organization
- Service classes and their responsibilities
- Dependency injection patterns
- Service class dependencies and layering
- Service lifetimes and scoping

### Business Logic Distribution
- Where business rules are implemented
- Service boundaries and interactions

## Cross-Cutting Concerns

### Logging and Monitoring
- Logging framework and patterns
- Monitoring and telemetry

### Security Implementation
- Authentication mechanisms
- Authorization patterns
- Security middleware and policies

### Error Handling
- Exception handling strategies
- Error propagation patterns

### Configuration Management
- Configuration sources and patterns
- Environment-specific settings

## Communication Patterns

### Internal Communication
- How components communicate within the application
- Event handling patterns
- Message passing mechanisms

### External Communication
- API design and patterns
- External service integration
- Communication protocols used

## Performance and Scalability

### Caching Strategies
- Caching layers and implementations
- Cache invalidation patterns

### Asynchronous Processing
- Async/await patterns
- Background processing
- Queue-based processing

## Testing Architecture

### Test Organization
- Unit test structure and patterns
- Integration test approaches
- Test isolation strategies

## Architecture Analysis

### Domain Boundaries
- Identified bounded contexts and aggregates
- Module and component separation
- Feature organization patterns

### Shared Components
- Common libraries and utilities
- Shared data models and DTOs
- Cross-cutting modules

### Data Organization
- Database schema structure and relationships
- Data domain groupings
- Reference data and lookup tables
- Data consistency patterns

### Infrastructure Considerations

#### Containerization and Orchestration
- Current containerization patterns (Docker, container images)
- Orchestration patterns and configurations
- Container registry and image management strategies
- Resource allocation and scaling patterns

#### Networking and Communication
- API Gateway patterns (if applicable)
- Authentication patterns (mTLS, JWT, etc.)
- Resilience patterns (circuit breakers, retries)
- Load balancing and routing configurations

#### Deployment and DevOps
- CI/CD pipeline structure and workflows
- Database migration strategies
- Deployment strategies (blue-green, rolling, etc.)
- Feature flag and release patterns

#### Monitoring and Observability
- Tracing and correlation patterns
- Logging aggregation and analysis
- Health checks and monitoring endpoints
- Performance monitoring strategies

#### Configuration and Secrets Management
- Centralized vs. distributed configuration approaches
- Secret management and rotation strategies
- Environment-specific configuration distribution
- Service discovery and registration patterns

## Technical Debt and Improvement Opportunities

### Code Quality Issues
- Areas with potential for improvement
- Tightly coupled components
- Violations of architectural principles

### Performance Considerations
- Identified bottlenecks or concerns
- Resource utilization patterns
- Scalability considerations

## Observations and Notes

### Architectural Strengths
- Well-designed aspects of the architecture
- Effective patterns and practices in use

### Areas of Concern
- Potential risks or issues identified
- Technical debt observations
- Recommendations for investigation
```

## Important Guidelines

- Focus on technical implementation details and architectural decisions
- Use technical terminology appropriate for software architects and senior developers
- Provide specific code examples and patterns where relevant
- Document the architecture as it exists, noting both strengths and areas of concern
- Include diagrams using Mermaid syntax where helpful for visualization
- Consider current architecture patterns and their effectiveness
- Document assumptions about technical decisions where code intent is unclear
- Provide a comprehensive reference that can inform future technical decisions

**Please analyze the provided codebase and create this technical architecture analysis document (`.specify/memory/technical-architecture.md`) as a markdown file.** Ensure that the document provides comprehensive technical insights that serve as a definitive reference for understanding the solution's architecture.
