---
name: aporia-architecture-scan
description: >-
  Scans the architecture of a product from its repository and writes it to
  Aporia with aporia:write_architecture, for one environment: where it runs,
  its zones, what runs in each (compute, database, storage, network, search,
  stream), what runs outside the host, who talks to whom, and the choice behind
  a part when the repository explains it. A scan that changes something is
  kept as a new version, so the Architecture page shows what the repository
  says today and what changed since the last read. Use for
  /aporia-architecture-scan, scanning or drawing the architecture, refreshing
  the Architecture page after the infrastructure changed, or when the
  Architecture page is empty. For the first specifications of a product, use
  aporia-onboard.
---

# Aporia architecture scan

The Architecture page of Aporia shows the infrastructure of the product **as the repository says it is**: one diagram per environment, read by an agent, and a new version each time a read changes it. Nobody draws it by hand, so it never drifts for long. This skill is that read.

The page is also where a teammate learns **why a part is the way it is**: why one writer, why this host and not another. Aporia keeps no separate record of technical choices. A choice the repository explains is told on the card it shaped, with the file that explains it.

You reach Aporia through the **MCP server only**, pinned to one product. Call its tools fully qualified, as `aporia:*`. If the tools are not available, stop and ask the user to connect the Aporia MCP server.

**A question to the human goes through the tool your agent has for asking one** (`AskUserQuestion` in Claude Code): one decision per question, with the answers you expect as its options. A question written in the middle of a message gets lost. Without such a tool, or when nobody is there to answer (a scheduled run), the questions close your message, numbered, one decision each.

**`aporia:write_architecture` is not among your tools?** The Aporia server was added with `?tools=work`, the profile of a coding session, which leaves it out. Ask the human to add the server again without `?tools=work`, then start over.

## The bar

- **Evidence only.** Every node is something the repository shows: a deployment file, an infrastructure definition, a compose service. A service you assume exists is not in the scan.
- **A diagram, not an inventory.** A person reads it in a minute. Under twenty cards reads well. Group what belongs together.
- **Infrastructure, not features.** What a feature does lives in its specification. Here: what runs, where, and what it talks to.
- **The why is quoted, never guessed.** A `description` carries a choice or a limit only when a file states it. You do not explain an infrastructure from its shape.
- **No secret, ever.** You read the names of the services in an example environment file. You never read, copy or send a value.

## Workflow

```
Scan progress:
- [ ] Phase 0: load the context and the current scan (aporia:get_architecture)
- [ ] Phase 1: read what deploys and what runs
- [ ] Phase 2: draw zones, nodes and edges
- [ ] Phase 3: show the human, wait for their answer, then write (aporia:write_architecture)
```

### Phase 0: context

`aporia:get_context` once: the product's name and vision tell you what you are looking at. Keep the `sessionId` the first write returns, for the session.

`aporia:get_architecture { environment }` reads the latest version of the environment you scan: its `version`, zones, nodes and edges, or `scan: null` when no scan has run. It is not evidence, the repository is. You read it for its ids.

### Phase 1: read

Read what deploys and what runs, not what the code imports. In this order:

1. the deployment workflows and pipelines (CI files, deploy scripts);
2. the infrastructure as code (Terraform, Pulumi, CDK, serverless and platform config files);
3. the container and compose files, the process files, the scheduled jobs;
4. the example environment file, for the services the product talks to (names only);
5. the architecture documents: the README section that says where the product runs, the infrastructure README, the decision records. They hold the why. For what exists, the code wins: a document can be older than the infrastructure it describes.

Note for each thing you find **the file that shows it**: it becomes the `doc` of its node.

**Read the branch the environment deploys from**: the one a deployment workflow names, or the default branch when none does. Production is what was deployed, not what you are working on. Record what you read in `branch`: the branch, or the tag when production deploys from tags. When you stand on a work branch, say so and scan the deploying branch instead, or tell the human what your branch changes.

**What the repository cannot settle is a question, not a choice of yours:**

- it does not say where the product runs: ask the human for the `host`;
- two files disagree (two hosting configurations, two regions): ask which one is live;
- it says the product is not released yet: ask before writing a `production` scan;
- a service is named only in the example environment file: keep it when the code calls it, with the file that calls it as its `doc`. Drop it otherwise.

