# Contributing to Think Inside the Box

Thank you for your interest in contributing! This project is a **community RFC** — the goal is to refine the CoDoc (Co-packaged Documentation) specification through public discussion and real-world feedback before formal standardization.

All kinds of contributions are welcome: critiques, adoption reports, specification proposals, and website improvements.

## Ways to Contribute

> **Where to post:** open-ended conversation — critiques, ideas, questions, adoption stories — goes to [Discussions](https://github.com/AmadeusITGroup/Think-Inside-the-Box-White-Paper/discussions). Concrete, actionable work items (specific edits, factual errors, website bugs) go to [Issues](https://github.com/AmadeusITGroup/Think-Inside-the-Box-White-Paper/issues).

### 💬 Feedback and critiques

The fastest way to contribute is to tell us where the proposal falls short:

- **Start a [Discussion](https://github.com/AmadeusITGroup/Think-Inside-the-Box-White-Paper/discussions)** to challenge an argument, point out a gap in the analysis, or flag an ecosystem we have overlooked. Pick whichever category fits best (e.g., **Ideas** or **General**).
- Substantive disagreements are especially valuable — the specification gets stronger by surviving scrutiny.

When opening a critique, please include:

- The section of the [white paper](white-paper.md) you are responding to
- A concrete example or counter-example, if you have one
- What you would change, or the question you think needs answering

When a discussion converges on a concrete change, a maintainer will convert it into an issue so it can be assigned and worked on.

### 📦 Adoption reports

Have you shipped (or consumed) co-packaged documentation in a real project? We want to hear about it:

- What ecosystem and registry (npm, PyPI, Maven, …)?
- What did you put in the box — directory layout, formats, entry points?
- How did AI agents perform with and without it?
- What would you do differently?

Share your story in the **🙌 Show and tell** [Discussions category](https://github.com/AmadeusITGroup/Think-Inside-the-Box-White-Paper/discussions/categories/show-and-tell).

### 📝 Specification proposals

Section 7 of the white paper defines the proposed CoDoc standard. To propose a change to the specification itself (naming, directory structure, metadata, discovery rules), please **start a [Discussion](https://github.com/AmadeusITGroup/Think-Inside-the-Box-White-Paper/discussions) first** so the idea can be debated before it becomes a tracked work item. Once a proposal is agreed, it moves to an issue — and then a pull request.

### 🌐 Website improvements

The [website/](website/) directory contains the Astro site that presents the paper. Bug fixes, accessibility improvements, and design refinements are welcome — see [Working on the website](#working-on-the-website) below.

## Editing the White Paper

The paper lives in [white-paper.md](white-paper.md) and the summary in [README.md](README.md). When proposing edits:

1. **Open an issue first** for anything beyond typos and minor clarifications.
2. **Keep the tone** — technical, direct, and evidence-based. Cite sources for external claims.
3. **Preserve the structure** — numbered sections with descriptive headings. If you add or rename a section, update the anchor links in [README.md](README.md) accordingly.
4. **Keep diffs reviewable** — one logical change per pull request; avoid mixing reformatting with content changes.
5. **Do not bump the version number yourself** — the working group assigns versions when changes are merged.

## Working on the Website

The website is an [Astro](https://astro.build) project. To work on it locally:

**Prerequisites:** Node.js >= 22.12.0

```sh
cd website
npm install
npm run dev      # local dev server at localhost:4321
npm run build    # production build to website/dist/
npm run preview  # preview the production build locally
```

Styling conventions:

- Keep light colors as the default theme.
- Place dark-palette overrides under `:global(:root[data-theme='dark'])` so both modes stay consistent.
- The theme toggle persists the selection in local storage via `data-theme` on the document root.

## Pull Request Process

**Before opening a pull request, check the [open issues](https://github.com/AmadeusITGroup/Think-Inside-the-Box-White-Paper/issues) first.** This avoids duplicated effort and keeps everyone's work focused:

1. **Search the open issues** to see whether your fix or enhancement has already been identified.
2. **If a matching issue exists and has no assignee** — feel free to take it: assign it to yourself (or comment asking to be assigned), then open your pull request referencing it.
3. **If a matching issue exists and is already assigned** — coordinate in the issue thread instead of starting parallel work.
4. **If no matching issue exists** — open one first (except for trivial fixes like typos) so the change can be discussed and tracked before you invest time in a PR.

Once you have an issue to work against:

1. Fork the repository and create a branch from `main`.
2. Make your change, keeping the PR focused on a single topic.
3. Reference the related issue (e.g., `Closes #12`) in the PR description.
4. Ensure the website still builds (`npm run build`) if you touched anything under `website/`.
5. A member of the working group will review your PR. We aim to respond within a week.

## Community Expectations

- Be respectful and constructive — critique ideas, not people.
- Assume good faith; this is a proposal, and reasonable people will disagree with parts of it.
- Keep discussions on-topic in issues and PRs so threads stay actionable.

## Questions?

Not sure where your contribution fits? Start a [Discussion](https://github.com/AmadeusITGroup/Think-Inside-the-Box-White-Paper/discussions) and ask — we are happy to point you in the right direction.

---

**Maintained by:** AI Documentation Working Group (Andres Igea Oviedo, Andrea Scorti)
