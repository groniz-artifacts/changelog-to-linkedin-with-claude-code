# How to Turn a Changelog into LinkedIn Posts with Claude Code

Claude Code can read the context in a repository, so it can turn a structured changelog into LinkedIn drafts without making you paste every entry into a prompt. But "summarize `CHANGELOG.md` and post" is too loose an instruction. First separate public facts from internal notes. Then choose one reader problem for each post and ask Claude Code for a draft plus a fact map. A maintainer can add the lesson behind the change and approve the exact copy.

After copy approval, Groniz handles the LinkedIn connection, platform-specific formatting, scheduling, and delivery. Groniz is an agent-driven publishing connector for 32+ networks and supports both LinkedIn profiles and Pages. Claude Code can use it through a Skill, the native CLI, or MCP. You can draft and prepare delivery in one working session, but publishing still requires a separate, explicit action.

## How the changelog moves through review

```text
CHANGELOG.md + linked documentation
  → eligible update list
  → one audience angle per update
  → LinkedIn draft and fact map
  → human revision and approval
  → runtime settings discovery
  → Groniz schedule request
  → delivery verification and native measurement
```

If the changelog contains several updates with distinct reader outcomes, this workflow can produce a short series. Most patches still do not need their own post.

## Prerequisites

You will need:

- Claude Code with access to the repository and its guidance files
- a reviewed `CHANGELOG.md` and links to public documentation
- a Groniz account with a connected LinkedIn profile or Page
- one supported Groniz route: Skill, CLI, or MCP
- approved image files, if needed
- a named reviewer who can authorize both the copy and the publication details

