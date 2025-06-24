# Use of AI Agents in Software Engineering

## Overview

In this day and age, AI systems have developed beyond simple text completion and entered the "agentic" era.
This period is defined by the introduction of systems with the following key differentiators:

1. **Context**: The output of the system is no longer based solely on training data alone, but biased with context prompts (eg. repository-specific patterns).
2. **Observations**: Through prompting and heuristics, the system is endowed with the ability to make state observations (eg. execution outputs).
3. **Actions**: The system has a notion of performing "actions" through both trained and prompted mechanisms (eg. tool use).
4. **Execution Loop**: Observe, reason, act and repeat (eg. ReAct framework).

The confluence of these aforementioned characteristics give rise to autonomous systems (agents) that are capable of performing high-level tasks with reduced supervision.
The contextualization, reasoning, observation, and restriction of action space are effective strategies for mitigating the shortfalls of modern LLMs: they focus and preserve context in large tasks.

To make effective use of AI agents, you must treat them as senior engieners with junior tendencies; they have the semantic "knowledge" of a senior engineer while having the "experience" of a mere junior.

This document focuses specifically on the Cursor + Claude Code development experience.

### Claude Code vs. Cursor?

The two tools are complementary and should be applied for specific scenarios that leverage their strengths:

#### Claude Code

Cycle time: 1-10 minutes

Claude Code is very good at understanding large contexts, processing mixed formats, and reasoning both with documentation and code.
Use Claude Code when you are working at the repository level or across many files with mixed format.
Using Clause Code to refactor/fix individual files is quite slow and expensive; it can do it, but you have much better options.

#### Cursor

Cycle Time: 5-60s

Cursor is fast and works well with focused contexts, but requires more specific prompting.
Use Cursor to work with individual files or snippets within files.
Cursor performs poorly compared to Claude Code when dealing with large contexts; you need to be very specific and guide its reasoning along.

## Using Claude Code

Claude Code is a CLI coding agent that can write code and documentations end-to-end with the right amount of instruction and encouragement.
On first run of Claude Code, it is advisable to run `/init` for Claude to read through your codebase and create a `CLAUDE.md` document.
Subsequent runs rely on this `CLAUDE.md` as long-term memory for repository specific details (eg. patterns, design choices, memories that you have given to Claude).

Clear and specific requirements typically result in more successful outcomes.
Think to yourself: "What would a junior engineer need, in terms of project requirements, to execute the project to my desired outcome?"

### Setting up MCP tooling

