# The Centaur's Guides

This repo contains "Centaur Guides," which are my attemptto create a collection of living documents
that help solve various technical problems. Some are my own work, while others were generated in
dialogue with AI. 

For example, the very first guide is "reliable-ai-editing.md", which OpenAI
largely wrote after a dialogue with it on how to make in-place edits more reliable and less
noisy (fewer non-semantic changes, so diffs would be clearer). 

The intro to this README, in contrast, was started by ChatGPT, but had to be largely rewritten by
me, because while LLMs are great text predictors, they are not mindreaders.

Many centaurs are written in Markdown so they can live alongside code, be read on GitHub, and be
easily adapted to project instructions or AI chat workflows. I plan on adding more centaurs in
the future, so stay tuned!

## Reliable AI Editing 
[`reliable-ai-editing.md`](reliable-ai-editing.md) is a Centaur Guide focused on safe,
predictable AI-assisted code updates. It addresses a specific failure mode of conversational
AI tools: generating plausible but incorrect code edits when authority boundaries are unclear.

The guide introduces:
 - Authority gating (“no code → no edits”)
 - Editor contracts for explicit AI behavior constraints
 - A two-phase workflow (design → apply)
 - Editing variants (minimal diff vs. refactor)
 - One-line safety guards
 - End-to-end example workflows
 - Common failure modes and anti-patterns

It is intended for developers, researchers, and tooling authors who rely on AI for real code 
maintenance and want correctness, reviewability, and trust rather than convenience alone.

