---
name: code-comment-style
description: Apply this project's JSDoc, code classification, and JSX classification conventions whenever writing or modifying code.
---

# Code Comment Style

Use comments to make responsibility, validation boundaries, and non-obvious
reasoning easy to scan. Comments must explain why a rule exists or how an
unusual mechanism works; do not restate what the next line already says.

## Comment types

| Type | Purpose | Format |
|---|---|---|
| JSDoc | Explain a declaration's responsibility and contract | `/** ... */` |
| Code classification | Divide declarations, branches, and workflow stages | `// Simple title` |
| JSX classification | Divide independent UI sections | `{/* Simple title */}` |
| File section | Divide substantial module-level groups | `/*==== Section ====*/` |
| Block explanation | Show a shape, rule set, diagram, or worked example | `/* ... */` |

## Core rules

- **Classify first:** split dense code into meaningful groups or workflow stages.
- **Explain purpose and mechanism:** describe the contract, reason, invariant, or
  non-obvious behavior instead of narrating syntax.
- **Use short English titles:** prefer `// Teacher authorization` or
  `// Attempt selection state` over sentence-length headings.
- **Keep related code together:** a title applies until the next classification title
  or the end of the surrounding block.
- **Match repository formatting:** use four-space indentation, double quotes, semicolons,
  and existing TypeScript/JSX formatting in examples.
- **Preserve project terms:** use established domain and translation-key terminology.
- **Preserve recognized directives:** do not rewrite comments such as
  `// @vitest-environment node` or an existing `// todo:` marker.
- **Keep documentation-only changes behavior-free:** do not change runtime logic, APIs,
  declaration order, or unrelated formatting while adding comments.
- **Keep comments current:** update or remove a title and its explanation when the code's
  responsibility changes.

## 1. JSDoc

### What it does

- Defines the responsibility of a function, component, hook, type, or constant.
- Documents parameters, return values, errors, side effects, and invariants when needed.
- Explains why a declaration exists instead of narrating its implementation line by line.

### Simple declaration

```ts
/** Matches the complete timezone-bearing timestamp contract shared by student data mappers. */
const ISO_TIMESTAMP_PATTERN = /.../;
```

### Function contract

```ts
/**
 * Reads a non-empty identity string without normalizing the accepted value.
 *
 * The caller supplies the error boundary so this helper does not replace the
 * caller's exception type or message adapter.
 *
 * @param value - The value to validate.
 * @param options - The caller-owned validation and error contract.
 * @returns The original validated string.
 */
export function readIdentity(
    value: unknown,
    options: PaperDataValidationOptions,
): string { ... }
```

### Props and type fields

Document the type and each field when ownership or behavior matters:

```ts
/** Configures the controlled Markdown editor. */
export type EditorProps = {
    /** Canonical Markdown value owned by the parent. */
    value: string;
    /** Receives Markdown after the editor serializes the document. */
    onChange: (markdown: string) => void;
    /** Optional image pipeline; uploads are rejected when it is absent. */
    onImageUpload?: (file: File) => Promise<string>;
};
```

### Use JSDoc for

- Exported functions, components, hooks, types, and public props when their contract
  is not obvious from the name and type.
- Non-trivial internal helpers.
- Styled components that own a layout or interaction invariant.
- Persistence, API, storage, and restoration boundaries.
- Immutable snapshots and resumable workflows where ordering or idempotency matters.

Do not add JSDoc to every trivial local constant or obvious JSX element. A concise
classification comment is sufficient when the surrounding group already explains
the responsibility.

Use the repository's `@param name - description` and `@returns` style. Keep the
summary sentence on the opening line when it remains readable; use a separate
paragraph for the reason or mechanism.

## 2. Code classification

### What it does

- Gives each declaration group or workflow stage a simple, scannable title.
- Shows where one responsibility ends and the next begins.
- Makes long functions and hooks readable without turning every line into prose.

### File and declaration groups

For module-level sections, the repository uses a separator title:

```ts
/*==== Helper Functions ====*/
```

Use this only for a real file-level group. For local groups and component logic,
prefer `//` titles.

```ts
// Selection state
const [selectedIds, setSelectedIds] = useState<ReadonlySet<string>>(() => new Set());
const [expandedStudentIds, setExpandedStudentIds] = useState<ReadonlySet<string>>(
    () => new Set(),
);

// Grading summary
const hasManualTeacherComment = gradingRecords.some((item) => item.hasManualTeacherComment);
const hasTeacherProgress = gradingRecords.some((item) => item.status === "inProgress");

// Preview data
const preview = useMemo(
    () => createPreviewData(attempts, grading),
    [attempts, grading],
);
```

