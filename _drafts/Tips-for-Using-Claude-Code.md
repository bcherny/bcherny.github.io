---
layout: post
title: "Tips for using Claude Code"
date: 2025-04-13
categories: AI
---

Last month, Anthropic [released](https://www.anthropic.com/news/claude-3-7-sonnet) Claude Code, an experimental tool for agentic coding. I created Claude Code as a research project, to be a way for Anthropic engineers and researchers to more natively use Claude as part of their coding workflows.

Claude Code is intentionally low level and unopinionated, giving you as close to raw model access as possible. Code does not force you to use specific workflows, instead opting to be a power tool that is flexible, customizable, scriptable, and safe. This flexibility, while powerful, can be a challenge for engineers that have not worked with agentic coding tools before, at least until they've discovered their own best practices.

In this post, I'll lay out some of the general patterns I've seen work well, both for engineers and researchers using Code at Anthropic, and for engineers using Code for all sorts of codebases, languages, and environments outside of Anthropic. Nothing in this list is set in stone; use it as a starting point, and crucially, experiment and find what works best for you!

_For more in depth docs, see [claude.ai/code](https://claude.ai/code)._

# 1. Tune your environment

Claude Code is an agentic coding assistant, meaning it will automatically pull in context. This can eat up time and tokens, and is something you can improve significantly.

## a. Write a CLAUDE.md

CLAUDE.md is a special file that Claude automatically pulls into context when you start a conversation. Because it is auto-added to context, CLAUDE.md is a conventient place document:

- Common bash commands and MCP tools
- Core files and utility functions
- Code style guidelines
- Memories

There is no specific format your CLAUDE.md needs to use. Personally, I like to keep my CLAUDE.md's short and human-readable. There are a few places you can put CLAUDE.md files:

- The root of your repo, or wherever you run `claude` from. This is the most basic kind of CLAUDE.md, that you will use most of the time. Name it CLAUDE.md and check it into git so that you can share it across sessions and with your team (recommended), or name it CLAUDE.local.md and .gitignore it.
- Any parent of the directory you run `claude` from. This is most useful for monorepos, where you might run `claude` from _root/foo_, and have CLAUDE.md's in both _root/CLAUDE.md_ and _root/foo/CLAUDE.md_. Both of these will be pulled in automatically.
- Any child of the directory you run `claude` from. This is the inverse of the above, and in this case, Claude will pull in CLAUDE.md's on demand when you work with files in child directories.
- Your home folder (in _~/.claude/CLAUDE.md_), so that it applies to all your `claude` sessions.

Add content to your CLAUDE.md's manually, or press the `#` key to give Claude an instruction that it will then automatically incorporate into the relevant CLAUDE.md. I use `#` frequently as I code, to document commands, files, and style guidelines as I go. I then include CLAUDE.md changes in my commits, so that others on my team can benefit.

Learn more in the [docs](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview#manage-claudes-memory).

## b. Tune your CLAUDE.md

Your CLAUDE.md's will be included in the prompt we send to Claude, and should be prompt tuned the same way you'd tune any other frequently used prompt. A common mistake I see is people dumping content into their CLAUDE.md without taking the time to iterate on that content. It's worth taking the time to experiment and see what works best to elicit better instruction following from the model.

I occasionally run my CLAUDE.md's through a [prompt improver](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/prompt-improver), and will often tune instructions (eg. to clarify cases, or add "IMPORTANT", "YOU MUST", etc.) to improve adherence.

## c. Curate Claude's list of allowed tools

By default, Claude Code asks for permission for any action that might result in changes to your system: file writes, many bash commands, MCP tools, etc. This behavior intentionally errs on the side of being conservative out of the box. You should curate the list of allowed tools to allow additional tools that you know are safe, or to allow unsafe tools that are easy to undo (eg. file editing, `git commit`, etc.).

There are four ways to manage allowed tools:

1. Select "Always allow" when prompted for permission to use a tool
2. Start Claude Code, then use the `/allowed-tools` command to add or remove tools from the allowlist. For example, add `Edit` to always allow file edits, `Bash(git commit:*)` to allow git commits, or `mcp__puppeteer__puppeteer_navigate` to allow navigating with the Puppeteer MCP server.
3. Manually edit your _.claude/settings.json_ or _~/.claude.json_
4. To allow tools for a specific session, use the `--allowedTools` CLI flag ([example](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview#automate-ci-and-infra-workflows))

More details in the [docs](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview#permission-rules).

## d. If using Github, install the `gh` CLI

Claude knows how to use the `gh` CLI to interact with Github to create issues and pull requests, read comments, and so on. If you don't have `gh` installed, Claude can also use the Github API or MCP server.

# 2. Common workflows

Claude Code does not force you to use a specific workflow. This means flexibility -- you can use Code the way you want -- and it also means the community has experimented and come up with a number of workflows. Below is a short list of some of the most common ones.

## a. Explore, plan, code, commit

This is a workhorse workflow, useful for a variety of problems. It looks like this:

1. Start by asking Claude to read the relevant files, images, or URLs, providing either general pointers ("read the file that handles logging") or specific filenames ("read logging.py"). Tell Claude not to write any code just yet.
2. Then, ask Claude to make a plan for how to approach a specific problem. Optionally, use the word "think" to trigger extended thinking to more deeply trade off alternatives; use stronger language to encourage longer thinking: "think" < "think hard" < "megathink" < "think harder" < "ultrathink".
3. Ask Claude to implement its solution in code
4. Finally, ask Claude to commit the result

Without #1-#2, Claude will tend to jump to coding a solution. Sometimes, that's what you want. By asking it to research and plan ahead, you can squeeze significantly better performance out of Claude in cases where a problem needs deeper thinking upfront.

## b. Write tests, commit; code, iterate, commit

This is my favorite workflow for changes that are easily verifiable with unit, integration, or e2e tests. There has been a lot written about TDD, and in the age of agentic coding, TDD becomes an even more powerful tool. Here's what it looks like:

1. Ask Claude to write a few tests (eg. by giving it a set of expected input and output pairs), telling Claude not to run the tests yet because we're doing TDD.
2. When you're happy with the tests, ask Claude to commit them
3. Then, ask Claude to write code to make the tests pass. Instruct Claude not to edit the tests, and to keep going until the tests pass. It will usually take a few iterations for Claude to write code, run the tests, adjust the code, and run the tests again.
4. Once you're happy with the changes, commit again

When Claude has a target to iterate against -- a visual mock, a test case, or another kind of output -- its outputs tend to significantly improve. By giving Claude an expected output, it can better make changes, see the results of those changes, and hill climb to get the outputs to match.

## c. Write code, screenshot result, iterate

Similar to (b), you can give Claude a visual mock to iterate against. The workflow looks like this:

1. Give Claude a way to take a screenshot of your browser (eg. with the [Puppeteer](https://github.com/modelcontextprotocol/servers/tree/c19925b8f0f2815ad72b08d2368f0007c86eb8e6/src/puppeteer) MCP server) or simulator (eg. with [this iOS simulator](https://github.com/joshuayoes/ios-simulator-mcp) MCP server)
1. Give Claude a visual mock by copy + pasting the image in, or drag + dropping it in, or giving Claude the file path to the image
1. Tell Claude to write code to implement what's in the mock, then screenshot the result, and repeat until the two match
1. Stop Claude when you're happy with the result, and commit

Like a human, Claude's outputs tend to become significantly better if you give it a way to iterate. The first version might be pretty good, but after 2-3 iterations it will look even better. Give Claude the tools to see its outputs.

## g. Safe yolo mode

Instead of babysitting Claude, you can use `claude --dangerously-skip-permissions` to bypass all permission checks and let Claude keep going until it produces a result. This can be useful for workflows like automatically fixing lint errors or greenfield prototyping where you just want some boilerplate code to get started.

Letting Claude run arbitrary commands is risky, and can result in data loss, your system getting borked, or even data exfiltration (eg. due to a prompt injection attack). To minimize these risks, use `--dangerously-skip-permissions` in a container without internet access. [Here](https://github.com/anthropics/claude-code) is a reference implementation using Docker Devcontainers.

## b. Codebase Q&A

When onboarding to a new company or codebase, use Claude Code to talk to your codebase instead of bugging other engineers. When given general questions -- eg. "how does logging work?"; "how do I make a new API endpoint?"; "what edge cases does `CustomerOnboardingFlowImpl` handle?" -- Claude is able to agentically explore the codebase to intelligently compose an answer by looking at the code.

At Anthropic, using Claude Code for Q&A is a core way that new hires onboard, significantly improving speed of onboarding and reducing load on their teams' engineers. There is no special prompting formula that works best here. Just ask Claude, and it will dig through the code to give an answer.

## d. Use Claude to interact with git

There are a number of ways to use Claude to interact with git. Personally, I use Claude to drive git 90%+ of the time, to:

- Look through git history, to answer questions like "what changes made it into v1.2.3?" or "why is this API designed the way it is?". It helps to explicitly prompt Claude to look through git history to answer queries like these. Similar to Codebase Q&A, this can significantly speed up new hire onboarding for historical questions about why something is the way it is.
- Write commit messages. Claude will look at your changes and recent history automatically, to compose a message taking all the relevant context into account.
- Drive complex git operations like reverting files, resolving rebase conflicts, and comparing and grafting patches

## e. Use Claude to interact with GitHub

I use Claude for 90%+ of my Github interactions, too. A few examples:

- To use Claude to create a Pull Request, just ask. Claude also understands the shorthand "pr". Similarly to how it creates git commits, Claude will diff and gather context to generate a good commit message.
- I use Claude to 1-shot resolutions for simple code review comments: just tell it to fix comments on your PR (optionally, give it more specific instructions) and push back to the remote when it's done
- When the build on a PR is failing, I'll tell Claude to look at the failures and push a fix
- I have Claude loop over open Github issues to categorize and triage them

Similarly to using Claude for git, this means you no longer need to remember `gh` command line incantations, and you can trivially automate a number of toilsome tasks.

## f. Use Claude to work with Jupyter notebooks

Researchers and data scientists at Anthropic use Claude Code to read and write to their Jupyter notebooks. Claude can read outputs, including images, giving you a fast way to explore and interact with your data. You don't need to do anything special to get this to work, but a workflow I like is to have Claude Code and my .ipynb file open side-by-side in VSCode.

# 3. Tips for any workflow

The above tips help with a number of specific workflows. The tips below apply to a broad range of workflows.

## a. Be specific in your instructions

If you give Claude generic instructions, it may get it right in the first shot 30% of the time. As your instructions get more specific, that success rate can improve 2-3x, and reduce the need to adjust course later on.

For example:

- Bad: "add tests for foo.py"
- Good: "write a new test case for foo.py, covering the edge case where the user is logged out. avoid mocks"
- Bad: "why does ExecutionFactory have such a weird api?"
- Good: "look through ExecutionFactory's git history and summarize how its api came to be"
- Bad: "add a calendar widget"
- Good: "look at how existing widgets are implemented on the home page, to understand the patterns and specifically how code and interfaces are separated out. HotDogWidget.php is a good example to start with. then, follow the pattern to implement a new calendar widget that lets the user select a month, and paginate forwards/backwards to pick a year. don't use a library, and instead build it from scratch."

Claude can read between the lines to infer what you mean, but it can't read your mind. Be specific, and the result will be more aligned with what you expect.

## a. Give Claude images

Claude is great at working with images and diagrams, and has a few ways to ingest them:

1. Paste in screenshots (protip: hit cmd+ctrl+shift+4 to screenshot to clipboard on MacOS)
2. Drag and drop images directly into the prompt input
3. Give Claude a filepath to read an image from

This is especially useful when working with design mocks, as an input for Claude to iteratively build a website or app that looks like the mock.

## b. Give Claude URLs

Paste specific URLs along with your prompts to have Claude fetch and read the URLs. To avoid repeatedly being prompted for the same domains (ie. "docs.foo.com"), run `/allowed-tools` and add the domain to your tool allowlist.

## c. Course correct early and often

If you toggle auto-accept mode (shift+enter) and give Claude a prompt, it will pull in files, reason through the problem, and give you code back. The more effort you put into your prompt, the better result you'll get out. However, most of the time you should also nudge Claude along the way to adjust its approach. By being an active collaborator, you will often get better results than if you let Claude go off and do its thing.

You have three tools to course-correct Claude:

1. Press the Escape key to interrupt Claude. You can interrupt Claude anytime, including while it's thinking, or during a tool call, or when Claude is proposing a file edit. Interrupting retains everything in context, so you can refer back to what Claude was doing, including rejected file edits, and either tell Claude to take a different approach, or to continue what it was doing and also do something else after.
2. Double-tap Escape to jump back in history. This is useful for cases where Claude went down the wrong path, and you want to adjust your earlier prompt to explore a different direction. Just edit the prompt and repeat until you get the result you're looking for.
3. Ask Claude to undo its changes. I'll sometimes do this followed by #2, to get Claude to pursue a different path.

Once in a while, Claude 1-shots what I had in mind and gets it perfect on the first try. But most of the time, I make use of these course-correction tools to guide Claude toward the right solution.

## d. Use `/clear` to keep context focused

When using Claude for long-running sessions, its context can fill up with conversation, file contents, and bash commands that are irrelevant to its current task. This can hurt performance, and occasionally distract and side track Claude. Use the `/clear` command often to zero out the context window when you're done with a task and ready to move on to the next task.

## e. Use checklists for complex workflows

For large tasks that have multiple steps or require exhaustive solutions -- common examples: code migrations, fixing all 100 lint errors, running complex build scripts -- you can significantly improve performance by having Claude use a Markdown file as a scratchpad.

For example, to fix a large number of lint issues, you can do the following:

1. Have Claude run the lint command, and write all the resulting errors (including filenames and line numbers) to a Markdown file, as a simple checklist
2. Then, instruct Claude to go down the checklist one by one, to fix and verify each issue and check it off before moving on to the next issue

# 3. Use headless mode to automate your infra

Claude Code ships with [headless mode](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview#automate-ci-and-infra-workflows), a powerful way to use Claude in non-interactive contexts like CI, pre-commit hooks, build scripts, and larger scripts.

Use the `-p` flag with a prompt to enable headless mode. You can also toggle on JSON output with `--json` and verbose output (to see a step by step transcript) with the `--verbose` flag.

## a. Use Claude to lint

## b. Use Claude for code review

# 4. Give Claude more tools

## a. Bash tools

## b. MCP

.mcp.json

## c. Local prompts

# 5. Multi-Claude

## a. Have one Claude write; use another Claude to verify

## b. Have multiple checkouts of your repo

## c. Use git worktrees

## d. Use sub-agents

## e. Use headless mode with a custom harness

---

How do you use Claude Code? Drop a comment on Hacker News or on Threads.

_For more in depth docs, see [claude.ai/code](https://claude.ai/code)._
