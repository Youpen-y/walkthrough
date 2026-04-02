# Walkthrough

**English** | [**中文**](./README.zh-CN.md) 

A skill that generates interactive HTML walkthroughs with clickable Mermaid diagrams — flowcharts and ER diagrams — to explain codebase features, flows, architecture, and database schemas.

Inspired by [Amp's Shareable Walkthroughs](https://ampcode.com/news/walkthrough).

**[Live demo — walkthrough of the walkthrough skill itself](https://youpen-y.github.io/walkthrough/examples/walkthrough-how-it-works.html)**

## Usage
Trigger the skill with prompts like:

```
walkthrough how does authentication work
explain this flow
walk me through the checkout process
how does X work
database schema
explain the tables
```

The agent will:
1. Explore the relevant parts of your codebase using parallel subagents
2. Synthesize findings into 5-12 key concepts and their connections
3. Generate a single `walkthrough-{topic}.html` file in the project root
4. Open it in your browser

## Examples

**Feature flow:**
> Use the walkthrough skill and explain the process of what happens when a user submits a form.

**Architecture overview:**
> Walk me through how the plugin system is organized.

**Database schema (ER diagram):**
> Use the walkthrough skill and explain how the invites entity is stored in the database. Use an ER diagram.

**Data flow:**
> How does state flow from the composable to the component?

## Installation

### Quick install

```bash
npx skills add https://github.com/Youpen-y/walkthrough --skill walkthrough
```
### Manual install
Copy the `skills/walkthrough/` directory into your project's `.claude/skills/` folder:

```
your-project/
  .claude/
    skills/
      walkthrough/
        skill.md
        references/
          html-patterns.md
```

## Structure

```
skills/walkthrough/
  skill.md                      # Main skill definition
  references/
    html-patterns.md            # HTML template, CSS, and JS patterns reference
```

- **skill.md** — The skill prompt that the agent follows. Defines the workflow: scope understanding, parallel codebase exploration, diagram type selection, and HTML generation.
- **references/html-patterns.md** — Complete reference for the generated HTML files: React component architecture, Mermaid config, Shiki setup, color palette, pan/zoom implementation, and all the patterns needed to produce a working walkthrough.

## Tech stack (generated files)
The output HTML files are fully self-contained with CDN dependencies:
- **Mermaid 11** (ESM) — diagram rendering (flowcharts and ER diagrams)
- **Shiki** (ESM) — syntax highlighting with `vitesse-dark` theme
No build step. Just open the HTML file in a browser.
## Key Features
### Node types with color coding
| Type | Color | Use for |
|------|-------|---------|
| `component` | Purple | Main components, pages, processors |
| `utility` | Blue | Utils, helpers, pure functions |
| `external` | Gray | Libraries, browser APIs, external services |
| `event` | Cyan | Events, user actions, triggers, entry points |
| `data` | Green | Stores, state, data structures, config |
### Optional code snippets
Each node *may* include an optional code snippet (max 5 lines). Most nodes should have no code — only include the single most illuminating piece when truly necessary.
### Node sizing
Nodes use explicit width/height styling via Mermaid's `style` syntax to ensure text fits properly:
### TL;DR summary
Rendered as a collapsible accordion (collapsed by default) so the diagram is the first thing users see.
### Dark mode
Every walkthrough uses a pure black background (`#000000`), white text, and purple accents. Never generate light-mode walkthroughs.
## Testing
The `evals/` directory contains an eval harness that runs the skill against a set of test prompts and grades the output.
### Prerequisites
- `claude` CLI installed and authenticated
- Node.js >= 18
### Running evals
```bash
# Run all 16 test prompts
bash evals/run.sh
# Run only the 4 critical prompts (faster feedback loop)
bash evals/run.sh --subset
# Run a single prompt by ID
bash evals/run.sh --id explicit-01
# Skip the LLM rubric grader (deterministic checks only)
bash evals/run.sh --skip-llm
# Use a specific model (default: sonnet)
bash evals/run.sh --model opus
```
You can also set defaults via environment variables:
```bash
EVAL_MODEL=opus EVAL_MAX_BUDGET=3.00 bash evals/run.sh
```
### How it works
Each eval run:
1. Copies the project into a temp directory with the skill installed
2. Runs `claude -p` with each prompt from `evals/prompts.csv`
3. Collects any generated `walkthrough-*.html` files
4. Runs two graders:
   - **Deterministic** (`graders/deterministic.mjs`) — checks file existence, HTML structure, CDN deps, node count, diagram type
   - **LLM rubric** (`graders/llm-rubric.mjs`) — uses Claude to score readability, descriptions, code snippets, and diagram accuracy against `graders/rubric.md`
5. Generates a summary report in `evals/results/<timestamp>/summary.json`
Results are saved to `evals/results/` (gitignored). A `latest` symlink always points to the most recent run.
### Test prompts
The prompts in `evals/prompts.csv` cover:
- **Explicit triggers** — `$walkthrough how does X work`
- **Implicit triggers** — `walk me through X`, `explain this flow`
- **Diagram types** — flowchart and ER diagram cases
- **Negative cases** — prompts that should *not* trigger the skill
- **Edge cases** — vague prompts, broad scope
