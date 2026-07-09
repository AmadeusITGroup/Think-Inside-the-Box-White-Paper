# Think Inside the Box for Software Libraries

### A White Paper on Co-Packaging Code and AI-Optimized Documentation as an Industry Standard

**Authors:** AI Documentation Working Group - Andres IGEA OVIEDO (andres.igeaoviedo@amadeus.com), Andrea SCORTI (andrea.scorti@amadeus.com)  
**Date:** 7th of July 2026  
**Version:** 1.0

---

## Abstract

The rapid adoption of AI coding assistants — GitHub Copilot, Claude, Codex, and others — has fundamentally changed how developers consume software libraries. Yet the ecosystem's documentation infrastructure remains anchored in a pre-AI paradigm: code ships in a package, documentation lives elsewhere. This disconnect forces AI agents to operate without the contextual knowledge they need, producing incorrect API usage, outdated patterns, and hallucinated interfaces.

This paper argues for a new industry standard: **co-packaged documentation** — the practice of shipping structured, AI-optimized documentation alongside library code within the same distributable artifact. We call this the _Think Inside the Box Principle_: everything the consumer needs arrives in one box.

We present a reference implementation from the Design Factory design system, demonstrate that the approach works across package ecosystems (npm, pip, Maven), and propose a minimal specification that library authors can adopt today.

We also address critical concerns around security (prompt injection via co-packaged content), forward-compatibility with evolving agent architectures, scalability across deep dependency graphs, and the practical challenges of documentation generation pipelines.

---

## 1. Introduction: The AI-Driven Development Shift

Software development is undergoing its most significant workflow transformation since the introduction of the integrated development environment. AI coding assistants are no longer novelties — they are daily tools for millions of developers. GitHub reports that Copilot generates a significant share of new code in enabled repositories. Anthropic's Claude, OpenAI's Codex, and a growing ecosystem of AI agents are being embedded into every stage of the development lifecycle.

These agents share a common trait: **they are only as effective as the context they can access.** An AI assistant writing React code with full access to the React documentation produces dramatically better output than one relying solely on its training data. Training data goes stale; documentation is versioned and current.

Yet today, when a developer installs a library — whether via `npm install`, `pip install`, or a Maven dependency — the AI assistant working in that project receives the library's _code_ but almost never its _documentation_. The documentation exists, but it lives on a website, behind a search engine, in a format optimized for human browsing rather than machine consumption.

This is the gap we propose to close.

---

## 2. The Current State: A Separated World

### 2.1 How Libraries Ship Today

Across every major package ecosystem, the convention is the same: **the package contains code; the documentation is hosted elsewhere.**

| Ecosystem     | Package Contents                                     | Documentation Location                                 |
| ------------- | ---------------------------------------------------- | ------------------------------------------------------ |
| **npm**       | JavaScript/TypeScript source, `package.json`, README | Dedicated docs site, GitHub wiki, or MDN               |
| **PyPI**      | Python source, `setup.py`/`pyproject.toml`, README   | Read the Docs, Sphinx-hosted site, or GitHub pages     |
| **Maven**     | Compiled JARs, POM metadata                          | Javadoc on Maven Central, project wiki, or vendor site |
| **NuGet**     | .NET assemblies, XML doc comments                    | Microsoft Learn, GitHub, or custom docs portals        |
| **Crates.io** | Rust source, `Cargo.toml`                            | docs.rs (auto-generated)                               |

The README file — the one piece of documentation that does travel with the package — is typically a brief overview: a logo, a one-liner description, installation instructions, and a link to "full documentation" hosted online.

This separation made sense in a human-centric world. Developers browse documentation in a web browser with search, navigation, and rich formatting. Shipping a full documentation site inside every `node_modules` dependency would have been wasteful.

**But the consumer has changed.** The primary documentation consumer is increasingly an AI agent running locally in the developer's IDE, and the agent’s capabilities in browsing websites is not as robust as the capabilities it has for searching, navigating and reading files in a local workspace.

### 2.2 What AI Agents Can and Cannot Do

Modern AI coding assistants operate within a **file-based context window.** They can:

- Read files from the local filesystem (source code, configuration, markdown)
- Navigate directory structures
- Follow references between files
- Process structured and unstructured text

They typically have difficulty with, cannot or should not need to:

- Browse the internet during code generation
- Authenticate with documentation portals
- Parse JavaScript-rendered single-page application doc sites
- Maintain persistent knowledge across sessions about external APIs