**Only a file or the human settles a question.** One that gets no answer stays open: silence is not a yes, and you do not answer in the human's place. `host` is required, so without one from a file or from the human nothing is written. A placeholder ("Not stated") is not a host. The same holds while which configuration is live, or whether to scan `production`, is open. A detail the human answers that they do not know stays out of the diagram, or is told as unsettled in the `description` of the node it concerns.

**Nothing deploys** (a library, a script run by hand): say so and write nothing. The Architecture page is optional.

### Phase 2: draw

Read `references/shared/content-style.md` for the shape of every string.

**`host`** is where it runs, in the team's words: "AWS eu-west-3", "Vercel and Convex".

**Zones** are the frames. Each has an `id`, a `label` and `offHost`: `false` for the host itself, `true` for what runs elsewhere (users, third parties), drawn dashed. One host zone is the usual case, with the host as its label. Do not cut a host into layers (edge, application, data): the columns already show them, and each extra zone spreads the diagram. A product that runs on two platforms takes one host zone for each. An optional `note` is one line under the zone.

**Nodes** are what runs, each in its zone. A node is something that deploys or is provisioned on its own, not a folder of the code: a platform that runs your functions, your scheduled jobs and your HTTP routes together is one node. Only `id`, `zoneId` and `title` are required:

| Field | What it holds |
|---|---|
| `id`, `zoneId` | a short stable string of your own (`api`, `db`), and the zone it sits in |
| `title` | the name the team uses: "API", "Job queue". Sixteen characters at most: the card cuts a longer one |
| `category` | `compute` runs code, `database` keeps records, `storage` holds files (a bucket, a static site, a CDN in front of one), `network` routes traffic (a load balancer, a gateway), `search` indexes, `stream` moves events (a queue, a log pipe). Leave it out for a neutral node: a browser, a third party |
| `summary` | the second line of the card, two or three words: "7 pools", "One writer" |
| `description` | shown when the card is opened: what it does, then the choice or the limit a teammate must know, when a file states it |
| `stack` | what deploys it, as the repository names it: the infrastructure stack or module (`backend`, `data`). Without one, its runtime ("Node 24, Hono") |
| `doc` | the file that shows it or explains it: a path in the repository, or a URL |

**Keep the ids.** A zone or a node that is still there keeps the `id` it has in the current scan, even when its title changed. Aporia follows a part from one version to the next by its id, and an edge by its two ends: a new id for the same part reads as one part removed and another added.

**Edges** are who talks to whom: `from` the caller, `to` the called, and a `label` of one to three words when it adds something ("HTTPS", "SQL", "webhook").

**The order you send is the layout.** The scan carries no coordinates:

- Zones read left to right. The off-host zones listed before the first host zone stack on the left, the ones listed after the last host zone stack on the right. So list who comes in first (users, the systems that call the product), then the host, then what the host calls.
- In a host zone, a node sits one column to the right of the node that calls it, and right of the furthest one when several call it. Only the edges inside the zone place its nodes. An edge drawn backwards puts a database before its API.
- In an off-host zone, the nodes stack top to bottom in the order you send them.
- **The page shows five columns before it scrolls sideways**: one for who comes in, three for the host, one for what it calls. In the host, each step of a chain of calls takes a column. Past three, keep the edges of the main path (the way in, what computes, where the data lives) and tell the rest of the chain in the `description` of the node it starts from.

**Keep the arrows few:**

- The third parties the host calls take one line in the `note` of their zone ("Called by the API and the workers"), not an arrow each. Draw an arrow to one only when it says what the note cannot: a pipe with a name.
- A third party that calls back (a webhook) is told in its `summary` or in the note, not drawn: an arrow from the right-hand zone back into the host crosses the whole diagram.
- Those who come in by the same door are one node: a browser and a desktop shell that open the same app.
- Arrows that reach the same card pile their labels up. Label one of them at most.
- Several things of one kind are one node, with the count in its `summary`.

Stay inside the bounds: 12 zones, 60 nodes, 120 edges. Past them the call is refused, and the right answer is to group, not to split the scan.

### Phase 3: show, then write

