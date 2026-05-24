<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# AI agents need a feedback protocol for the docs they break

*by Maciek Stopa, FixYourDocs · 2026-05-24*

Every coding agent breaks against bad documentation, every day. The agent
retries, falls back to guessing, fabricates a function that does not exist,
or quietly fails. The maintainer of those docs hears about it from zero of
those agents. The signal exists. The signal is real. The signal is being
discarded at industrial scale.

This essay argues that the missing piece is not better models, better
retrieval, or better prompts. The missing piece is a structured channel
out — an open protocol that lets an agent file a report when the docs
break its task, and lets the docs team see it next to the issues filed by
humans. I am building that protocol, and the reference clients for it,
under the project name **FixYourDocs**. This is the why-now and the
what-it-is.

## 1. The break

Sit next to anyone using a modern coding agent for a week and you will
watch the same scene play out. The agent reads a quickstart. The
quickstart tells it to run `brew install foo`. The formula was renamed
to `fooctl` in version 1.4. The agent's command fails. The agent guesses.
It tries `brew install foo-cli`, then `brew install foo@latest`, then
gives up and writes a shell snippet that works around the missing binary
entirely. The task gets done — clumsily — and the agent moves on. The
docs page stays exactly as wrong as it was.

This happens in the dozens of times per day per agent. Multiply that by
the population of agents running right now — Cursor, Claude Code,
GitHub Copilot, OpenAI's Codex, Gemini CLI, plus everything built on the
[Model Context Protocol](https://modelcontextprotocol.io) — and the
number of "docs failed me here" events crosses, conservatively,
hundreds of thousands a day. None of those events reach the maintainer.

The maintainer's view is the inverse. They see GitHub issues from the
small fraction of humans who hit the same wall and bothered to write it
up. They do not see the thousand silent failures behind each issue.
They cannot prioritise what they cannot measure. So the broken page
stays broken until a human gets angry enough to file the issue
themselves.

This is a coordination failure, not a model-quality problem. We have a
sender (the agent) that knows something is wrong. We have a receiver
(the maintainer) that wants to know. There is no channel between them.

## 2. Why now

The conditions for a fix did not exist eighteen months ago. They do now.

**Agents already read repo instruction files.** The
[AGENTS.md format](https://agents.md) — proposed jointly by OpenAI, Sourcegraph,
Google, Cursor, Factory, and others — gives any project a
machine-readable home for "here is how to work in this codebase." Every
major agent reads it. That means there is a place to put one extra line
that says *"if the docs are wrong, here is how to tell me."*

**There is a shared transport layer.** The
[Model Context Protocol](https://modelcontextprotocol.io) standardises
how agents talk to external systems. It is explicit that "extensions,
not core spec" is the right place for niche protocols
([MCP roadmap](https://modelcontextprotocol.io/development/roadmap)).
A docs-feedback exchange is exactly that shape: a small, well-defined
extension that ships alongside the core, addressable by any MCP client
without changing MCP itself.

**The docs industry is already trying.** [Mintlify](https://mintlify.com),
[Fern](https://buildwithfern.com), [Context7](https://context7.com),
[kapa.ai](https://kapa.ai), and the chat-bot vendors built into every
modern docs site are all racing to make documentation more
agent-friendly. They are converging on the same conclusion from
different directions: the docs page is no longer the final destination,
it is an intermediate node in an agent's workflow. None of them, today,
ship a way for the agent at the other end of that workflow to say
*"this didn't work."* That is the missing primitive.

**The pain is concentrated.** The same handful of pages get re-read by
every agent that hits them — the install page, the auth page, the
versioning page, the migration page. Fixing one bad page repairs
thousands of future agent runs. The cost-to-benefit ratio of a docs
fix has never been more lopsided in the maintainer's favour. They just
need to know which page.

The protocol I am proposing — version 0 is published at
<https://docsfeedback.org/spec/v0> — is the smallest thing that closes
this loop. An agent POSTs a JSON report to a documented endpoint. The
report names the page, classifies the failure kind, summarises what
broke, optionally suggests a fix, and stops. The maintainer's tooling
picks it up and triages it like any other issue. That's the whole
exchange.

## 3. What "open protocol" buys

The only way this works is if no single company owns it. The reason is
not ideology, it is incentives.

If Anthropic owned the protocol, neither Cursor nor OpenAI would adopt
it. If Cursor owned the protocol, neither Anthropic nor Google would
adopt it. If a single docs vendor — Mintlify, Fern, GitBook — owned
the protocol, the other vendors would build a competing one and
fragment the surface for every maintainer. Every individual company's
rational move is to refuse a competitor's standard. The only standard
that all of them can adopt without losing face is one that nobody owns.

There is recent precedent for this working. The MCP spec was published
by Anthropic and adopted by Cursor, OpenAI, Google, and the broader
ecosystem within a year, because the spec was governed openly and the
code was Apache-licensed. AGENTS.md was published as a multi-vendor
artifact from day one and went from announcement to broad adoption in
months. The pattern is the same: small spec, permissive licence, a
maintainer organisation that visibly does not favour one client over
another.

The Docs Feedback Protocol is published under the same terms:

- The spec prose is licensed [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/).
  Copy it, fork it, embed it in your own documentation. Attribute the
  source.
- The JSON Schema and the reference SDKs
  ([Python](https://github.com/fixyourdocs/sdk-python),
  [TypeScript](https://github.com/fixyourdocs/sdk-typescript)) are
  [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0), with the
  explicit patent grant. There is no contributor-only escape hatch.
- The CLA is the standard Apache Individual CLA. Contributions are
  collected via the
  [CLA Assistant](https://github.com/contributor-assistant/github-action)
  bot — the same mechanism many Linux Foundation and Apache projects
  use.
- The project is registered as the entity "FixYourDocs"; the wordmark
  exists so we can stop someone from publishing a malicious
  "FixYourDocs" client, not so we can charge for the name. The
  trademark policy will live in the org profile and explicitly permits
  *"compatible with the Docs Feedback Protocol"* and *"implements the
  Docs Feedback Protocol"* phrasing.

If a year from now this protocol is being maintained by a multi-vendor
working group at a neutral foundation — the
[Linux Foundation](https://www.linuxfoundation.org/) or
[AAIF](https://aaif.io/) — and FixYourDocs the company is one of
several reference implementers, that is the success state. The
protocol succeeds *because* it is not the company's moat.

## 4. What we are building, and what we are not

The line between the protocol and the product matters. I want to draw
it explicitly because the muddier this line, the less anyone trusts
the protocol.

**The protocol (Apache 2.0, free forever).** The wire format. The JSON
Schema. The Python and TypeScript reference clients. A small CLI
(`npx fixyourdocs init`) that drops the AGENTS.md snippet into a repo.
An MCP server so any MCP-aware agent can file reports without bespoke
integration. These are public, permissively licensed, and will remain
so. They have no telemetry, no phone-home, no "free tier" of a paid
service hiding inside.

**The hub (FSL-1.1-Apache-2.0, commercial SaaS).** A receiving endpoint
that organisations can point their `/.well-known/docs-feedback.json`
discovery file at, instead of standing up their own. It collects
reports, groups them by page, deduplicates, and produces a digest the
docs team actually reads. It is licensed under the
[Functional Source License](https://fsl.software/) — source-available,
non-compete for two years, then automatically Apache 2.0. Sentry
[adopted FSL](https://blog.sentry.io/introducing-the-functional-source-license-freedom-without-free-riding/)
for the same reasons I am: the model is honest about the
"commercially-supported open core" pattern, and the two-year sunset is
a credible commitment that the code becomes a community asset on a
predictable clock.

The hub will have a paid tier, and pricing will be public when it
ships. If you do not want the hosted hub, run your own — the protocol
does not care; that is the entire point.

**Things we are explicitly not building.** A new editor. A new docs
authoring tool. A "platform" of vendor lock-in. A telemetry product
that sells aggregated agent-behaviour data to anyone. A walled-garden
where reports filed via a competitor's client get second-class
treatment.

The constraint I am holding myself to is that any feature that would
make the protocol worse-for-everyone-else but better-for-us must be
declined — even if it would shorten our path to revenue.

## 5. The ask

If you are a docs maintainer:

- Add a snippet to your repo's `AGENTS.md` telling agents how to file
  reports against your docs. The snippet is one paragraph; the
  reference text lives at
  [`fixyourdocs/agents-md-snippet`](https://github.com/fixyourdocs/agents-md-snippet).
- Point your `/.well-known/docs-feedback.json` at either your own
  receiving endpoint or the hosted hub at `hub.fixyourdocs.io` once it
  is live. Pick whichever you prefer; either path uses the same
  protocol.
- If something about the spec rubs you the wrong way, file an issue at
  [`fixyourdocs/protocol`](https://github.com/fixyourdocs/protocol).
  The spec is v0 for a reason — every constraint is a candidate for
  revision before v1.

If you are building an agent:

- Read the v0 spec at <https://docsfeedback.org/spec/v0>. It is short.
- Install
  [`fixyourdocs`](https://pypi.org/project/fixyourdocs/) (Python) or
  [`@fixyourdocs/sdk`](https://www.npmjs.com/package/@fixyourdocs/sdk)
  (TypeScript). Both ship a typed `Client.send(report)` and nothing
  else.
- When your agent fails against docs and you can identify the page,
  send a report. The receiving organisation will thank you, eventually.

If you are building a docs platform or an agent vendor:

- Open a PR. Write to <hello@fixyourdocs.io>. The fastest way to make
  the protocol better is to bring the constraints your platform
  actually has, before v1 freezes them out.

If you want to support the work:

- Star the [protocol repo](https://github.com/fixyourdocs/protocol).
- Sponsor on [GitHub Sponsors](https://github.com/sponsors/fixyourdocs)
  once that is live; the sponsor tiers will exist to fund maintainer
  time, not to gate features.
- Talk about the project. The single biggest accelerant for an open
  protocol is the number of people who reference it by name.

The docs are not going to fix themselves, and the agents are not going
to stop reading them. The least we can do is build the wire between
the two.

---

*FixYourDocs is a permissively-licensed project for closing the loop
between AI agents and the documentation they consume. The protocol is
free forever; the hosted hub is one implementation among many.*

*Contact: <hello@fixyourdocs.io> · GitHub:
[`@fixyourdocs`](https://github.com/fixyourdocs) · Spec:
<https://docsfeedback.org>*