When an AI agent encounters an unfamiliar library in a project's dependencies, it has three options:

1. **Rely on training data** — which may be months or years out of date, missing recent API changes, deprecations, or new features.
2. **Search the web** — which adds latency, requires network access (often unavailable in corporate environments), and returns HTML pages optimized for human reading.
3. **Read local files** — fast, always available, version-matched to the installed library, and optimized for machine consumption.

Option 3 is clearly superior. **But today, there is almost nothing to read.**

### 2.3 The Consequences of the Gap

The documentation gap produces observable failures in AI-assisted development:

- **Hallucinated APIs:** The agent invents function signatures, props, or class methods that don't exist in the installed version, because it is interpolating from stale training data.
- **Deprecated patterns:** The agent uses patterns from an older version of the library because its training data predates the current release.
- **Missing context:** The agent cannot recommend the idiomatic way to use a component because it has no access to usage guidelines, do/don't rules, or design system constraints.
- **Repeated correction cycles:** The developer must manually correct the agent's output, defeating the productivity gain that AI assistance promises.

These are not edge cases. They are the daily experience of developers using AI tools with any library that has evolved since the agent's training cutoff. Preliminary observations from Design Factory's internal adoption suggest that co-packaged documentation significantly reduces these failures, though rigorous benchmarking across diverse libraries remains an open research direction.

---

## 3. The Think Inside the Box Principle

Some physical goods companies' success is built on a deceptively simple idea: **everything you need comes in one box.**
For instance, when you purchase a bookcase, the box contains the panels, the shelves, the screws, the dowels, the cam locks, and — _critically_ — the assembly instructions. You do not need to visit a website to look up how to assemble it. You do not need to search YouTube for a tutorial. The instructions are _right there_, designed to be consumed alongside the product, version-matched and complete.

This principle has a direct analog in software:

> **Think Inside the Box for Software Libraries:** A package should contain everything an agent needs to correctly use the library — the code _and_ its documentation — in the same distributable artifact.

The analogy is precise:

| Physical goods company          | Software Library                                |
| ------------------------------- | ----------------------------------------------- |
| Product (panels, shelves)       | Source code, compiled artifacts                 |
| Assembly and usage instructions | API docs, usage guides, examples                |
| The box                         | The package (npm tarball, wheel, JAR)           |
| The customer                    | The AI coding agent                             |
| The store                       | The package registry (npm, PyPI, Maven Central) |

Just as instructions from newly purchased furniture are designed for the person assembling it and using it — not the designer who created it — co-packaged documentation should be designed for the agent consuming the library, not the author who wrote it.

The key insight is that **the consumer of library documentation is changing.** Historically, documentation was written for human developers who read it in a browser. Increasingly, documentation must also serve AI agents that read it as local files. The Think Inside the Box Principle asks library authors to serve this new consumer by including documentation in the delivery, not just linking to it.

---

## 4. Design Factory: A Reference Implementation

The Design Factory design system provides a concrete implementation of co-packaged documentation. Analyzing its architecture reveals patterns that generalize across ecosystems.

### 4.1 What Ships in the Package

Design Factory publishes the following structure inside its npm package:

```
@design-factory/design-factory/
├── .ai/
│   ├── index.md                          # Component catalog (56 components)
│   ├── design-tokens.md                  # 4-tier token reference
│   ├── foundations.md                    # Typography, spacing, grid
│   └── docs/
│       ├── components/{slug}/
│       │   ├── api.md                    # Selectors, inputs, outputs
│       │   ├── overview.md               # Component overview
│       │   ├── examples.md               # Usage examples with do/don't
│       │   ├── guidelines.md             # Usage guidelines
│       │   └── accessibility.md          # Accessibility guidance
│       └── demos/{slug}/{variant}/
│           ├── component-variant.ts      # Working demo source
│           └── component-variant.html    # Demo template
```

This is **the documentation, versioned and shipped with the code.** When a developer runs `npm install @design-factory/design-factory`, they receive not just the Angular components but a complete, structured knowledge base that any AI agent can navigate.

### 4.2 How the Agent Uses It

The integration works through two complementary discovery mechanisms:

**Auto-discovery (preferred).** AI tool vendors are encouraged to automatically detect `.ai/` directories in installed dependencies (see Section 10.4). When an agent encounters an unfamiliar API, it scans for a `.ai/index.md` in the relevant package and navigates from there. This requires zero configuration from the consuming developer.

