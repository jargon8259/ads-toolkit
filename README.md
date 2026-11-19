# ADS Toolkit

ADS Toolkit is a collection of small, privacy-focused web tools designed to support Assessment Development workflows.

The toolkit is built with SvelteKit and TypeScript and is intended to remain lightweight, portable, and local-first. Most tools should work entirely within the user's browser: users provide local files or other input, ADS Toolkit processes them locally, and the resulting files or information are returned without sending source data to a server.

The project is intended for a small team, but tools should generally be institution-agnostic and suitable for public hosting when their functionality permits it.

## Goals

ADS Toolkit should:

- Reduce repetitive Assessment Development workflow tasks.
- Provide focused tools rather than one large monolithic application.
- Process sensitive or work-related files locally whenever possible.
- Require no application-level user accounts or authentication.
- Avoid storing uploaded or generated content.
- Be deployable as a static website.
- Remain usable outside any particular institution when a workflow is generic.
- Provide clear instructions and validation rather than silently modifying data.
- Grow incrementally as new workflow needs emerge.

## Example tools

Planned tools may include utilities for:

- Reading and editing Canvas rubric CSV files.
- Preparing updated rubrics for import into Canvas.
- Validating Canvas-compatible files before import.
- Inspecting or transforming CSV, JSON, XML, QTI, ZIP, and similar files.
- Comparing assessment-related files.
- Reformatting structured assessment content.
- Preparing files for import into other systems.
- Performing assessment-development calculations or validations.

Not every tool needs to relate to Canvas. The toolkit is a home for small utilities that make Assessment Development workflows safer, faster, or less repetitive.

## Privacy model

ADS Toolkit follows a **local-first** model.

Unless a tool clearly states otherwise:

1. Files are read using browser APIs.
2. File contents remain on the user's device.
3. Processing occurs in the browser.
4. Input files are not uploaded to a server.
5. Processed content is not persistently stored.
6. Generated files are downloaded directly by the user.
7. The application does not require an account.
8. Core tools should not depend on external APIs.

Avoid storing user content in `localStorage`, IndexedDB, cookies, analytics systems, or other persistent storage.

Browser storage may be used for harmless application preferences, such as theme or UI settings, when appropriate.

## Architecture

ADS Toolkit is a static SvelteKit application.

The basic workflow should generally be:

```text
Local file or user input
          ↓
      Browser API
          ↓
  Parse / validate
          ↓
   Transform / edit
          ↓
       Preview
          ↓
 Generate output file
          ↓
       Download
```

Core processing logic should be separated from the user interface.

For example:

```text
RubricEditor.svelte
        ↓
rubric-parser.ts
rubric-validator.ts
rubric-transformer.ts
rubric-exporter.ts
```

This makes processing logic easier to test, reuse, and maintain.

## Project structure

As the toolkit grows, prefer a structure similar to:

```text
src/
├── lib/
│   ├── components/
│   │   ├── FileDropzone.svelte
│   │   ├── ToolCard.svelte
│   │   ├── ValidationResult.svelte
│   │   └── DownloadButton.svelte
│   │
│   ├── tools/
│   │   └── ...
│   │
│   ├── files/
│   │   └── ...
│   │
│   └── utils/
│       └── ...
│
├── routes/
│   ├── +page.svelte
│   └── tools/
│       └── ...
│
└── ...

tests/
├── fixtures/
└── ...
```

Individual tools may have their own domain modules under `src/lib/tools/`.

## Technology

The project currently uses:

- SvelteKit
- Svelte
- TypeScript
- Bun
- Tailwind CSS
- Tailwind Forms
- Tailwind Typography
- Prettier
- ESLint
- Vitest
  - Unit testing
  - Component testing
- Playwright
- SvelteKit static adapter
- Svelte enhanced images

The project intentionally does **not** currently require:

- A database
- Server-side application logic
- Application authentication
- An AI/LLM runtime
- Persistent user storage

These should only be introduced when a concrete tool requirement justifies the additional architecture.

## Development

Install dependencies:

```sh
bun install
```

Start the development server:

```sh
bun run dev
```

Build the static application:

```sh
bun run build
```

Run available tests and checks using the scripts defined in `package.json`.

## Recreating the initial project

The original project was scaffolded with:

```sh
bun x sv@1.6.0 create --template demo --types ts --add prettier eslint vitest="usages:unit,component" playwright="demo:yes" tailwindcss="plugins:typography,forms" sveltekit-adapter="adapter:static" enhanced-img --install bun ads-toolkit
```

This command documents the initial scaffold. The repository itself is the source of truth after creation.

## Testing philosophy

Tests are especially important because many ADS Toolkit tools transform structured files.

### Unit tests

Use Vitest for deterministic processing logic such as:

```text
input
  ↓
parser
  ↓
validation
  ↓
transformation
  ↓
expected output
```

File parsers, validators, calculations, serializers, and exporters should generally be testable independently from Svelte components.

### Component tests

Use Vitest component testing for reusable Svelte interfaces and interactions.

Examples include:

- File selection and drop zones
- Validation messages
- Editable data controls
- Tool configuration forms
- Preview components

### End-to-end tests

Use Playwright for a small number of important complete workflows.

For example:

```text
Open Canvas Rubric Editor
        ↓
Load test fixture
        ↓
Edit criterion
        ↓
Validate rubric
        ↓
Export CSV
        ↓
Verify expected result
```

Prefer meaningful tests of core workflows over large numbers of superficial UI tests.

## Tool design

Each tool should make its purpose obvious.

Where practical, a tool should provide:

- A short description.
- Supported input formats.
- A clear input or file-selection area.
- Validation before transformation/export.
- Useful error messages.
- A preview when appropriate.
- An explicit export/download action.
- A concise explanation of how data is handled.
- A short usage guide.

Destructive or ambiguous transformations should not occur silently.

## Accessibility

Tools should be usable with keyboard navigation and assistive technology.

Prefer:

- Semantic HTML
- Explicit labels
- Native controls when practical
- Visible focus states
- Sufficient color contrast
- Error messages that do not rely solely on color
- Responsive layouts

Accessibility should be considered part of the tool's functionality rather than a later visual enhancement.

## Deployment

The project uses SvelteKit's static adapter.

The intended deployment artifact is therefore static HTML, CSS, JavaScript, and related assets.

This supports deployment models such as:

- GitHub Pages
- Other static web hosts
- Compatible internal static hosting

A deployment may be publicly accessible or placed behind host-level access controls without requiring ADS Toolkit itself to implement authentication.

## Guiding principle

> Build small, deterministic, privacy-focused workflow tools that do one job well.

AI can help develop ADS Toolkit, but the application itself should not require AI when deterministic browser-side processing can solve the problem.