# AGENTS.md

This file provides instructions to coding agents working in the ADS Toolkit repository.

## Project mission

ADS Toolkit is a collection of focused web utilities supporting Assessment Development workflows.

The primary users are a small Assessment Development team. However, tools should be institution-agnostic whenever the underlying workflow is generic.

Do not introduce organization-specific assumptions, branding, domains, identifiers, URLs, credentials, or business rules unless a specific requirement calls for them.

## Core architecture

ADS Toolkit is:

- SvelteKit
- TypeScript
- Bun-based
- Tailwind CSS
- Statically deployed
- Browser-first
- Local-first
- Privacy-focused

The project uses SvelteKit's static adapter.

Assume there is **no application server** unless the task explicitly changes that architectural decision.

## Privacy requirements

Privacy is an architectural requirement.

By default:

- Process user files entirely within the browser.
- Do not upload file contents.
- Do not transmit user data to third parties.
- Do not add telemetry or analytics.
- Do not persist uploaded file contents.
- Do not place sensitive content into `localStorage`.
- Do not add databases.
- Do not add application authentication.
- Do not add external APIs simply for convenience.
- Do not introduce server-side processing when browser APIs can perform the operation.

Harmless user preferences may be persisted locally when useful.

If a requested feature would require sending user content over a network, storing user content, using credentials, or introducing server infrastructure, treat this as an architectural boundary. Make the requirement explicit rather than quietly introducing network behavior.

## Static-site constraint

Code must remain compatible with static deployment unless a task explicitly states otherwise.

Do not introduce dependencies on:

- `+page.server.ts`
- `+layout.server.ts`
- SvelteKit server endpoints
- server hooks
- server-only environment variables
- server-side databases
- server-side sessions

Prefer browser APIs and client-side TypeScript.

## Dependency policy

Prefer:

1. Browser platform APIs.
2. Existing project dependencies.
3. Small, focused, well-maintained libraries.
4. New infrastructure only when justified by a concrete requirement.

Do not add large dependencies to solve trivial problems.

Before adding a dependency, verify that existing code or browser APIs cannot reasonably provide the functionality.

Do not introduce a database, authentication framework, AI SDK, state framework, or backend simply because it may be useful later.

Build for current requirements while keeping extension points clean.

## Code organization

Separate workflow/domain logic from Svelte UI code.

Avoid placing substantial parsing, validation, conversion, or export logic directly inside `.svelte` components.

Prefer:

```text
UI
 ↓
domain function
 ↓
typed data
 ↓
domain function
 ↓
output
```

For example:

```text
RubricEditor.svelte
        ↓
parseRubric()
        ↓
Rubric
        ↓
validateRubric()
        ↓
transformRubric()
        ↓
exportRubric()
```

Domain functions should be deterministic and independently testable whenever practical.

## TypeScript

Use TypeScript intentionally.

Prefer:

- Explicit domain types.
- Discriminated unions where useful.
- Narrow types over broad strings.
- `unknown` followed by validation for untrusted input.
- Pure functions for transformations.
- Clear return values for validation.

Avoid `any` unless there is a compelling reason.

Files supplied by users are untrusted input. Parse and validate them accordingly.

## Svelte

Keep Svelte components focused on presentation and interaction.

Prefer reusable components for recurring UI concepts such as:

- File selection/drop zones
- Tool headers
- Tool cards
- Validation results
- Warnings
- Data previews
- Download/export actions
- Help and privacy notices

Do not abstract a component merely because two pieces of markup currently look similar. Extract components when they represent a meaningful reusable interface concept.

## Styling

Use Tailwind CSS as the primary styling mechanism.

The project includes:

- Tailwind Forms
- Tailwind Typography

Use Forms for application controls where appropriate.

Use Typography's `prose` utilities for documentation, guides, instructions, release notes, and other long-form material.

Maintain a consistent visual language across tools.

Prefer accessibility and clarity over decorative complexity.

## File-processing tools

Many ADS Toolkit features will follow:

```text
local file
   ↓
parse
   ↓
validate
   ↓
typed internal representation
   ↓
edit/transform
   ↓
serialize
   ↓
browser download
```

Keep these stages separable.

Do not tightly couple parsing and exporting to the interface.

When handling formats such as CSV, JSON, XML, QTI, ZIP, or other structured files:

- Preserve data that the tool does not intentionally modify.
- Handle Unicode correctly.
- Account for quoting and escaping rules.
- Surface malformed input rather than silently guessing where that could corrupt output.
- Provide useful validation errors.
- Test meaningful edge cases.

## Validation philosophy

Prefer validation before transformation/export.

Validation messages should tell the user:

1. What is wrong.
2. Where it is wrong, when identifiable.
3. Whether it blocks export.
4. What the user can do about it.