Show the human the diagram in text before writing: the zones in order, the nodes of each, the edges, and every `description` that states a choice with the file it comes from. Ask what is missing or wrong. They know what the repository does not say. What they add on their word alone carries no `doc`.

**Write only after the human answers.** With no answer, or with a question of Phase 1 still open, make no call: the page would show a version nobody read. Close instead by saying that nothing was written to Aporia, listing what the repository shows as what you read, and leaving each open question for the human to answer.

Once they agree, one call:

```jsonc
// aporia:write_architecture input
{
  "environment": "production",
  "host": "Demo cloud, Paris",
  "branch": "main",
  "zones": [
    { "id": "users", "label": "Users", "offHost": true },
    { "id": "app", "label": "Demo cloud, Paris", "offHost": false },
    { "id": "third", "label": "Third parties", "offHost": true, "note": "Called by the API" }
  ],
  "nodes": [
    { "id": "browser", "zoneId": "users", "title": "Browser", "summary": "Buyers, team" },
    { "id": "api", "zoneId": "app", "category": "compute", "title": "API", "summary": "HTTP and jobs", "stack": "backend", "doc": "infra/api.tf" },
    {
      "id": "db",
      "zoneId": "app",
      "category": "database",
      "title": "Database",
      "summary": "One writer",
      "description": "Orders and accounts. One writer by design. The API never reads a replica.",
      "stack": "data",
      "doc": "docs/architecture/database.md"
    },
    { "id": "mail", "zoneId": "third", "title": "Email provider", "summary": "Order emails" }
  ],
  "edges": [
    { "from": "browser", "to": "api", "label": "HTTPS" },
    { "from": "api", "to": "db", "label": "SQL" }
  ]
}
```

A node names a zone that exists and an edge names nodes that exist, or the call is refused.

**The scan is the whole picture of its `environment`, never a patch**: what you leave out reads as removed. Scan `production` first. Scan `staging` only when the repository shows it and it differs: same ids, same picture, and only what differs changed.

The answer says what the scan did:

```jsonc
// aporia:write_architecture answer
{
  "environment": "production",
  "version": 3,
  "status": "new",
  "nodes": 5,
  "changes": {
    "host": false,
    "zones": { "added": [], "removed": [], "changed": [] },
    "nodes": { "added": ["queue"], "removed": [], "changed": ["api"] },
    "edges": { "added": [{ "from": "api", "to": "queue" }], "removed": [], "changed": [] }
  }
}
```

`status` is `new` when the scan was kept as a version, `unchanged` when nothing differs from the latest one. Then no new version is created and no Activity event either. The latest version is dated again, with your branch, your reader and your layout order. `changes` compares with the previous version, and is `null` for the first scan.

Close by telling the human the version written and what changed since the previous one, from `changes`, with the titles of the parts and not their ids: "V3: the queue was added, the API changed." Then where to see it: the Architecture page of the product (`product.url` from Phase 0 gives the address of the product).

## When to run it again

After a change to the infrastructure: a new service, a database moved, a provider replaced. Not after every pull request: a change to a feature does not move the architecture.

## Acceptance checklist

- [ ] Every node of the host cites, in `doc`, the file that shows it, or was confirmed by the human.
- [ ] Every choice told in a `description` is stated by a file, not deduced.
- [ ] The diagram reads in a minute: grouped, not exhaustive, with few arrows.
- [ ] The zones are sent in reading order: who comes in, the host, what the host calls.
- [ ] The host takes three columns at most, and every title fits its card.
- [ ] No value of any secret was read or sent.
- [ ] The human saw the diagram and answered before it was written, and the `host` comes from a file or from them.
- [ ] The scan is whole, and every part that did not change kept its id.
- [ ] The human was told the version written and what changed since the previous one.

## Anti-patterns (reject)

A node for a service the code only imports a client for, with no sign it is deployed. One node per file. Features drawn as nodes. A reason invented for a choice the repository does not explain. An arrow from every node to every third party. A webhook drawn as an arrow back across the diagram. A chain of six calls drawn as six columns. A patch sent as a scan, which reads as everything else removed. A new id for a part that did not change. A scan of a work branch written as production. A scan written when nobody answered. A placeholder or a guess sent as the `host`. One node per folder of the backend. A secret pasted into a `description`.
