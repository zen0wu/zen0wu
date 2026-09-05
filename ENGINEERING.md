# Engineering

## Code should read like a workflow

- Use concrete domain nouns and verbs. Avoid placeholder words such as `data`, `value`, `item`, `block`, or `state`, and process jargon such as `prepare`, `preflight`, or `handle`, unless they are genuinely precise.
- Declare a variable as close as possible to its first use.
- Keep control flow streamlined. Avoid nested conditionals and inner functions unless the structure genuinely requires them.
- Order a file as the main entry point or workflow, mid-layer business functions in call order, then leaf and shared utilities. Put only the types and constants needed to understand the workflow before it.
- Use prominent section headers for distinct workflows and short intent comments for major phases of a long function.
- Keep cleverness local and sparse. If it cannot be removed, explain it at the beginning of the relevant code block.
- Prefer stateless, pure functions when practical.

## Boundaries should own one concept

- A mid-layer function must own a recognizable domain step. If its purpose is not obvious from its name and signature, redesign the boundary before documenting it.
- Keep discovery and iteration, parsing, policy, validation, and mutation separate. Lower-level operations handle one explicit unit; orchestration owns scanning and repetition.
- Treat function-valued parameters, fields, and return values as a design smell. Use a callback only when varying behavior is itself the domain concept.
- Validate external and versioned data at the boundary, then convert it once into typed, immutable, canonical domain objects. Do not leak raw maps, version checks, or legacy-only fields into core validation or execution.
- Make illegal states unrepresentable without forcing the design into functional, object-oriented, or another form of purism.

## Abstractions must earn their place

- Create a reusable or parameterized abstraction only when it generalizes at least three real examples. A one-call-site function is justified only when it names a real domain phase or invariant and makes its caller easier to scan.
- Before finishing, inventory every new function, type, field, option, and layer. Remove or inline anything that does not own a clear domain concept or invariant.

## Fail explicitly

- Let failures surface. Add `try`/`catch` only when the code can handle a specific failure honestly.
- Throw specific error classes or typed error values. Never throw a generic `new Error`.

## Plan before mutation

For multi-target or destructive workflows, build the complete plan, validate the whole plan, then apply mutations. Execution should consume the plan without rediscovering or reinterpreting the source data.

## Architecture reasoning

- Define every system component, integration, event, or overloaded technical term before using it, then use one canonical name for it throughout the explanation.
- Name the exact component responsible for each behavior. Do not hide responsibility behind vague subjects such as "it", "the platform", or "the current path".
- Separate what the underlying product supports, what an integration supports, what its API or infrastructure adapter exposes, and what the current repository configuration enables.
- Qualify and classify limitations. Distinguish physically impossible, unsupported by the underlying product or API, unsupported by the current implementation or configuration, possible but unvalidated, and possible but rejected because of operational trade-offs.
- Resolve ambiguous terminology before reasoning. Analyze interpretations separately when they materially change the answer.
- Before making an impossibility claim, check the nearest alternative mechanism that relaxes one assumption and verify its capabilities using current code or primary documentation.
- Use a capability matrix when two or more independent dimensions affect the conclusion.

# Architecture case studies

## Layered integrations

Bad:

> The platform supports inline comments, but the provider does not, so the current path needs a separate webhook.

Why?

- "Platform", "provider", "current path", and "separate webhook" are not defined.
- The reader cannot tell whether "provider" means the external product's native integration, an API client, or an infrastructure-as-code adapter.
- The same component may be renamed later as a "service" or "built-in behavior", making the explanation internally inconsistent.
- The claim mixes product capability, configuration exposure, and the repository's current configuration.

Good:

> GitHub emits a `pull_request_review_comment` event for each new inline comment or reply. Buildkite's native GitHub integration can create builds from that event, but requires the comment to contain a configured command phrase and come from a trusted author. The Buildkite Terraform provider does not expose the setting that enables this trigger. Therefore, the current Terraform configuration cannot enable arbitrary inline-comment triggers.

Why?

- Each component has one stable name.
- Every capability and limitation is attributed to the component responsible for it.
- The explanation defines the event and distinguishes product behavior, Terraform exposure, and current configuration.
- The conclusion follows from the named constraints without introducing another vague mechanism.

## Infrastructure layers

Bad:

> Non-Auto EKS requires one node group per AZ for persistent EBS.

Why?