Distinguish errors from warnings.

Do not silently "fix" ambiguous user data.

## Testing

The project includes:

- Vitest unit testing
- Vitest component testing
- Playwright end-to-end testing

### Unit tests

Prioritize unit tests for domain logic.

Important targets include:

- Parsers
- Validators
- Transformations
- Serializers/exporters
- Calculations
- Format conversions

Use fixtures for realistic file-processing cases.

Whenever fixing a parsing/transformation bug, add a regression test when practical.

### Component tests

Use component tests for important reusable interactions.

Do not reproduce every unit test through a component test.

### Playwright

Use Playwright for critical end-to-end workflows.

Prefer a small number of meaningful scenarios such as:

```text
load fixture
→ inspect parsed information
→ modify data
→ validate
→ export
```

Do not make the end-to-end suite unnecessarily large or brittle.

## Test data

Never commit real student, faculty, course, assessment, or institutional data as test fixtures.

Fixtures must be synthetic.

Synthetic data should be realistic enough to exercise the relevant file format and edge cases without imitating identifiable records.

## Tool UX

Each tool should answer four questions quickly:

1. What does this tool do?
2. What should I give it?
3. What will it do with my data?
4. What can I download or copy when finished?

Prefer focused workflow interfaces over dashboards.

A user should not need documentation to understand the basic happy path.

For complex tools, provide a concise guide.

## Documentation

Update relevant documentation when behavior changes.

Each substantial tool should document:

- Purpose
- Accepted inputs
- Output
- Important validation rules
- Privacy/data handling behavior
- Known limitations

Documentation should describe actual behavior rather than aspirational features.

## Accessibility

Accessibility is a functional requirement.

Use:

- Semantic HTML
- Associated form labels
- Appropriate headings
- Keyboard-accessible interactions
- Visible focus states
- Accessible names
- Sufficient contrast
- Useful error and status announcements

Do not rely on color alone to communicate status.

Prefer native browser behaviors over custom implementations when they provide better accessibility.

## Error handling

User-facing errors should be understandable to non-developers.

Prefer:

> This CSV is missing the `criterion_id` column required for rubric import.

over:

> TypeError: Cannot read properties of undefined

Technical details may be logged during development but should not replace actionable user messages.

Malformed user files should not crash the entire application.

## Security

Treat all uploaded/local files as untrusted.

Do not inject file contents as unsanitized HTML.

Never place secrets in client-side code.

Do not assume a static site can safely conceal:

- API keys
- access tokens
- passwords
- private endpoints
- organization secrets

Anything shipped in the static application must be considered visible to users.

## Performance

Most tools will process relatively small files, so clarity should generally take precedence over premature optimization.

However:

- Avoid unnecessary copies of large files.
- Avoid blocking the UI during expensive processing when practical.
- Consider Web Workers if a real workflow becomes computationally expensive.
- Release object URLs and other browser resources when they are no longer needed.

Optimize based on demonstrated need.

## Git and change discipline

Keep changes scoped to the requested feature.

Do not perform broad refactors unrelated to the task.

Do not replace working architecture merely to adopt a fashionable pattern.

Before considering work complete:

1. Run formatting.
2. Run linting/type checks available in the project.
3. Run relevant Vitest tests.
4. Run relevant Playwright tests for affected critical workflows.
5. Build the static application successfully.

Fix failures introduced by the change rather than disabling checks.

## Avoid speculative complexity

This repository is expected to grow, but future needs are not current requirements.

Do not preemptively add:

- Authentication
- Databases
- Server runtimes
- Cloud functions
- AI integrations
- Complex state management
- Microservices
- Collaboration infrastructure

Add architecture when a real tool requires it.

## AI-specific guidance

AI agents are development collaborators, not an architectural justification for adding AI to ADS Toolkit.

Prefer deterministic implementations when the workflow can be reliably expressed with code.

For example:

```text
Preferred:
CSV → parser → validator → transformation → CSV

Not preferred:
CSV → LLM → interpreted output → CSV
```

An AI-powered feature is appropriate only when the requirement actually benefits from probabilistic language or reasoning capabilities.

## Guiding principle

When choosing between two reasonable implementations, prefer the one that keeps ADS Toolkit:

- simpler,
- local-first,
- deterministic,
- testable,
- accessible,
- portable,
- transparent about data handling,
- and compatible with static deployment.

## GitHub Pages compatibility

ADS Toolkit is deployed as a GitHub Pages project site.

Production builds may be served from a repository base path
rather than the domain root.

- Do not assume the application is hosted at `/`.
- Use SvelteKit path utilities for internal application URLs.
- Avoid hard-coded root-relative asset and navigation paths.
- All new routes and assets must work when deployed beneath
  the GitHub Pages repository path.
- Changes must preserve static prerendering compatibility.
