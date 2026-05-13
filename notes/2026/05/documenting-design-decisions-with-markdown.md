---
title: "Documenting Design Decisions with Markdown"
slug: "documenting-design-decisions-with-markdown"
type: "concept"
tags: ["documentation", "design-principles", "markdown", "codebase-standards", "communication"]
summary: "This note explains how to use Markdown to effectively document design decisions and principles directly within a codebase for clarity and maintainability."
created: 2026-05-13
updated: 2026-05-13
source_question: "What is awesome design md inculcate it in my codebase. Check and understand"
links:
  - slug: "improving-code-reading-debugging-pr-review-skills"
    relation: "related"
    why: ""
  - slug: "authentication-tokens-bearer-tokens-and-sso"
    relation: "related"
    why: ""
review:
  last_reviewed: null
  next_review: 2026-05-13
  step: 0
  confidence: 0
quiz:
---

## Mental model
Imagine a codebase as a complex machine. The code itself shows *how* the machine works, but it doesn't always explain *why* certain parts are shaped the way they are, or *what problems* they were built to solve. Design documentation, especially when written in Markdown and kept alongside the code, acts as the "owner's manual" for these decisions. It captures the context, constraints, alternatives considered, and the rationale behind significant architectural or implementation choices, making the codebase much easier for current and future engineers to understand, debug, and evolve. It's about encoding the "why" next to the "what."

## Diagram
```mermaid
flowchart TD
    A[New Feature / Problem] --> B{Design Discussion / Research};
    B --> C{Decision Made};
    C --> D[Document Decision in Markdown];
    D -- (e.g., `docs/decisions/feature-x-arch.md`) --> E[Implement Code];
    E -- (references D) --> F[Codebase];
    F --> G{Future Engineer / Debugger};
    G --> H[Reads Code];
    H -- (needs context) --> I[Consults Markdown Design Doc];
    I --> J[Understands "Why"];
```

## Prerequisites
1.  **Basic Markdown syntax:** Understanding how to use headings, lists, code blocks, and links in Markdown.
2.  **Version Control (e.g., Git):** Familiarity with how to add, commit, and push files in a repository, as design docs live alongside code.
3.  **Software Design Principles:** A general understanding that design involves making choices, considering trade-offs, and solving problems.

## How it actually works
Documenting design decisions with Markdown involves creating dedicated files within your codebase's repository to capture the rationale behind significant choices. This makes the documentation discoverable and version-controlled alongside the code it describes.

1.  **Identify a significant decision:** Not every line of code needs a design doc. Focus on architectural choices, complex algorithms, major API designs, tricky integration points, or anything that might puzzle a future reader.
2.  **Choose a location:** A common pattern is to have a `docs/` directory at the root of your project, often with subdirectories like `docs/architecture/`, `docs/decisions/`, or `docs/adr/` (Architectural Decision Records).
3.  **Create a Markdown file:** Name it descriptively, e.g., `docs/decisions/2023-10-27-user-auth-strategy.md` or `docs/architecture/payment-gateway-integration.md`.
4.  **Structure the document:** Use Markdown headings to organize information. A typical structure might include:
    *   **Title:** Clear and concise name for the decision.
    *   **Status:** (e.g., "Accepted", "Proposed", "Deprecated")
    *   **Context:** What problem are we trying to solve? What are the existing conditions or constraints?
    *   **Decision:** What is the chosen solution or approach?
    *   **Alternatives Considered:** What other options were explored?
    *   **Pros and Cons (for each alternative and the chosen decision):** Why was the chosen path better than others? What are its drawbacks?
    *   **Consequences:** What are the implications of this decision for future development, performance, security, etc.?
    *   **References:** Links to related tickets, research, or other documentation.
5.  **Write the content:** Clearly articulate the "why" behind the decision. Be objective, concise, and provide enough detail for someone unfamiliar with the context to understand.
6.  **Commit and push:** Treat design documentation like code. It's part of the codebase and should be version-controlled. This ensures it evolves with the project and is available to everyone.
7.  **Reference in code/PRs:** Link to these Markdown docs from relevant code comments, READMEs, or pull request descriptions to guide readers. This practice significantly improves [[improving-code-reading-debugging-pr-review-skills]].

## Two examples

### Example 1 — canonical: Architectural Decision Record (ADR)
A common pattern for documenting specific decisions.

```markdown
# ADR 0005: Choose JWT for API Authentication

## Status
Accepted

## Context
Our existing API uses session-based authentication, which couples user state to the server and makes horizontal scaling more complex. We are building a new mobile application and need a stateless authentication mechanism that is scalable, secure, and compatible with microservices architecture. The mobile app will communicate directly with our API gateway.

## Decision
We will adopt JSON Web Tokens (JWTs) as the primary authentication mechanism for our API. Upon successful login, the server will issue a JWT, which the client will store and send with subsequent requests in the `Authorization: Bearer <token>` header. The API gateway will validate the JWT's signature and expiration before forwarding the request.

## Alternatives Considered

### 1. OAuth 2.0 (Full Flow)
*   **Pros:** Industry standard, robust, supports delegated authorization.
*   **Cons:** More complex to implement for simple authentication, often overkill when only authentication (not delegated authorization) is needed for our own first-party clients.

### 2. API Keys
*   **Pros:** Simple to implement.
*   **Cons:** Less secure (easy to compromise if leaked), no built-in expiration or revocation, difficult to manage per-user access.

### 3. Session Tokens (with shared state)
*   **Pros:** Familiar, easy to invalidate.
*   **Cons:** Requires shared session storage (e.g., Redis) across instances, harder for horizontal scaling, not ideal for mobile clients without cookies.

## Consequences
*   **Positive:** Improved scalability for API, stateless server design, better support for mobile clients and microservices.
*   **Negative:** JWTs are stateless, so explicit revocation (e.g., for logout or compromise) requires additional mechanisms (e.g., a blocklist/revocation list or short expiry with refresh tokens). Increased payload size slightly due to token.
*   **Future Work:** Implement refresh token mechanism for better UX and security. Consider token rotation.

## References
*   [[authentication-tokens-bearer-tokens-and-sso]]
*   Jira Ticket: AUTH-123 - Implement JWT for new mobile API
```

