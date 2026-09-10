---
title: "1.3. week 3 worklog"
date: 2026-09-10
weight: 3
chapter: false
---

# Week 3: Backend Development and Containerization

### Week 3 Objectives:

- Configure Spring Boot application
- Set up Spring Data JPA and database mappings
- Develop API endpoints
- Configure Docker for application
- Build and test Docker Image locally
- Optimize Docker Image size

### Tasks to be accomplished this week:

| No. | Task | Start Date | Completion Date | Resource |
|-----|------|-----------|-----------------|----------|
| 2 | Configure Spring Boot application properties | 27/04/2026 | 27/04/2026 | IntelliJ IDEA |
| 3 | Set up Spring Data JPA and database mappings | 28/04/2026 | 28/04/2026 | Spring Documentation |
| 4 | Develop REST API endpoints | 29/04/2026 | 29/04/2026 | Spring Boot Guide |
| 5 | Create Dockerfile and build Docker Image | 30/04/2026 | 30/04/2026 | Docker Documentation |
| 6 | Test Docker Image locally and optimize | 01/05/2026 | 01/05/2026 | Docker Best Practices |

### Results achieved in Week 3:

**Spring Boot Configuration:**
- Successfully configured Spring Boot application with AWS RDS connection
- Set up application.yml with environment-specific properties
- Configured Spring Data JPA with Hibernate ORM
- Created database entity classes with proper annotations

**API Development:**
- Developed complete REST API endpoints for core functionality
- Implemented Spring Security for authentication
- Set up JWT token-based authorization
- Created request/response DTOs with validation

**Spring Data JPA:**
- Created repository interfaces for database operations
- Implemented custom query methods
- Configured relationship mappings (One-to-Many, Many-to-Many)
- Set up lazy loading configuration

**Docker Implementation:**
- Created optimized Dockerfile with multi-stage build
- Configured Docker entrypoint for application startup
- Set up environment variables for configuration
- Optimized Docker Image size from ~800MB to ~250MB

**Local Testing:**
- Built Docker Image successfully on development machine
- Ran Docker container locally with RDS connection
- Tested API endpoints with curl and Postman
- Verified database operations from containerized application
- Fixed connection issues and optimized performance

