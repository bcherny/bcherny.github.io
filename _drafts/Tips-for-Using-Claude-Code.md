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

# 2. Common workflows

Claude Code does not force you to use a specific workflow. This means flexibility -- you can use Code the way you want -- and it also means the community has experimented and come up with a number of workflows. Below is a short list of some of the most common ones.

## a. Explore, plan, code, commit

This is a workhorse workflow, useful for a variety of problems. It looks like this:

1. Start by asking Claude to read the relevant files, images, or URLs, providing either general pointers ("read the file that handles logging") or specific filenames ("read logging.py"). Tell Claude not to write any code just yet.
2. Then, ask Claude to make a plan for how to approach a specific problem. Optionally, use the word "think" to trigger extended thinking to more deeply trade off alternatives; use stronger language to encourage longer thinking: "think" < "think hard" < "megathink" < "think harder" < "ultrathink".
3. Ask Claude to implement its solution in code
4. Finally, ask Claude to commit the result

Without #1-#2, Claude will tend to jump to coding a solution. Sometimes, that's what you want. By asking it to research and plan ahead, you can squeeze significantly better performance out of Claude in cases where a problem needs deeper thinking upfront.

## a. Write tests, commit; code, iterate, commit

This is my favorite workflow for changes that are easily testable with unit, integration, or e2e tests. There has been a lot written about TDD, and in the age of agentic coding, TDD becomes an even more powerful tool. Here's what it looks like:

1. Ask Claude to write a few tests (eg. by giving it a set of expected input and output pairs), telling Claude not to run the tests yet because we're doing TDD.
2. When you're happy with the tests, ask Claude to commit them
3. Then, ask Claude to write code to make the tests pass. Instruct Claude not to edit the tests, and to keep going until the tests pass. It will usually take a few iterations for Claude to write code, run the tests, adjust the code, and run the tests again.
4. Once you're happy with the changes, commit again

## b. Write code, see result, iterate 3-4x

## b. Codebase Q&A

## d. Use Claude to interact with git

Write commit messages, revert files, etc.

## e. Use Claude to interact with GitHub

Make PRs, fix build errors, do code review, triage issues

## f. Use Claude to work with Jupyter notebooks

## g. Safe yolo mode

# 3. Tips for any workflow

## a. Be specific in your instructions

## a. Give Claude images

Paste in screenshots, diagrams, etc. Especially useful when combined with iteration.

## b. Give Claude URLs

## c. Course correct early and often

If you toggle auto-accept mode (shift+enter) and give Claude a prompt, it will pull in files, reason through the problem, and give you code back. The more effort you put into your prompt, the better result you'll get out. However, most of the time you should also nudge Claude along the way to adjust its approach. By being an active collaborator, you will often get better results than if you let Claude go off and do its thing.

You have three tools to course-correct Claude:

1. Press the Escape key to interrupt Claude. You can interrupt Claude anytime, including while it's thinking, or during a tool call, or when Claude is proposing a file edit. Interrupting retains everything in context, so you can refer back to what Claude was doing, including rejected file edits, and either tell Claude to take a different approach, or to continue what it was doing and also do something else after.
2. Double-tap Escape to jump back in history. This is useful for cases where Claude went down the wrong path, and you want to adjust your earlier prompt to explore a different direction. Just edit the prompt and repeat until you get the result you're looking for.
3. Ask Claude to undo its changes. I'll sometimes do this followed by #2, to get Claude to pursue a different path.

Once in a while, Claude 1-shots what I had in mind and gets it perfect on the first try. But most of the time, I make use of these course-correction tools to guide Claude toward the right solution.

## d. Use /clear to keep context focused

## e. Use checklists for complex workflows

Ask Claude to write a checklist for you, then follow it.

# 3. Use headless mode to automate your infra

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