- "Non-Auto EKS" does not identify the node provisioner.
- "Node group" conflates EKS Managed Node Groups with Karpenter NodePools.
- It presents a limitation of multi-AZ Auto Scaling Groups as a limitation of standard EKS.
- It calls something impossible without naming the constraints under which it is impossible.

Good:

> With an EKS Managed Node Group backed by a multi-AZ Auto Scaling Group, replacement placement is not reliably driven by a pending Pod's EBS AZ constraint. Standard EKS with upstream Karpenter can instead use one multi-AZ NodePool: Karpenter reads the existing PersistentVolume's zone affinity and provisions the replacement node in that AZ. EKS Auto Mode can do the same while operating the node provisioning and EBS integration for us.

Why?

- It names the responsible mechanisms and resources.
- It scopes the limitation to the configuration where it applies.
- It distinguishes platform capability from the current implementation.
- It identifies the nearest valid alternative.

# Code case studies

1. Boolean logics

Bad code:

```ts
function setGoMemoryForBoxyTypecheck() {}
  if (process.env.BOXY_NAME !== undefined && process.env.GOMEMLIMIT === undefined) {
    process.env.GOMEMLIMIT = '20GiB'
  }
}
```

Why?

- process.env.X and Y are not the intention. They're the condition. Readers without context won't understand what they do.
- Concatenating these two conditions are confusing because they don't speak the same matter.
- Mutation of process.env is BAD. The intention of this code is to pass the the memory limit to spawned process, then we should pass this when spawning.

Good code:

```ts
function getGoMemory() {}
  if (process.env.BOXY_NAME !== undefined) {
    return process.env.GOMEMLIMIT ?? '20GiB'
  }
  return undefined
}
```

Why:
- The name of the function is a bit more generalized. We should think of this as "it's a function to configure memory limit", not "a function to set memory only for boxy when running typecheck".
- This is a pure function. Does not mutate state (getter, not setter).
- The condition nesting are well justified. The first level says "We want to adjust memory for boxy" and the second level (even tho written as trinary) says "We don't want to overwrite existing values".

2. Data shape

Bad code:

```ts
type Document = {
  title: string
  isDraft: boolean
  publishedAt?: Date
  archivedAt?: Date
}

function getDocumentAction(document: Document): string {
  if (document.archivedAt !== undefined) {
    return "Restore"
  }

  if (document.isDraft) {
    return "Publish"
  }

  if (document.publishedAt !== undefined) {
    return "Unpublish"
  }

  return "Publish"
}
```

Why?

- The type allows confusing states: draft and published, archived and draft, not draft but also not published.
- The function is guessing the product state from scattered fields.
- The final `return "Publish"` is a fallback for a state that should not exist.
- The return type says nothing about which actions are actually allowed.

Good code:

```ts
type Document =
  | { type: "draft"; title: string }
  | { type: "published"; title: string; publishedAt: Date }
  | { type: "archived"; title: string; archivedAt: Date }

type DocumentAction = "publish" | "unpublish" | "restore"

function getDocumentAction(document: Document): DocumentAction {
  if (document.type === "archived") {
    return "restore"
  }

  if (document.type === "published") {
    return "unpublish"
  }

  return "publish"
}
```

Why?

- The input type names the real product states directly.
- The output type names the real actions directly.
- Each state owns only the fields that make sense for that state.
- The function does not need a fallback for impossible data.
- The code is simpler because both sides of the function use honest data shapes.

3. Abstraction timing

Bad code:

```ts
type SaveModelOptions<T, Model> = {
  table: string
  getId: (input: T) => string
  getData: (input: T) => object
  afterSave: (model: Model) => Promise<void>
}

function createSaveModel<T, Model>(options: SaveModelOptions<T, Model>) {
  return async function saveModel(input: T) {
    const id = options.getId(input)
    const data = options.getData(input)

    const model = await db[options.table].upsert({
      where: { id },
      update: data,
      create: { id, ...data },
    })

    await options.afterSave(model)

    await sendSlackMessage({
      channel: "#engineering-feed",
      text: `Saved ${options.table} ${id}`,
    })

    return model
  }
}

const saveUser = createSaveModel<UserInput, User>({
  table: "user",
  getId: input => input.id,
  getData: input => ({ name: input.name, email: input.email }),
  afterSave: user => emailQueue.enqueue("verify_email", { userId: user.id }),
})

const saveProject = createSaveModel<ProjectInput, Project>({
  table: "project",
  getId: input => input.id,
  getData: input => ({ name: input.name }),
  afterSave: project => permissions.rebuildProject(project.id),
})
```

