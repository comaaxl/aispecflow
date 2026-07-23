# Common Document Types - Reference Guide

This is a **reference**, not a rigid template. When producing a document, use this guide to check that key questions have been covered and to see structure suggestions. Adapt freely to the project's actual needs. The user may ask for document types not listed here - use your judgment and follow professional engineering conventions.

---

## Requirements (需求规格文档)

**Audience**: Engineering team, spec pipeline
**Purpose**: Structured contract - precise, unambiguous, machine-consumable

**Key questions to cover**:
- What problem does this solve? Who has this problem?
- What are the functional and non-functional requirements?
- What is explicitly in scope vs. out of scope?
- What are the constraints (technical, business, existing system)?
- What are the risks and mitigations?
- How do we know this is done? (success criteria)

**Structure suggestion**:
```
# Requirements: <Feature/Change Name>
## Problem Statement
## Users & Stakeholders
## Functional Requirements
## Non-Functional Requirements
## Success Criteria
## Scope Boundaries (In / Out / Assumptions)
## Risks & Mitigations
## Dependencies
```

---

## PRD (产品需求文档)

**Audience**: Business stakeholders, managers, new team members
**Purpose**: Narrative document - accessible, context-rich, human-facing

**Key questions to cover**:
- What is the background and motivation? Why now?
- How does the current state work? What's broken or missing?
- What are the goals and explicit non-goals?
- Who are the users? What are their scenarios and workflows?
- What is the high-level solution approach?
- How will success be measured?

**Structure suggestion**:
```
# PRD: <Feature/Change Name>
## Background & Motivation
## Current State
## Goals & Non-Goals
## Users & Scenarios
## Solution Overview
## Success Metrics
## Risks & Dependencies
```

---

## Technical Architecture (技术架构文档)

**Audience**: Engineering team
**Purpose**: Describe system structure, component design, and technical decisions

**Key questions to cover**:
- What is the current architecture? (if existing project)
- What are the major components and their boundaries?
- What is the data model? How is data stored and accessed?
- What technology choices are being made and why? (trade-offs)
- How is the system deployed? (topology, environments)
- How does the system scale? What about failure/disaster recovery?
- How do components communicate? (sync, async, protocols)

**Structure suggestion**:
```
# Technical Architecture: <Project/Feature Name>
## Architecture Overview (with diagram if possible)
## Component Design
## Data Model
## Technology Selection & Trade-offs
## Communication & Integration
## Deployment Topology
## Scalability & Fault Tolerance
```

---

## API Design (接口文档)

**Audience**: Frontend and backend developers, integration partners
**Purpose**: Define the contract between systems

**Key questions to cover**:
- What are all the endpoints/interfaces?
- What are the request/response formats for each?
- How is authentication handled?
- What is the error code system?
- What are the data models/schemas?
- Is there a versioning strategy?
- Are there rate limits or quotas?

**Structure suggestion**:
```
# API Design: <Project/Feature Name>
## Authentication & Authorization
## Endpoints
### <Method> <Path>
  - Description
  - Request (params, body, schema)
  - Response (success, error codes)
  - Example
## Error Codes
## Versioning Strategy
```

---

## UI Prototype (原型设计)

**Audience**: Product, design, stakeholders, developers
**Purpose**: Validate page structure, user flows, and interaction patterns before full development

**Key questions to cover**:
- What are the key pages/screens?
- What is the user flow? (entry -> action -> outcome)
- What is the information architecture? (navigation, page hierarchy)
- What interaction patterns are used? (forms, modals, lists, etc.)
- What is the visual style? (color scheme, density, tone)
- What states need to be handled? (loading, empty, error, success)

**Output form**: Depends on what the user wants and what's practical:
- Runnable frontend code (HTML/CSS/JS) - most useful for validation
- Page structure description + wireframe descriptions
- Component breakdown + interaction flow descriptions

**Structure suggestion (for description-based output)**:
```
# UI Prototype: <Project/Feature Name>
## Page Structure & Navigation
## User Flows
### Flow 1: <name>
  - Entry -> Step 1 -> Step 2 -> Outcome
## Page Details
### <Page Name>
  - Layout description
  - Key components
  - States (loading / empty / error / success)
## Visual Style
## Interaction Patterns
```

---

## Custom Document Types

If the user asks for a document type not listed above, use your judgment:
1. Identify the audience and purpose
2. Determine what key questions need to be answered
3. Follow professional engineering conventions for that document type
4. Produce it with an appropriate structure

The value of grilling is the same regardless of document type - eliminate ambiguity, resolve contradictions, sharpen fuzzy language, then produce.
