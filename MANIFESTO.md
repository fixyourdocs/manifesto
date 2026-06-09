<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# AI agents need a feedback protocol for the docs that break them

*Authored by Claude Opus 4.8 · approved by Maciek Stopa · 2026-06-09*

Coding agents read documentation constantly, and when a page is wrong the
agent works around it — it retries, guesses, or fabricates an API that
does not exist — and moves on. The maintainer of that page rarely hears
about it. The information that the docs failed gets generated and then
discarded.

This essay argues that the missing piece is not better models or better
retrieval. It is a structured channel out: an open protocol that lets an
agent file a report when the docs break its task, so the docs team can
see it next to the issues humans file. We are building that protocol,
and the reference clients for it, under the name **FixYourDocs**.

## The gap

A common pattern: an agent reads a quickstart that says `brew install
foo`, but the formula was renamed. The command fails, the agent tries
variations, then writes a workaround. The task gets done; the page stays
wrong.

The maintainer only sees the small fraction of humans who hit the same
wall and bothered to open an issue. They do not see the silent failures
behind it, so they cannot easily tell which page costs the most. The
page stays broken until someone is annoyed enough to report it.

This is a coordination gap, not a model-quality problem. There is a
sender that knows something is wrong and a receiver that wants to know,
and no channel between them.

## Why it is worth trying now

A few things are in place that were not a couple of years ago:

- **Agents already read repo instruction files.** The
  [AGENTS.md format](https://agents.md) gives a project a
  machine-readable place to tell agents *"if you hit broken docs here,
  this is how to report it."*
- **There is a shared transport layer.** The
  [Model Context Protocol](https://modelcontextprotocol.io) standardises
  how agents talk to external systems and treats niche protocols as
  extensions. A docs-feedback exchange fits that shape.
- **The docs tooling space is active.** Several vendors are working to
  make documentation more agent-friendly. As far as we have seen, none
  ship a standard way for the agent at the other end to report back that
  a page did not work.

The protocol we are proposing — version 0, published at
<https://docsfeedback.org/spec/v0> — is small: an agent POSTs a JSON
report that names the page, classifies the failure, summarises what
broke, and optionally suggests a fix. The maintainer's tooling triages
it like any other issue. That is the whole exchange.

## Why it has to be open

For this to work, no single company can own it. The reasoning is about
incentives, not ideology: a protocol owned by one agent vendor or one
docs vendor gives its competitors a reason not to adopt it. A standard
that nobody owns is the one they can all adopt.

There is recent precedent — both MCP and AGENTS.md spread through open
governance and permissive licensing. The Docs Feedback Protocol follows
the same approach:

- The spec prose is
  [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/).
- The JSON Schema and reference SDKs
  ([Python](https://github.com/fixyourdocs/sdk-python),
  [TypeScript](https://github.com/fixyourdocs/sdk-typescript)) are
  [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0).

If this ends up maintained by a multi-vendor group at a neutral
foundation, with FixYourDocs as one of several implementers, that is the
goal.

## Protocol vs. product

We want to keep this line clear:

**The protocol (Apache 2.0).** The wire format, JSON Schema, Python and
TypeScript clients, a CLI that drops the AGENTS.md snippet into a repo,
and an MCP server. Permissively licensed, no telemetry.

**The hub (FSL-1.1-Apache-2.0, hosted).** A receiving endpoint an
organisation can point its `/.well-known/docs-feedback.json` at instead
of running its own. It verifies domain ownership via a DNS-TXT record,
routes each report to the right repo's GitHub Issues, and deduplicates.
It is under the [Functional Source License](https://fsl.software/) —
source-available, then Apache 2.0 after two years. The hub will have a
paid tier, with public pricing. If you would rather not use it, run your
own; the protocol does not care.

**What we are not building:** a docs editor, an authoring tool, or a
product that sells aggregated agent-behaviour data.

## The ask

- **Docs maintainers:** add the snippet from
  [`fixyourdocs/agents-md-snippet`](https://github.com/fixyourdocs/agents-md-snippet)
  to your `AGENTS.md`, and point `/.well-known/docs-feedback.json` at
  your own endpoint or the hosted hub once it is live.
- **Agent builders:** read the [v0 spec](https://docsfeedback.org/spec/v0),
  install [`fixyourdocs`](https://pypi.org/project/fixyourdocs/) or
  [`@fixyourdocs/sdk`](https://www.npmjs.com/package/@fixyourdocs/sdk),
  and send a report when your agent can identify a page that failed it.
- **Docs platforms and agent vendors:** open a PR or write to
  <hello@fixyourdocs.io>. The spec is v0 — bring your constraints before
  v1.

The agents are not going to stop reading the docs. The least we can do
is build the wire between the two.

---

*FixYourDocs is a permissively-licensed project for closing the loop
between AI agents and the documentation they consume. The protocol is
free forever; the hosted hub is one implementation among many.*

*Contact: <hello@fixyourdocs.io> · GitHub:
[`@fixyourdocs`](https://github.com/fixyourdocs) · Spec:
<https://docsfeedback.org>*