Your agent is as good as the tools you give it! Augmenting [this guide on Reddit](https://www.reddit.com/r/ClaudeAI/comments/1jf4hnt/setting_up_mcp_servers_in_claude_code_a_tech/):

```bash
# Filesystem
claude mcp add filesystem -s user -- npx -y u/modelcontextprotocol/server-filesystem ~/Documents ~/Desktop ~/Downloads ~/Projects

# Puppeteer
claude mcp add puppeteer -s user -- npx -y u/modelcontextprotocol/server-puppeteer

# Web Fetching
claude mcp add fetch -s user -- npx -y u/kazuph/mcp-fetch

# Browser Tools
claude mcp add browser-tools -s user -- npx -y u/agentdeskai/browser-tools-mcp@1.2.1

# Serena Code Semantic Search
claude mcp add serena -- uvx --from git+https://github.com/oraios/serena serena-mcp-server --context ide-assistant --project $(pwd)

# Check whats been installed
claude mcp list
```

I have found that [Serena](https://github.com/oraios/serena) greatly enhances the development efficiency of Claude Code by providing more focus in the code context, but your mileage may vary!

### Optimizing Documentation

To prepare your repository for Claude Code, it is helpful to prepare a few key pieces of documentation.
Create a `docs/` folder in the repository root, and add the following to Claude Code's memory (with `#` command):

```text
All repository-level documentation is located under the `docs/` folder. 
General coding standards and design patterns are documented in `docs/design_patterns.md`.
For database and ORM patterns, follow guidelines in `docs/database_schema.md`.
For testing, follow guidelines in `docs/testing.md`.

Feature-specific documentation can be found in `docs/features/<feature_name>`.
```

Create the aforementioned documentation.

#### Developer loop/interface

I have opted to use `Makefile` to specify the "API" for my repositories; you can also use the `package.json` or other script-management systems when applicable.

To ensure Claude Code can understand how to work with your repository in a standardized way, add the following memory:

```text
This repository is controlled by a standardized interface for linting, building, testing, and running the system in development mode documented in `./docs/makefile.md`
Use these commands exlucisvely to write new code, debug, and test your work.
Follow the "Usage" section of `makefile.md` for your development loop.
```

Add a `docs/makefile.md` file:

```markdown

# Makefile API

- `install`: Installs project-level dependencies (eg. `npm install`, `uv sync`)
- `setup`: Installs all system and project-level dependencies (eg. `docker`, `uv`, `npm`)
- `build`: Performs a production build, if applicable
- `lint`: Lint check
- `fix`: Fix all auto-fixable linter errors
- `typecheck`: Check for type errors, if applicable
- `check`: Runs `lint` and `typecheck`
- `test`: Runs the test suite(s)
- `test:one`: Runs a single test
- `apispec`: Generate the OpenAPI YAML for this repository, if applicable
- `dev`: Run the system in development mode

## Usage

Before finishing your work, run `make check && make test` to ensure the entire codebase conforms to the design standards.
Use `make test:one` to run individual tests.

```

### Developing Net New Features

1. Start by drafting a Project Requirements Document (PRD) for the new feature in Markdown with the following format:

    ```markdown
    # <feature> Project Requirements

    ## Functional Description
    A plaintext description of what <feature> is, why is it important, and how it works roughly.
    Include specifics like: 

    - <feature> takes in user input with `InputSchemaType` and outputs `OutputSchemaType`
    - <feature> performs XYZ transform on the input to produce the output
 
    ## Constraints
    A list of hard requirements that MUST be met with "shall" terminology:
    - Solution SHALL use `acme` library
    - Feature SHALL validate inputs and throw error

    ## Success Criteria
    Mixed plain text and bullet point description of how the implenentor can know that the feature is correctly implemented.
    - Example input + expected output pairs
    - <feature> exposes CRUD endpoints such that `XResource` can be created, replaced, updated, and deleted with REST endpoints
    - <feature> follows design patterns specified in `./docs/design_patterns`
    Write this in a way that you would hand off to any other engineer on the team.

    ## Testing Criteria
    Follow test patterns set out in `./docs/test_patterns.md`.
    Specific testing requirements:
    - Ensure >90% test coverage for the core files for <feature>
    - Test edge case `c`
    - Create fixtures of `XResource` and `YResource` and use them in all tests for <feature>
    ```

2. Prompt Claude Code to create an action plan

    ```text
    Let's role play: You are a principal [Python/ML/Node/ReactJS] engineer who exclusively developed and maintained this repository. You lead a team of junior engineers will implement the <feature> documented in `docs/features/<feature>_requirements.md`. 

    You are focused on design simplicity, codebase/test scalability, conformance to existing design/test patterns and standards listed in `docs/`, and adherence to coding/testing best practices for Python.
    
    Make sure it is sufficiently detailed and complete such that you can hand this off to your junior engineering team to implement without any ambiguity.

    Task: Write a comprehensive implementation (`docs/features/<feature>_implementation.md`) plan with step-by-step instructions and key technical decisions (with justification) to implement <feature>.
    ```

3. Loop: Review the generated implementation plan and prompt Claude to iteratively refine until you both are satisfied with its specificity.

    ```text
    Review the implementation plan you have created and cross-check it against the <feature>_requirements.md. You are focused on design simplicity, codebase scalability, conformance to existing design patterns and standards listed in `docs/`, and adherence to coding best practices for Python.

    Task: If the implementation plan is incomplete, incorrect, or insufficiently thought out, revise the implmentation accordingly
    ```

    This step is similar to the question refinement that the OpenAI Deep Research agent performs, but with more loops. Claude will almost always converge to a "the implmentation plan is sufficient".

    For more complex scenarios, it is helpful to use a more advanced model (ie. Opus) to refine the design. In this case, you can switch the model to Opus and prompt:

    ```text
    You are smarter and more technically capable now! Review the implementation plan and ensure that it meets all of the requirements and success criteria set forth in `<feature>_requirements.md`.

    You are focused on design simplicity, codebase scalability, conformance to existing design patterns and standards listed in `docs/`, and adherence to coding best practices for Python.
    ```

4. Once you and Claude are both happy with the implementation plan, `/clear` the session (to avoid context pollution), and prompt for implementation:

    ```text
    You have a requirements document <feature>_requirements.md and a detailed implementation plan <feature>_implementation that your principal engineer has designed. Follow the implementation plan to implement the <feature> and ensure that it meets all requirements as documented in the <feature>_requirements.md.

    Make sure you follow the development cycle documented in makefile.md, write code according to design_patterns.md, and create and execute tests according to testing.md.
    Proceed with your development loop until ALL tests, linting, and typechecking is passing, and the feature you have built meets the success criteria documented in the <feature>_requirements.md! I believe in you; go get em!
    ```

    I have found, empiracaally, these coding agents tend to produce better outputs when given light positive encouragement.

### Documentation

For writing documentation, you should always ensure that Claude Code has the latest context with `/init`.

Then you can prompt Claude to generate specific documentation with:

```text
Create markdown documentation for <component/feature> for an audience of <audience> in `docs/features`.
You should focus on the following aspects:
- Design consideration 1
- Technical choice 2
- Specific component X
Include specific references to code location and attach code snippets when necessary.
```

Avoid asking Claude to work with overly generic statements (eg. "create documentation for this repository"); it is always able to generate documentation, but the output may not always be useful.

**Component/Feature**: Focuses Claude on a very narrow context so it can create much more detailed documentation.
**Audience**: Biases Claude to include the right level of detail.
    - eg. "senior Python ML engineers with a strong background in numpy, pytorch, and statistics" will meaningfully bias Claude to avoid overly verbose documentation on library and statistics basics when documenting ML code.

### Debug/Troubleshooting

When asking Claude to troubleshoot problems, use the following prompt:

```text
Problem Description: I am getting the following error when hitting the `/xyz` endpoint
[pasted lines]

To replicate the error: .venv/bin/pytest tests/test_something.py::failing_test.

Hypothesis: I think this problem could be due to <some factor>.

Explain the root cause of the problem to me step by step, and implement a fix.
```

**Replication steps**: The replication steps is one of the *most important* aspects of the prompt. It will help Claude understand exactly what it needs to run to replicate the problem. Otherwise, it is likely for Claude to go on tangents to generate its own replication steps and waste loops (or get stuck!).
**Hypothesis**: This section is *optional* but can focus Claude on the problem if you know what the problem is and just want Claude to fix it.

## Using Cursor

To make effective use of Cursor and enforce code style and standards across the codebase, you should prompt Claude Code to generate a series of Cursor rules based on the documentation listed in `docs/`.

You may want to include `<feature>_implementation.md` files, in addition to the relevant code files, to the Cursor context when working with individual files or specific snippets. 
