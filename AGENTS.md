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
- Only create abstraction when generalizing over at least 3 examples.
- Make illegal state irrepresentable, but also don't fall into the trap of pureist (FP, OO, ...).

# Coding agent rules

If you're a coding agent, follow these rules strictly!

- Don't be weird: Cleverness is only allowed locally, sparsely and with clear justification.
- No fallback: You have the tendancy to add fallback/make things safe. Just let it fail and only add try/catch when absolutely needed.
- No nesting: Make serious efforts to make the logic streamlined. Only add nesting when absolutely necessary.
- Simple and dumb: Be as simple and dumb as possible. Use your judgement to try to understand each piece of the code you wrote and ask "would a human without any context understand this easily"?
- Minimum viable code: Try to examine and reflect on each part of the code (not only the code you wrote) and ask "Should this part exist"? If not, ruthless remove them.
- Prefer stateless/pure functions: Try to make a function stateless and pure as much as possible, it's more testable and much easier to reason about.
- Name things literally: Name functions/variables by what they do, not every condition/context.

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

    throw error
  }
}
```

Why?

- `args` is too general. It hides the fact that this object is really configuring document-opening behavior.
- `openDocument` sounds simple, but it secretly means read, parse, recover, and maybe create.
- The defaults are too strong. Missing files and invalid JSON become empty documents unless the caller opts out.
- The `catch` treats unrelated failures the same way: file missing, invalid JSON, permission errors, and disk errors.
- Returning an empty document is not error handling. It is inventing data.
- `createIfMissing` and `recoverInvalidJson` are API smells because they hide product decisions inside boolean options.

Good code:

```ts
async function readDocument(filePath: string): Promise<Document> {
  const text = await fs.readFile(filePath, "utf8")
  return JSON.parse(text) as Document
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
- `readDocument` does not catch errors it cannot honestly handle.
- Missing files, invalid JSON, and permission errors fail with their real cause.
- The API is smaller because callers choose the behavior directly instead of configuring a vague helper.