Why?

- An abstraction is not free. `createSaveModel` creates a second-order concept every reader has to learn.
- Here, the abstraction exists because two functions would have looked similar, not because the reader benefits from learning a new concept.
- Passing `getId`, `getData`, and `afterSave` is a smell here. The helper depends on callbacks instead of plain data.
- `afterSave` is an escape hatch for arbitrary domain behavior.
- Posting to Slack is surprising from a generic save helper. Side effects in an abstraction should fit the abstraction's name and scope.

Good code:

```ts
async function saveUser(input: UserInput) {
  const user = await db.user.upsert({
    where: { id: input.id },
    update: {
      name: input.name,
      email: input.email,
    },
    create: {
      id: input.id,
      name: input.name,
      email: input.email,
    },
  })

  await emailQueue.enqueue("verify_email", { userId: user.id })

  return user
}

async function saveProject(input: ProjectInput) {
  const project = await db.project.upsert({
    where: { id: input.id },
    update: {
      name: input.name,
    },
    create: {
      id: input.id,
      name: input.name,
    },
  })

  await permissions.rebuildProject(project.id)

  return project
}
```

Why?

- The code is boring, but each function says exactly what it does.
- The side effects are concrete and domain-specific.
- There is no generic callback-shaped escape hatch.
- The duplication is small and still easy to read.
- There are only two examples, so the shared shape has not earned a name yet.

4. API design, defaults, and error handling

Bad code:

```ts
type OpenDocumentArgs = {
  filePath: string
  createIfMissing?: boolean
  recoverInvalidJson?: boolean
}

async function openDocument(args: OpenDocumentArgs): Promise<Document> {
  const createIfMissing = args.createIfMissing ?? true
  const recoverInvalidJson = args.recoverInvalidJson ?? true

  try {
    const text = await fs.readFile(args.filePath, "utf8")
    return JSON.parse(text) as Document
  } catch (error) {
    if (createIfMissing || recoverInvalidJson) {
      return {
        title: "Untitled",
        blocks: [],
      }
    }

    throw new Error(`Failed to open document: ${args.filePath}`)
  }
}
```

Why?

- `args` is too general. It hides the fact that this object is really configuring document-opening behavior.
- `openDocument` sounds simple, but it secretly means read, parse, recover, and maybe create.
- The defaults are too strong. Missing files and invalid JSON become empty documents unless the caller opts out.
- The `catch` treats unrelated failures the same way: file missing, invalid JSON, permission errors, and disk errors.
- `new Error` is too generic. Callers can only inspect a string instead of handling a typed failure.
- Returning an empty document is not error handling. It is inventing data.
- `createIfMissing` and `recoverInvalidJson` are API smells because they hide product decisions inside boolean options.

Good code:

```ts
class InvalidDocumentJsonError extends Error {
  readonly filePath: string
  readonly sourceError: unknown

  constructor(filePath: string, sourceError: unknown) {
    super(`Invalid document JSON: ${filePath}`)
    this.name = "InvalidDocumentJsonError"
    this.filePath = filePath
    this.sourceError = sourceError
  }
}

async function readDocument(filePath: string): Promise<Document> {
  const text = await fs.readFile(filePath, "utf8")
  return parseDocument(filePath, text)
}

function parseDocument(filePath: string, text: string): Document {
  // JSON.parse throws SyntaxError, which is a parser detail.
  // Convert it once at the document boundary so callers see a typed document error.
  try {
    return JSON.parse(text) as Document
  } catch (sourceError) {
    throw new InvalidDocumentJsonError(filePath, sourceError)
  }
}

async function createDocument(filePath: string): Promise<Document> {
  const document = createEmptyDocument()

  await fs.writeFile(filePath, JSON.stringify(document))

  return document
}

function createEmptyDocument(): Document {
  return {
    title: "Untitled",
    blocks: [],
  }
}
```

Why?

- Reading and creating are separate operations.
- The empty document default exists only in the creation path.
- Invalid JSON is converted into a typed document error at the parser boundary.
- `readDocument` still does not catch errors it cannot honestly handle.
- Missing files and permission errors fail with their real cause.
- The API is smaller because callers choose the behavior directly instead of configuring a vague helper.