The title names what the group owns. Add a focused comment only when a value has a
non-obvious invariant:

```ts
// Persistence queue
const saveRevisionRef = useRef(0);
// Serializes writes so an older operation cannot overwrite a newer snapshot.
const saveQueueRef = useRef<Promise<void>>(Promise.resolve());
```

### Workflow stages

Short action or outcome titles are valid classifications when they accurately divide
the workflow:

```ts
// Validate
const validation = validatePaperData(value, options);

// Invalid
if (!validation.valid) {
    return fail(validation.message);
}

// Send to server
await saveDraft(draft);

// Clear local draft
await clearDraft();

// No error
navigate(nextPath);
```

Prefer a more specific title when it stays simple:

| Generic title | Clearer title |
|---|---|
| `// Clear` | `// Clear local draft` |
| `// Error` | `// Publication error` |
| `// Data` | `// Preview data` |
| `// Send` | `// Send to server` |

### Algorithm stages

```ts
// Initial scan state
let cursor = 0;
let fence: string | null = null;

// Find image label
const labelStart = markdown.indexOf("![", cursor);

// Parse image destination
const destination = parseImageDestination(markdown, labelStart);

// Replace from back to front
for (const image of images.toReversed()) {
    markdown = replaceImageDestination(markdown, image);
}

// Final integrity check
assertNoLocalImageUrls(markdown);
```

### Local explanations

Inside a category, add `//` explanations for:

- Race prevention and asynchronous ordering.
- Security or trust boundaries.
- Coordinate-system rules.
- Cleanup requirements.
- Browser or framework behavior that is not obvious from the code.
- A deliberately unusual implementation choice.

```ts
// Image dimensions
// A 90-degree turn exchanges the visible width and height.
const width = swapsDimensions ? image.naturalHeight : image.naturalWidth;
```

Prefer a trailing comment only when it explains a non-obvious rule and remains
short enough to read with the statement. Move a longer explanation to the line
above or into JSDoc.

Do not comment self-evident assignments individually:

```ts
// Bad: repeats the syntax without adding a category or useful explanation.
busyRef.current = true; // Set busy to true.
```

## 3. JSX classification

### What it does

- Divides a long render tree into independent UI sections.
- Identifies what each section renders or controls.
- Lets readers find dialogs, actions, feedback, and editor areas quickly.

### Example

```tsx
{/* Validation feedback */}
<PublicationValidationAlert issues={issues}/>

{/* Question editor */}
<PaperQuestionCard questionId={questionId}/>

{/* Retry action */}
<Button onClick={retryPublication}>Retry</Button>

{/* Progress dialog */}
<PublicationProgressDialog step={publicationStep}/>
```

### Title style

- Use a short noun or action phrase.
- Use normal capitalization and spacing.
- Keep existing domain terms.
- A title may be simple when its enclosed JSX clearly explains the details.

```tsx
{/* Good */}
{/* Student preview */}
{/* Draft action */}
{/* Publish action */}
{/* Internal error */}

{/* Avoid inconsistent spacing or unclear abbreviations */}
{/*student Preview*/}
{/*btn*/}
```

Do not classify every isolated element. Add a title when the section contains related
controls, a distinct branch, or an important dialog or feedback surface.

## 4. Block explanations

### What it does

- Presents information that is easier to understand as a list, shape, diagram, or table.
- Keeps a complex validation contract next to the code that enforces it.

### Shape example

```ts
/*
Accepted persisted record:
- Non-null object
- Matching schema version
- Valid editor state
- One binary record per referenced asset ID
*/
assertPaperEditorRecord(value);
```

### Diagram example

```ts
/*
Original image:     1200 x 800
Rotation 0 / 180:  1200 x 800
Rotation 90 / 270:  800 x 1200
*/
```

Use JSDoc instead when documenting a declaration's public contract.

## Classification checklist

Before finishing:

- [ ] Dense declarations are divided by responsibility.
- [ ] Long workflows are divided into clear stages and outcomes.
- [ ] Long JSX is divided into meaningful UI sections.
- [ ] Every title is short and easy to understand.
- [ ] Each category's code or explanation makes its responsibility and non-obvious
      mechanism clear.
- [ ] Complex contracts use JSDoc, lists, shapes, or examples instead of one long paragraph.
- [ ] JSDoc uses the repository's `@param name - description` and `@returns` style.
- [ ] File-level groups use `/*==== Section ====*/` only when a real module section exists.
- [ ] Comments do not repeat obvious assignments or every individual JSX element.
- [ ] Comments do not change or misrepresent runtime behavior.
- [ ] Comments use current project terminology and consistent formatting.