For the broader agent setup, see [automating social publishing with Claude Code](https://groniz.com/blog/automate-social-media-posting-with-claude-code). For channel-level review principles, see [LinkedIn posting with AI agents](https://groniz.com/blog/linkedin-posting-with-ai-agents).

The Skill teaches Claude Code the Groniz workflow and installs the CLI behind it:

```bash
npx skills add groniz/groniz-cli
```

To work directly with the native CLI:

```bash
curl -fsSL https://groniz.com/install.sh | sh
groniz auth:login
```

You can also connect Claude Code to the remote MCP server with a Groniz API key:

```bash
claude mcp add --transport http groniz https://mcp.groniz.com/mcp \
  --header "Authorization: Bearer YOUR_API_KEY"
```

The examples below use the CLI because you can inspect its final payload before execution. Claude Code can run the same commands when the Skill is installed. With MCP, ask Claude Code to inspect the tools and schemas exposed by the live server, then use the same source, review, and approval gates. Pick one route for the workflow so reviewers know which permissions and confirmation prompts they will see.

## Why a changelog is useful but insufficient

A changelog is good evidence of what changed, but it often says little about why the change matters. Entries such as "added CSV export" or "fixed webhook retries" usually leave out the user's situation, the trade-off, and the implementation lesson. A useful LinkedIn post needs at least one of those missing details.

[LinkedIn recommends](https://www.linkedin.com/help/linkedin/answer/a1517763) giving AI specific input, treating its output as a first draft, and reviewing that draft with your own expertise and personal knowledge. For changelog work, Claude Code can extract and organize the material. The maintainer supplies the reason, consequence, and firsthand detail.

Publish a feature list only when the list itself helps the audience. One update might support a product explanation, an engineering lesson, or a customer workflow note. Each angle needs different evidence.

## Prepare a public-safe source packet

Set an explicit source boundary for Claude Code. Repository access lets the agent follow references across files, but not every file belongs in public copy. An `AGENTS.md` file can make the rules persistent, including public-source-only drafting, required fact maps, and a mandatory pause before publication.

Create a packet like this:

```markdown
# LinkedIn changelog packet

Release: v2.4.0
Public date: 2026-07-18
Audience: API customers who manage failed jobs

Eligible entries:
- Added retry-status visibility in the dashboard.
- Added a documented endpoint for retry history.

Reader problem:
- Operators previously had to correlate separate logs.

Maintainer note:
- We learned that showing the latest error without the retry sequence hid the useful story.

Public sources:
- CHANGELOG.md#240
- docs/retry-history.md

Exclude:
- Internal incident IDs
- Customer names
- Unreleased queue changes
```

If an entry has no clear reader problem or public proof, leave it in the changelog. It does not need to become a post.

## Reusable Claude Code prompt

```text
Read CHANGELOG.md and the public files listed in linkedin-changelog-packet.md.
Do not use other repository files as factual sources.

Select at most three changelog entries that teach distinct lessons. For each:
1. Name the audience and its problem.
2. Choose one angle: product workflow, engineering lesson, or operator lesson.
3. Draft one LinkedIn post with a concrete opening, verified context, the
   maintainer's stated insight, and an understated close.
4. Return a table mapping every factual claim to a source line or heading.
5. Mark any missing evidence [VERIFY].

Avoid release-note dumping, invented customer reactions, inflated claims, and
generic engagement questions. Do not schedule or publish. Save drafts under
content/linkedin/ for human review.
```

This prompt makes Claude Code select before it writes. A changelog with twelve entries may have only two worth posting. The other ten can stay in the documentation.

## Turn release facts into a LinkedIn-native post

Check each draft by asking four questions:

1. Who encountered the problem?
2. What has changed, and where is that change verified?
3. What did the team learn or choose?
4. What should the reader understand, try, or discuss after reading?

The third question needs a human answer. Claude Code can surface a maintainer note, but the maintainer should rewrite it to describe the decision they actually made. Cut internal vocabulary and retain only the technical detail that supports the lesson. A specific constraint or trade-off usually says more than a broad claim about the release.

LinkedIn supports profiles and Pages through Groniz. Confirm whether the post speaks as an individual or an organization before editing pronouns, attribution, and call to action.

## Use separate copy and publication approvals

Before any write action, check:

- Every claim maps to the changelog packet or approved public documentation.
- The release is public and the date is correct.
- The author's lesson is written or explicitly confirmed by that author.
- No internal issue, customer, security, or roadmap detail appears.
- The final copy fits the selected profile or Page voice.
- The link and media are approved.
- The exact integration, date, time, and timezone are visible.
- A reviewer approved the words before separately approving the schedule.

You can easily undo a draft saved to the repository. You cannot as easily take back a post sent to a public account, so require publication permission at the write action.

## Discover LinkedIn settings and schedule through Groniz

Authenticate and resolve the live connection immediately before delivery:

```bash
groniz whoami
groniz integrations:list
groniz integrations:settings LINKEDIN_INTEGRATION_ID
```

The settings response defines the required fields, current length limit, and available integration tools. Use a dynamic tool only when the live response exposes it. Capabilities vary by provider and connection kind. A payload copied from another channel, or even another LinkedIn connection, is not a reliable template.

For approved media, upload first:

```bash
groniz upload ./content/linkedin/approved-v240.png
```

Copy only the returned `.path` into the scheduling request. Then give Claude Code a delivery instruction with clear limits:

```text
Using the live settings for LINKEDIN_INTEGRATION_ID, construct a schedule
request for the approved post file and 2026-08-07T14:30:00Z. If media is
present, use only its uploaded .path. Display the resolved payload, target,
and time. Wait for a separate "approve publication" message before calling
the write tool.
```

Once the request is resolved and publication is approved, the corresponding CLI shape is:

```bash
groniz posts:create \
  -c "$(cat ./content/linkedin/approved-v240.txt)" \
  -m "<returned-groniz-media-.path>" \
  -s "2026-08-07T14:30:00Z" \
  -t schedule \
  -i "LINKEDIN_INTEGRATION_ID" \
  --settings '<required-settings-json>'
```

Omit `-m` when the approved post has no media. Otherwise, replace it with only the `.path` returned by `groniz upload`. Replace the integration ID and settings placeholder with live values, and supply every required setting. Show the fully resolved command before asking for publication approval. This article does not claim a live end-to-end test for LinkedIn, so follow the live schema if it differs from an example here.

## Verify delivery and recover safely

Record the returned post ID, target integration, scheduled time, status, and platform URL when one is available. Then inspect the scheduled post in Groniz:

```bash
groniz posts:list \
  --startDate "2026-08-07T00:00:00Z" \
  --endDate "2026-08-08T00:00:00Z"
```

If the request times out or returns an ambiguous result, check for an existing post before retrying. Retrying without that check can create a duplicate.

When a request fails:

- refresh authentication after an identity or permission error
- rerun integration discovery when the destination is unclear
- reload settings after a schema or length error
- upload rejected media again and use the new returned `.path`
- revise the post instead of truncating it mechanically
- return to publication review if the copy, media, destination, or time changes

## Measure the series on LinkedIn

LinkedIn's native analytics vary by surface. Individual post analytics can include impressions, members reached, reactions, comments, reposts, saves, sends, and link visits. Page reporting has its own content metrics. Use LinkedIn's definitions for [individual post analytics](https://www.linkedin.com/help/linkedin/answer/a516971/post-analytics-for-your-content) and [Page content analytics](https://www.linkedin.com/help/linkedin/answer/a564051). Alongside the relevant metrics, record the angle, opening, source entry, author edits, and result window you chose.

Ask whether the framing helped the intended reader and what deserves more attention in the next source packet. "Did the changelog post go viral?" tells you little about the quality of the source or the lesson. Engineering lessons and product workflows have different goals, so note that difference before comparing their results.

When the review process is ready, use the [Groniz Claude Code setup](https://groniz.com/agents/claude-code) to connect Claude Code to the delivery layer.