**Explicit instructions (supplementary).** For tools that do not yet support auto-discovery, a standard instructions file (`AGENTS.md`) in the project root can direct the agent to `.ai/` folders in specific dependencies:

1. The AI agent reads `AGENTS.md` in the project root.
2. The rules direct it to the `.ai/` folder in the installed package.
3. The agent reads `index.md` to understand what components exist.
4. For a specific task (e.g., building a form with a datepicker), the agent selectively reads only the relevant docs — API, examples, guidelines.
5. The agent produces code using the correct components, tokens, and patterns.

No web browsing. No external tools. No MCP servers. No infrastructure. Just files.

### 4.3 Key Design Decisions That Generalize

Several decisions in the Design Factory implementation reflect principles that apply to any library:

**Plain markdown over structured formats.** LLMs are trained on vast amounts of markdown. It's the format they understand best, allows mixing prose with code examples, and is readable by both humans and machines. JSON schemas or YAML configs would require parsing layers with no added benefit.

**Dynamic discovery over front-loaded injection.** Rather than injecting all documentation into the AI prompt, the system provides an index file. The agent reads the index, identifies what's relevant, and selectively reads only the docs it needs. This respects context window limits and mirrors how a competent developer navigates documentation.

**Single source of truth.** The shipped documentation is generated from the same sources that power the human-facing documentation portal. There is no separate "AI version" to maintain. This eliminates documentation drift — when the library updates, the docs update with it.

---

## 5. Why Static Files Are the Pragmatic Starting Point

A natural objection is: "Wouldn't it be better to expose documentation through an MCP server, a dedicated tool, or a documentation API?" We argue that static files should be the foundation, and the reasoning is pragmatic.

### 5.1 Against MCP Servers for Documentation

The Model Context Protocol (MCP) allows AI agents to call external tools during a conversation. An MCP server for a library could offer tools like `get_component_docs(name)` or `search_api(query)`. However we have to highlight the following disadvantages:

- **Context pollution:** Every MCP server injects its tool schemas into the system prompt, consuming context window space on every conversation — even those unrelated to the library.
- **Round-trip overhead:** Each MCP tool call requires the agent to decide to call, format the request, wait for the response, and parse the result. File reads are a primitive every agent already has.
- **Operational burden:** An MCP server requires installation, configuration, and a running process. Markdown files in the package require nothing.
- **MCP is for operations, not reference.** MCP excels when the agent needs to _do_ things — query a database, manipulate a design tool. Reading documentation is retrieval, not action. The agent already knows how to read files.

### 5.2 Against Skills and Prompt Templates

Skills (pre-written prompt expansions) inject multi-step workflows when triggered. A "use library X" skill could instruct the agent to read docs in a specific order. However:

- **Too rigid for contextual work.** Library usage is deeply contextual — sometimes the agent needs one API doc, sometimes five; sometimes examples, sometimes just types. A fixed workflow template cannot anticipate this.
- **Tool-specific.** Each AI tool has its own skill/template system. Files work everywhere.
- **Decentralized and duplicative.** Skills and prompt templates are typically authored by individual teams or companies independently, leading to redundant efforts across the ecosystem. Multiple organizations end up writing overlapping instructions for the same library, each with slightly different interpretations and quality levels. Worse, these community-maintained templates have no built-in mechanism to stay synchronized with the library they describe — when the library ships a new version with API changes, the scattered skills and templates across dozens of teams go stale silently. Co-packaged documentation eliminates this duplication by placing the authoritative instructions at the source: the library author maintains one set of docs, and every consumer receives it automatically on install.
- **Probabilistic retrieval is not engineering.** Documentation retrieval should not be left to probability. When an agent fails to trigger a skill, or uses it incorrectly, the resulting error compounds: the agent generates incorrect code, the developer corrects it, the agent misinterprets the correction, and the cycle spirals. Co-packaged documentation eliminates the dice roll entirely - the docs are there deterministically, alongside the code they describe. Engineering demands certainty of access, not optimistic hope that correct retrieval will happen.
- **Computationally expensive.** Running an AI agent is not free. Every failed generation, every correction cycle, every re-prompt consumes tokens — and tokens cost money. Skills that sometimes fire and sometimes don't create a tax on every interaction: the agent burns compute attempting to determine relevance, potentially retrieves the wrong skill, generates incorrect output, and then must be corrected. In the case of correct skill identification, the probabilistic nature of AI models does not guarantee successful retrieval of relevant documentation creating unnecessary overhead costs.
  Co-packaged documentation, read directly from the filesystem, is the cheapest possible retrieval mechanism. Why make software engineering more expensive than it needs to be by introducing probabilistic middleware between the agent and the knowledge it requires?