### Example 2 — wrong-but-tempting / edge case: Undocumented "obvious" choice
A common pitfall is to skip documenting a decision because it feels "obvious" at the time.

**Problem:** A developer integrates a third-party payment gateway, choosing Stripe because "everyone uses it." They don't document *why* Stripe was chosen over PayPal, Braintree, or a custom solution.

**Consequence:** Six months later, a new developer needs to add a feature that requires a specific payment method not supported by Stripe in their region. They spend days researching alternatives, only to find that PayPal was explicitly rejected earlier due to high transaction fees (a detail now lost). If the original decision had been documented, it would have saved significant time and prevented potentially re-evaluating already discarded options.

## Why it's written this way
Markdown is chosen for design documentation for several reasons:
*   **Simplicity:** It's a lightweight, human-readable syntax that's easy to learn and write.
*   **Version Control Friendly:** Plain text files are ideal for Git. They produce clean diffs, making it easy to track changes, review updates, and see the history of a design decision.
*   **Tool Agnostic:** Markdown can be viewed and rendered by almost any text editor, IDE, or web browser without special software.
*   **Proximity to Code:** Keeping documentation in Markdown files directly within the repository means it lives alongside the code it describes. This makes it easier to find, update, and ensures it's part of the same review and deployment lifecycle.
*   **Focus on Content:** Its minimal formatting encourages writers to focus on the clarity and substance of the design explanation rather than elaborate presentation.

Alternatives like Confluence, Notion, or Google Docs are powerful but often live outside the codebase's version control, leading to documentation drift where the external document becomes outdated relative to the code. While useful for broader project management, they are less ideal for detailed, version-controlled design decisions tied directly to implementation.

## Failure modes
1.  **Documentation Drift:** Design documents become outdated because they are not updated when the code changes, leading to misleading information.
2.  **Over-documentation:** Documenting every trivial decision, creating a burden that discourages future documentation efforts and makes it hard to find important information.
3.  **Under-documentation:** Critical architectural decisions are never written down, leading to loss of institutional knowledge, repeated mistakes, and difficulty onboarding new team members.
4.  **Inconsistent Structure:** Lack of a standardized template for design docs makes them harder to read and compare, reducing their utility.
5.  **Hidden Docs:** Placing design documents in obscure locations or not linking them from relevant code or READMEs, making them undiscoverable.

## Quiz

### Q1
Consider a scenario where your team decides to switch from a monolithic architecture to microservices. What specific sections would be most critical to include in a Markdown design document for this decision, and why?
**Answer:** The most critical sections would be:
*   **Context:** Clearly explain *why* the monolithic architecture is no longer sufficient (e.g., scaling issues, team autonomy, technology lock-in).
*   **Decision:** State the chosen microservices approach, perhaps mentioning specific patterns (e.g., API Gateway, service discovery).
*   **Alternatives Considered:** Briefly describe other options (e.g., modular monolith, serverless functions) and why they were rejected.
*   **Consequences:** Detail the significant impact on deployment, operations, team structure, data consistency, and potential new challenges (e.g., distributed tracing, inter-service communication).
These sections explain the fundamental problem, the chosen solution, why it was chosen, and what its long-term effects will be, which is vital for such a large architectural shift.

### Q2
Why is the "Status" field (e.g., "Accepted," "Proposed," "Deprecated") included in the example ADR? Why not just have the document exist once it's written?
**Answer:** The "Status" field is crucial because design decisions are not always final or immediately implemented. It provides immediate context on the decision's lifecycle. "Proposed" indicates it's under review, "Accepted" means it's the current plan, and "Deprecated" signals that the decision is no longer valid or has been superseded. Without it, a reader might assume an old document still represents the current state, leading to confusion or incorrect implementations. It helps manage the evolution of design over time.

### Q3
If a team consistently updates their Markdown design documents but never links them from relevant code comments or READMEs, what is the likely failure mode, and what impact would it have on a new engineer trying to understand the codebase?
**Answer:** The likely failure mode is **Hidden Docs**. Even if the documentation is perfectly maintained, if it's not discoverable, it's effectively useless. A new engineer would likely struggle significantly. They would read the code and, without explicit pointers to the design decisions, would have to guess the "why" behind complex implementations. This would lead to slower onboarding, increased time spent debugging (because the context for decisions is missing), and potentially incorrect assumptions about how to extend or modify the system. The value of the documentation is lost if it cannot be easily found and referenced.

## Links
- **Related:** [[improving-code-reading-debugging-pr-review-skills]] — Good design documentation directly supports improved code reading, debugging, and PR review by providing essential context and rationale for code decisions.
