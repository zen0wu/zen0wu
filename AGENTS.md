# About me

My name is Zeno.
I like programming languages, various languages constructs and concepts.
I like exploring and learning new things.
I like deep thinking and convert those thinking into simple artifacts.

# My coding principles

- Abstraction balances: generalize real patterns, not one-off code.
- Simplicity wins: optimize for fewer and less interwined concepts, not less effort.
- Defaults matter: default to common cases, add options only when needed.

Things I like: 
- Simple and minimalist things.
- Clean code. 
- Think about various trade-offs deeply.
- Choose the simplest approach that works.

Things I don't like:
- Complex things.
- Over-engineered code/architecture.
- Unintuitive concepts.
- Tricks when designing high level architecture.
- Nesting: Nested if-else, inner functions, etc.

# My coding styles

- Use literal and intuitive variable names.
- Variable declarations should be as close as possible to their (first) usage.
- Whenever there's a cleverness in the code block, explain it at the beginning of that block.
- Each function should look like a few code blocks concatenated and the logic between them should be streamlined.
- Only create a reusable or parameterized abstraction when it generalizes at least 3 real examples. A one-call-site function is justified only when it names a real domain phase or invariant and makes its caller easier to scan.
- Make illegal state irrepresentable, but also don't fall into the trap of pureist (FP, OO, ...).

# Architecture reasoning

- Name the exact layer: Use the exact product, controller, and resource names. Do not collapse related mechanisms with `/` or use them as synonyms when their behavior differs.
- Qualify feasibility claims: Never say something is "impossible", "required", "the only way", or "cannot work" unless the claim names the active constraints, the component responsible for the behavior, and the invariant that prevents it.
- Classify limitations: Clearly distinguish physically impossible, unsupported by the underlying platform or API, unsupported by the current implementation or configuration, possible but not yet validated, and possible but rejected because of operational trade-offs.
- Resolve ambiguous terminology: Restate the concrete interpretation before reasoning. If different interpretations materially change the answer, analyze them separately.
- Check adjacent mechanisms: Before making an architectural impossibility claim, identify the nearest alternative mechanisms that relax one assumption and verify their capabilities using current code or primary documentation.
- Use a capability matrix when two or more independent dimensions affect the conclusion. Do not compress a multidimensional comparison into a single recommendation.

## Terminology discipline

- Define before use: Before using a system component, integration, event, or overloaded technical term that the user did not introduce, define it in plain language.
- One concept, one name: Choose one canonical term for each distinct concept and reuse that exact term throughout the response. Do not alternate between synonyms such as "platform", "service", "system", "built-in behavior", or "path".
- Ban unnamed mechanisms: Do not use vague subjects such as "it", "this", "the platform", "the current path", or "the built-in behavior" when more than one component is in scope. Name the responsible component.
- Separate capability layers: Distinguish what the underlying product supports, what a specific integration supports, what its API or infrastructure adapter exposes, and what the current repository configuration enables.
- Attribute every limitation: Write capability claims as "<exact component> supports/does not support <behavior>". Never write an unqualified "supported", "unsupported", "cannot", or "does not work".
- Explain qualifiers immediately: Define terms such as "trusted author", "command phrase", "native trigger", or "separate webhook" at first use, before reasoning about their consequences.
- Answer first: Start with the direct answer in the user's language. Introduce architecture terminology only when it is necessary to explain that answer.
- Check terminology before responding: Identify every named component in the draft, replace alternate names with its canonical term, and confirm every capability or limitation is attributed to the correct component.

## Architecture case study: layered integrations

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

## Architecture case study: infrastructure layers

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

# Coding agent rules

If you're a coding agent, follow these rules strictly!

- Don't be weird: Cleverness is only allowed locally, sparsely and with clear justification.
- No fallback: You have the tendancy to add fallback/make things safe. Just let it fail and only add try/catch when absolutely needed.
- Typed errors: Throw specific error classes or typed error values. Never throw generic `new Error`.
- No nesting: Make serious efforts to make the logic streamlined. Only add nesting when absolutely necessary.
- Simple and dumb: Write for a human with no task context. They should understand the overall workflow by scanning the entry point and understand why each function exists from its name and signature without reading its body.
- Reader-first structure: Order a file as the main entry point or workflow, mid-layer business functions in call order, then leaf and shared utilities. Put only the types and constants needed to understand the workflow before it. Use prominent section headers for distinct workflows and short intent comments for major phases of a long function.
- Obvious function boundaries: A mid-layer function must own a recognizable domain step. If its purpose is not obvious from its name and signature, redesign the boundary before adding documentation. If the domain concept itself is non-obvious, add a short docstring explaining what it means, why it exists, and one concrete example.
- One concept per boundary: Keep discovery and iteration, parsing, policy, validation, and mutation separate. Lower-level operations should handle one explicit unit; orchestration owns scanning and repetition. Do not combine independent requirements merely because they are part of the same feature.
- Prefer data over callbacks: Treat function-valued parameters, fields, and return values as a design smell. A callback is justified only when varying behavior is itself the domain concept. Otherwise, pass concrete data or keep the behavior in the layer that owns it.
- Minimum viable code: Before finishing, inventory every new function, type, field, option, and layer. State which domain concept or invariant it owns. If there is no clear answer, remove or inline it.
- Prefer stateless/pure functions: Try to make a function stateless and pure as much as possible, it's more testable and much easier to reason about.
- Name things literally: Use concrete domain nouns and verbs. Avoid placeholder words such as `data`, `value`, `item`, `block`, or `state`, and process jargon such as `prepare`, `preflight`, or `handle`, unless they are genuinely the most precise domain terms.
- Normalize at boundaries: Validate external and versioned data at the boundary, then convert it once into typed, immutable, canonical domain objects. Raw maps, version checks, and legacy-only fields must not leak into core validation or execution.
- Plan before mutation: For multi-target or destructive workflows, first build the complete plan, then validate the whole plan, then apply mutations. Execution should consume the plan without rediscovering or reinterpreting the source data.

## Case studies

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