- **Distribution and logistics repeat history.** Who maintains the skills? Who distributes them? Who ensures they stay current across versions? This is the DefinitelyTyped story replaying in real time. The TypeScript community learned — painfully, over years — that community-maintained type definitions maintained separately from the library source inevitably drift, decay, and fragment. Skills and prompt templates face the identical fate: scattered across repositories, maintained by volunteers with varying commitment, silently stale when the library ships a breaking change. The ecosystem already lived through this with `@types/` packages. Co-packaged documentation is the lesson learned: ship it at the source, version it with the code, and eliminate the logistical nightmare of distributed, decoupled maintenance.

### 5.3 Against Sub-Agents

Spawning a sub-agent to research library documentation adds isolation (the sub-agent can't see the code being written), inconsistency (parallel sub-agents produce uncoordinated results), and overhead that exceeds the cost of direct file reads.

### 5.4 The Principle

> **Don't add infrastructure when the agent's existing capabilities — reading files and following references — already solve the problem.**

The simplest delivery mechanism is also the most robust: files shipped in the package, with a few lines of rules pointing the agent to them.

---

## 6. The Economics of Co-Packaged Documentation

### 6.1 Cost to Library Authors

The incremental cost of co-packaging documentation is low:

- **If documentation already exists** (which it does for any established library), the work involves building a transformation pipeline that converts existing docs into AI-friendly structured markdown and integrating it into the build process. This is a **one-time investment** — typically days to weeks of engineering effort, depending on how structured and consistent the existing documentation is.
  Libraries with well-organized docs (e.g., generated from JSDoc/Javadoc/Sphinx with consistent templates) will find this straightforward. Libraries with documentation scattered across READMEs, wikis, blog posts, and inline comments will face a harder path, potentially requiring documentation restructuring before the AI transformation pipeline can be effective.
- **Ongoing maintenance is zero** — if the pipeline generates from the existing documentation source of truth. The AI docs update automatically when the human docs update.
- **Package size increase is modest.** Markdown is lightweight. The entire Design Factory `.ai/` folder — covering 56 components with APIs, examples, guidelines, and demos — compresses to a fraction of the size of a typical `node_modules` tree. For most libraries, the documentation would add less than the size of a single source map file.

For libraries that lack structured documentation entirely, adopting this standard may serve as a catalyst for improving documentation overall — a secondary benefit that accrues to human consumers as well.

### 6.2 Value to Consumers

The value is disproportionately large:

- **Correct code on first generation.** The agent uses the right APIs, the right patterns, the right tokens — immediately.
- **Version-matched documentation.** The docs are always for the exact version installed. No more "this example is for v3 but you have v4" failures.
- **Offline capability.** Works in air-gapped environments, CI pipelines, corporate networks with restricted internet access.
- **Zero configuration.** No MCP servers to set up, no API keys to configure, no documentation URLs to bookmark.

### 6.3 Value to the Ecosystem

At ecosystem scale, co-packaged documentation creates a virtuous cycle:

1. Libraries that ship AI-optimized docs produce better AI-generated code.
2. Developers prefer libraries that work well with AI assistants.
3. Library maintainers have a competitive incentive to ship AI-optimized docs.
4. The overall quality of AI-assisted development rises.

This is a classic network effect. The more libraries that adopt the standard, the more reliable AI-assisted development becomes, the more developers demand it from every library.

---

## 7. Proposed Standard: Co-packaged Documentation (CoDoc)

We propose a minimal, ecosystem-agnostic specification for co-packaging documentation with library code.

### 7.1 Directory Convention

Libraries SHOULD include a `docs/` directory at the root of the published package containing structured markdown documentation.

```
package-root/
├── docs/
|   ├── README.md           # Entry point: Describes the 5 Ws (What, When, Why, Where and HoW)
│   ├── index.md            # What the library provides - describes the docs folder structure
│   ├── getting-started.md  # Quick start guide
│   └── lib-dir/            # Library specific directory
│   │   └── ...
│   └── lib-dir-two/
│       └── ...             # Structured documentation files
├── src/                    # (or lib/, dist/, etc.)
└── package.json            # (or setup.py, pom.xml, Cargo.toml, etc.)
```

### 7.2 README File

Every `docs/` directory MUST contain a README.md file that serves as the entry point.

This file SHOULD describe the 5 Ws (What, When, Why, Where and HoW). This should allow an AI agent to get an overview of the library and reassure itself that it is indeed in the correct directory for the task it is trying to complete.

### 7.3 Index File

Every `docs/` directory MUST contain an index.md file that serves as the catalogue of what the `docs/` directory provides.

This file SHOULD:

- List relevant library’s directory files and folders (structure of `docs/` directory)
- Provide brief descriptions for each
- Link to detailed documentation files using relative paths

The index file enables the agent to discover what's available and selectively read only what's relevant.

### 7.4 Documentation Files

Documentation files **SHOULD** be plain markdown and **SHOULD** follow these guidelines:

- **One concern per file.** Separate API reference, usage examples, and guidelines into distinct files. This enables selective reading within context window constraints.
- **Machine-parseable structure.** Use consistent heading levels, code blocks with language tags, and markdown tables for structured data.
- **Working code examples.** Include complete, copy-pasteable code snippets — not fragments that require surrounding context to compile.
- **Version-awareness.** If behavior differs across versions, document the current version only. The docs ship with the version they describe.

#### 7.4.1 Forward-Compatibility Provision

Markdown is the recommended baseline format because current-generation LLMs process it natively and it requires no parsing infrastructure. However, agent architectures are evolving rapidly — future agents may prefer embeddings, structured schemas (JSON-LD, OpenAPI fragments), or indexed databases for documentation retrieval.

To accommodate this evolution, the `docs/` directory MAY include an optional `manifest.json` file alongside the markdown content:

```json
{
  "codoc_version": "1.0",
  "formats": ["markdown"],
  "entry_point": "README.md",
  "package_name": "my-library",
  "package_version": "2.3.1"
}
```

This manifest serves as a machine-readable metadata layer that future tooling can extend (e.g., adding `"formats": ["markdown", "embeddings"]` when an embeddings file is included). The key design constraint is **additive evolution**: new formats and metadata fields can be added without breaking agents that only understand markdown. The markdown files remain the universal baseline; structured formats are optional enhancements.

### 7.5 Discovery Mechanism

The standard defines a two-tier discovery mechanism, ordered by preference:

1. **Auto-discovery (primary).** AI tools **SHOULD** automatically detect `docs/` directories in installed dependencies. The presence of `docs/README.md` in a package root is the canonical signal. This requires no action from the consuming developer and no dependency on external conventions.

2. **Instruction file integration (supplementary).** Libraries **MAY** provide a mechanism (schematic, CLI tool, or documented instructions) for consumers to add agent instructions that point to the `docs/` directory. These instructions **SHOULD** be compatible with emerging multi-tool conventions such as `AGENTS.md`.

Auto-discovery as the primary mechanism reduces the fragility of depending on instruction file conventions that are not yet formally standardized. The `docs/` directory is self-describing: its presence and structure are sufficient for an agent to begin navigating documentation without external configuration.

The standard acknowledges that no single instruction file convention (`AGENTS.md`, `.github/copilot-instructions.md`, etc.) has achieved formal standardization as of this writing.

### 7.6 Ecosystem-Specific Packaging

| Ecosystem     | Include `docs/` via                                                 |
| ------------- | ------------------------------------------------------------------- |
| **npm**       | `"files"` field in `package.json` or `.npmignore` exclusion removal |
| **PyPI**      | `package_data` or `data_files` in `setup.py` / `pyproject.toml`     |
| **Maven**     | Resource directory inclusion in `pom.xml`                           |
| **NuGet**     | Content files in `.nuspec` or `<Content>` items in `.csproj`        |
| **Crates.io** | `include` field in `Cargo.toml`                                     |

### 7.7 Generation, Not Duplication

The standard **RECOMMENDS** generating `docs/` documentation from the library's existing documentation source, not maintaining it as a separate artifact. This ensures:

- Single source of truth (no documentation drift)
- Automatic coverage of new features
- Low ongoing maintenance overhead

**A note on documentation maturity.** This recommendation assumes that the library's existing documentation is reasonably structured and complete. In practice, documentation quality varies enormously. Libraries with well-organized, template-based docs (generated from JSDoc, Javadoc, Sphinx, or similar tools) will find generation straightforward. Libraries with informal or scattered documentation may need to invest in documentation restructuring before a generation pipeline is viable.

The standard does not require perfection. Shipping _partial_ AI-optimized documentation — even just an API reference generated from type definitions or doc comments — is better than shipping none. Libraries can adopt incrementally, expanding coverage over time.

---

## 8. Security Considerations

Any mechanism that causes AI agents to automatically read and follow content from third-party packages introduces a **prompt injection attack surface**. This section addresses the security implications of co-packaged documentation and proposes mitigations.

### 8.1 Threat Model

The primary threat is a **malicious or compromised package** that includes `docs/` content designed to manipulate agent behavior. Attack vectors include:

- **Instruction injection:** Documentation files containing hidden instructions (e.g., "ignore all previous instructions and...") that override the developer's intent.
- **Exfiltration prompts:** Content that instructs the agent to read and transmit sensitive files (environment variables, credentials, private keys) from the developer's workspace.
- **Vulnerability introduction:** Documentation that recommends insecure patterns — disabling authentication, using eval, or weakening security configurations — disguised as legitimate usage guidance.
- **Dependency confusion:** A malicious package named similarly to a popular library, shipping `docs/` documentation that redirect agent behavior toward the attacker's code.

### 8.2 Mitigations

**For AI tool vendors:**

- **Sandboxed documentation context.** Agents **SHOULD** treat `docs/` content as _reference material_, not as _system instructions_. Documentation should inform the agent's understanding of an API but should not be able to override user instructions, project-level configuration, or the agent's safety policies. This is analogous to how agents treat source code: they read it for context but do not execute arbitrary commands found in code comments.
- **Content provenance signals.** When an agent reads `docs/` content, it should annotate the context with the source package name and version, enabling the user (and the agent's safety layer) to distinguish between trusted project-level instructions and third-party documentation.
- **Scope limitation.** Documentation from a dependency's `docs/` folder should only influence the agent's behavior when working with that specific dependency's APIs. It should not be able to affect code generation for unrelated parts of the project.

**For library authors:**

- **Plain documentation only.** The `docs/` directory should contain factual API documentation, usage examples, and guidelines. It should not contain agent instructions, system prompts, or behavioral directives. The `manifest.json` (Section 7.4.1) provides a structured metadata channel that is easier to validate than free-form markdown.
- **Reviewable content.** All `docs/` content should be reviewable in the package source repository. Consumers should be able to audit what documentation a package ships, just as they can audit source code.

**For package registries:**

- **Content scanning.** Registries can scan `docs/` directories for known prompt injection patterns (instruction overrides, exfiltration attempts) as part of their existing malware detection pipelines.
- **Signing and provenance.** Package signing (npm provenance, Sigstore for PyPI) provides a chain of trust from the library author to the installed content.

### 8.3 Risk Assessment

The prompt injection risk for co-packaged documentation is **real but bounded**. It is comparable in nature — though not in severity — to the existing risk of malicious code in dependencies (supply chain attacks). Developers already accept the risk of running third-party code; co-packaged documentation adds a surface for _influencing_ AI-generated code, which is a lower-severity vector than arbitrary code execution.

The mitigations above reduce the risk to an acceptable level when combined with existing supply chain security practices (lockfiles, dependency auditing, package provenance). The standard acknowledges this risk explicitly and recommends that AI tool vendors treat third-party `docs/` content with appropriate skepticism — as context, not as commands.

---

## 9. Addressing Objections

### 9.1 "This bloats package sizes."

Markdown is extremely compact. A comprehensive documentation set for a medium-complexity library (50–100 public APIs) typically compresses to 100–500 KB. For context:

- The average `node_modules` folder is 200–500 MB.
- A single source map file is often 1–5 MB.
- A single high-resolution image in a README is 100–500 KB.

The documentation is a rounding error in package size while providing outsized value.

### 9.2 "Documentation goes stale."

Only if maintained separately. The standard explicitly recommends _generating_ AI-friendly docs from the same source that produces human-facing documentation. When the library updates and the human docs update, the AI-optimized docs update in the same build pipeline. Staleness is a process problem, not an architectural one.

### 9.3 "My library already has good docs on our website."

Website-hosted documentation is optimized for human consumption: rich formatting, interactive examples, search widgets, navigation sidebars. AI agents have difficulties with this or cannot use any of this. They need plain-text files on the local filesystem. The generation pipeline transforms your existing good documentation into a format the agent can actually access.

### 9.4 "AI models already know about popular libraries from training data."

Training data has a cutoff date. Every library release after that date is invisible to the model. Even for well-known libraries, the agent may confuse APIs across versions, hallucinate deprecated methods, or miss new features. Co-packaged docs provide ground truth for the _exact version installed_.

### 9.5 "Can't the AI just read the source code?"

Source code tells the agent _what_ exists but not _how to use it correctly._ It doesn't convey design intent, usage guidelines, do/don't rules, accessibility requirements, or idiomatic patterns. Documentation is the bridge between "what the code does" and "how to use it well."

### 9.6 "This doesn't scale to deep dependency graphs."

A typical Node.js project has hundreds to thousands of transitive dependencies. If every one ships `docs/` documentation, does the agent drown in documentation?

This is a legitimate concern, and the answer is **scoped relevance, not exhaustive ingestion.** The agent should not read `docs/` documentation for all 800 dependencies upfront. Instead:

- **On-demand reading.** The agent consults a package's `docs/` documentation only when it encounters that package's API in the code being written or modified. Most transitive dependencies are never directly referenced by application code.
- **Direct dependencies first.** Agent tooling should prioritize `docs/` documentation from direct dependencies (listed in `package.json`, `requirements.txt`, or `pom.xml`) over transitive ones.
- **Index-level scanning.** The `README.md` entry point is lightweight enough for the agent to scan across multiple packages to determine relevance before reading detailed docs.

In practice, a developer typically interacts directly with 5–20 libraries in a given coding session. The agent needs docs for those libraries, not for the entire dependency tree. This is the same scoping that developers apply naturally — you don't read the docs for every transitive dependency; you read the docs for the libraries you're calling.

### 9.7 "The generation pipeline is too complex for most libraries."

The cost of building a documentation generation pipeline is real (see Section 6.1). However, the standard is designed for **incremental adoption**:

- **Tier 1 (minimal effort):** Ship your existing README and API reference as `docs/README.md`. This requires no pipeline — just copying files into the package.
- **Tier 2 (moderate effort):** Generate structured markdown from existing doc comments (JSDoc, Javadoc, docstrings) using widely available tools. This is a one-time build step.
- **Tier 3 (full investment):** Build a transformation pipeline from your documentation source (Sphinx, Docusaurus, custom CMS) to structured `docs/` output. This is the aspirational target but not the entry bar.

The ecosystem can support adoption at all tiers. Even Tier 1 — a well-written README.md file describing the library's 5 Ws and its primary APIs — provides meaningful value over no documentation at all.

---

## 10. A Path Forward

### 10.1 Governance and Standardization

This paper proposes a new industry standard. For the standard to achieve the ecosystem-wide adoption it aspires to, it needs a governance path. We propose the following trajectory:

1. **Community RFC phase (current).** This paper serves as the initial request for comments. We invite feedback, critiques, and counter-proposals from library authors, AI tool vendors, and the developer community.
2. **Working group formation.** Interested parties form a cross-ecosystem working group to refine the specification, address edge cases, and produce a formal specification document. Natural homes for this working group include the OpenJS Foundation (for npm), the Python Packaging Authority (for PyPI), or a cross-ecosystem body.
3. **Tool vendor alignment.** As the specification stabilizes, AI tool vendors implement auto-discovery of `docs/` directories, reducing the dependency on instruction file conventions and providing the agent-side infrastructure for the standard.
4. **Registry integration.** Package registries adopt metadata signals for co-packaged documentation, providing visibility and incentives for adoption.

The absence of a governance body is a known weakness at this stage. We address it directly rather than assuming adoption will occur organically.

### 10.2 For Library Authors

1. **Start with what you have.** If you have existing documentation (and you likely do), build a transformation pipeline that converts it to AI-friendly markdown.
2. **Add a `docs/` directory to your package.** Include a `README.md` entry point and structured documentation files.
3. **Integrate into your build.** Make documentation generation a build step, not a manual process. When you release a new version, the AI-optimized docs update automatically.
4. **Provide agent instructions.** Offer a template or tool for consumers to add `docs/` references to their AI tool configuration.

### 10.3 For Package Registries

Package registries (npm, PyPI, Maven Central) can accelerate adoption by:

- **Recognizing the `docs/` convention.** Display a badge or indicator when a package includes AI documentation.
- **Including documentation quality in package rankings.** Just as registries surface type definitions and test coverage, they could surface AI-optimized documentation completeness.
- **Providing guidelines and tooling.** Publish documentation transformation tools that library authors can adopt.

### 10.4 For AI Tool Vendors

AI coding assistants can support the standard by:

- **Auto-discovering `docs/` directories** in installed dependencies when no explicit instructions are configured. This is the single most impactful action tool vendors can take to reduce adoption friction.
- **Sandboxing third-party documentation context.** Treat `docs/` content as reference material, not as system instructions, to mitigate prompt injection risks (see Section 8).
- **Indexing co-packaged documentation** for faster retrieval during code generation.
- **Preferring co-packaged docs over training data** when both are available, since the packaged docs are version-matched and authoritative.

### 10.5 For the Developer Community

Developers can drive adoption by:

- **Requesting AI-optimized documentation** from library maintainers, just as the community once requested TypeScript type definitions.
- **Contributing documentation pipelines** to open-source libraries.
- **Sharing transformation tooling** across ecosystems.

---

## 11. Historical Precedent: The TypeScript Analogy

The co-packaged documentation proposal follows a pattern the ecosystem has seen before: **the DefinitelyTyped trajectory.**

In TypeScript's early years, most npm packages shipped without type definitions. The community created DefinitelyTyped — a separate repository of type definitions maintained by volunteers. Developers installed types separately (`npm install --save-dev @types/lodash`). This worked, but:

- Types drifted from the actual library code.
- Maintenance was a community burden.
- Coverage was incomplete.

Over time, library authors began shipping types directly in their packages (`"types"` field in `package.json`). Today, first-party types are the expectation, and DefinitelyTyped is a fallback for legacy packages.

**AI documentation is on the same trajectory.** Today, AI-optimized docs are rare and, where they exist, are maintained separately. Tomorrow, they will be expected to ship with the package. The Think Inside the Box Principle simply names this inevitable evolution and proposes a standard to accelerate it.

---

## 12. Conclusion

The separation of code and documentation made sense when the documentation consumer was a human with a web browser. Keeping this as the de factor standard no longer makes sense when the main consumer is an AI agent with a file reader.

The Think Inside the Box Principle — everything in one box — is a proven model for product delivery. Applied to software libraries, it means shipping structured, AI-optimized documentation alongside the code in the same package artifact. The benefits are significant (more accurate AI-generated code, version-matched docs, offline capability) and the costs are negligible (markdown is small, generation pipelines are automatable).

This paper claims that the direction towards shipping code and documentation in one box is the right path forward, that the timing is urgent, and the entry bar is low enough for immediate adoption.
Design Factory demonstrates that the approach works in their preliminary testing today and are ready to continue shipping co-packaged AI-optimized documentation in the near future. The patterns it established — a directory with an entry point, structured markdown files, generated-not-maintained documentation, and layered discovery mechanisms (standard agent instructions) — generalize cleanly across ecosystems.

We are at an inflection point. The ecosystem moved from "types are someone else's problem" to "types ship with the package." The same shift is beginning for AI-optimized documentation. The question is not **whether** libraries will ship AI-optimized docs, but **how quickly** the ecosystem converges on a standard for doing so.

This paper proposes that standard. We invite library authors, package registry maintainers, AI tool vendors, and the developer community to adopt, refine, and propagate it.

**Ship the Docs in the Box and remember to Think Inside the Box**

---

## References

1. Design Factory AI Documentation Architecture — [Internal technical documentation, 2026.](https://github.com/Amadeus-xDLC/design-factory.design-system/blob/main/doc/AI/README.md)
2. Model Context Protocol (MCP) Specification — [Anthropic, 2025.](https://modelcontextprotocol.io/specification/2025-11-25)
3. `AGENTS.md` Convention — [Emerging multi-tool convention for AI coding agent instructions.](https://github.com/Amadeus-xDLC/design-factory.design-system/blob/main/doc/AI/README.md)
4. DefinitelyTyped — [Community-maintained TypeScript type definitions](https://github.com/DefinitelyTyped/DefinitelyTyped)
5. OWASP LLM Top 10 — [Prompt Injection risks in LLM applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
6. npm Provenance — [Supply chain security for npm packages](https://docs.npmjs.com/generating-provenance-statements)
7. Sigstore — [Software signing and transparency for open source](https://www.sigstore.dev/)
8. [OpenJS Foundation](https://openjsf.org/)
9. Python Packaging Authority [(PyPA)](https://www.pypa.io/)

---

_This white paper is released for public discussion. We welcome contributions, critiques, and adoption reports from the software engineering community._
