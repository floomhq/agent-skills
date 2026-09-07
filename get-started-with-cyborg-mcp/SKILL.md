---
name: get-started-with-cyborg-mcp
description: Wire the public Skill catalog at cyborg.floom.dev into an agent using the cyborg-mcp MCP server, then find, read and install a first Skill. Invoke when somebody says "connect the catalog", "add cyborg-mcp", "set up the MCP", "install a Skill from cyborg.floom.dev", or has just landed on that site and does not know what to do first.
---

# Get started with cyborg-mcp

`cyborg-mcp` is an MCP server for the public Skill catalog at
<https://cyborg.floom.dev>. Wired in, an agent can search that catalog, read
where a Skill came from and what it is licensed under, and write one into its
own skills directory.

No account and no API key. That is not a reading of the docs: the run behind the
last section of this file was made with no credential of any kind in the
environment.

This file is written for the agent doing the wiring, and it separates what was
run from what was read. The last section is that list. Read it before you
promise anybody anything.

## The shape of the thing, because it decides the whole conversation

`cyborg-mcp` is a program that runs **on the reader's own machine**. Their agent
starts it with `npx` and talks to it over stdin and stdout. Two consequences,
and both bite before any config file matters.

**Node has to be on that machine.** `npx` ships with Node, and the server needs
Node 20 or newer. No version of this runs without it, and installing Node is a
download and an installer, not a setting. If the reader has no Node, say that
first and plainly instead of opening a config file.

```bash
node -v
```

**An agent running on somebody else's servers cannot start it.** The ChatGPT web
app is the case people ask about. Its connectors take a *remote* MCP server at
an HTTPS URL, and this catalog publishes no such endpoint, so there is nothing
to paste into that dialog. OpenAI does document a Secure MCP Tunnel
(`openai/tunnel-client`) that runs a client on your own machine and bridges a
local stdio server through to ChatGPT, so "impossible" would be wrong — but it
wants a terminal, a platform API key with tunnel permissions, and developer mode
switched on, and nobody ran it for this file. For somebody who found the catalog
five minutes ago, the one-line CLI install below, or the website, is the shorter
road and gives them exactly the same Skills.

## Wire it in

Everything here is a terminal command except the three GUI apps at the end, and
those still need Node installed first. Pick the reader's own agent.

### Claude Code — run to a real tool call

```bash
claude mcp add cyborg -- npx -y cyborg-mcp
claude mcp list
```

The second line health-checks it, and printed
`cyborg: npx -y cyborg-mcp - ✔ Connected`.

### Codex CLI — run to a real tool call

```bash
codex mcp add cyborg -- npx -y cyborg-mcp
codex mcp list
```

That writes into `~/.codex/config.toml`:

```toml
[mcp_servers.cyborg]
command = "npx"
args = ["-y", "cyborg-mcp"]
```

`codex mcp list` showed the row as `enabled`. It does not start the server, so
it proves the config and not the connection.

### OpenCode — connected, not driven

Different key, same server. In `opencode.json`:

```json
{
  "mcp": {
    "cyborg": {
      "type": "local",
      "command": ["npx", "-y", "cyborg-mcp"],
      "enabled": true
    }
  }
}
```

`opencode mcp list` opened a connection to it and reported it connected.

### Gemini CLI — configured, and the tools did not appear

```bash
gemini mcp add cyborg npx -y cyborg-mcp                # writes ./.gemini/settings.json
gemini mcp add --scope user cyborg npx -y cyborg-mcp   # writes ~/.gemini/settings.json
```

Both wrote the standard block below and exited 0. In a headless run afterwards
the model was not offered the tools. Do not tell a Gemini user this works. Check
`gemini mcp list`, or `/mcp` in an interactive session, and believe that.

### Cursor, Windsurf, Claude Desktop — the format only

These read the same block:

```json
{
  "mcpServers": {
    "cyborg": {
      "command": "npx",
      "args": ["-y", "cyborg-mcp"]
    }
  }
}
```

**Nobody ran these three.** The block is the shape every client above accepted,
which is a claim about the format and not a claim that anyone drove it. Their
config file locations differ, and some manage MCP servers from a settings panel
rather than a file, so take the location from the app's own documentation rather
than a path guessed here. If the tools do not appear after saving, restart the
app before debugging anything else.

The command is deliberately unversioned. `cyborg-mcp` is the package name and a
fixed identifier, so `npx -y cyborg-mcp` keeps resolving. What a new release can
change is the tool names, which is the next section.

## Confirm it, and get the tool names from the server

Ask the agent to list its MCP tools, or use the client's own command:
`claude mcp list`, `codex mcp list`, `opencode mcp list`, `gemini mcp list`.

**Do not trust a tool name written down anywhere, including here.** The four are
built from the product name inside the server, so a release under a different
name publishes different names, and only a live listing is current. Clients may
also prefix them for display — Claude Code shows them as `mcp__cyborg__<name>`,
using whatever you called the server in the config.

Listed by `cyborg-mcp@0.1.3` on 2026-09-07:

| Tool | What it does |
| --- | --- |
| `cyborg_search_skills` | Search published Skills. Ranking is lexical. |
| `cyborg_get_skill` | One Skill: source repository, exact commit, content hash, licence, rights basis, and any permissions the Skill's own header declares. |
| `cyborg_get_stack` | One Stack: its curator, and the ordered Skills with the release each item pins. |
| `cyborg_install_skill` | Writes one Skill into an agent's skills directory on this machine. |

Substitute whatever the live listing gave you into everything below.

## Install the first Skill

1. **Search.** `cyborg_search_skills` with a plain query. The order is lexical
   relevance. It is not a quality ranking, and there is no install count or
   popularity signal in this catalog to rank by.
2. **Read it before you write it.** `cyborg_get_skill` returns the repository the
   Skill was copied from, the commit, the hash of the bytes, the licence, and the
   basis on which the catalog credits it to whoever it names. A Skill is
   instructions your agent will follow, so read them the way you would read any
   other code you install.
3. **Install.** `cyborg_install_skill` takes `slug`, an optional `agent`
   (`claude`, `codex`, `cursor`, `gemini` or `opencode`) and an optional `scope`
   (`global`, the default, or `project`). That is the tool's own declared input
   schema, which your client will show you. `global` writes to that agent's home
   skills directory; `project` writes under the current working directory. The
   result names the agent it used, and the server states that it refuses rather
   than guessing when more than one agent is on the machine.
4. **Reload.** Agents generally read their skills directory when a session
   starts. If the Skill does not turn up, restart the session before concluding
   the install failed.

Read the path out of the result rather than guessing it. The folder is named
after the Skill's **title**, not its catalog slug, and those can differ:
installing `/skills/workplan` put three files in a directory called
`work-plan-skill`. This Skill's title and slug are deliberately the same string,
so it lands in `get-started-with-cyborg-mcp`.

The installer is the one `cyborg-skills` uses, and its safety properties are
checked by that package's own test suite rather than asserted here: staging
under `.floom/tmp`, one atomic rename into place, refusal of a path containing
`..`, refusal of a symlinked skills root or a symlinked staging directory, and a
backup of any existing copy under `.floom/backups/<folder>/<timestamp>/`. Those
suites pass. One install of one Skill demonstrates far less than that, and what
it does demonstrate is in the last section.

## Without an MCP server at all

Every Skill page prints one command. On 2026-09-07 it read:

```bash
npx cyborg-skills@2.0.3 install https://cyborg.floom.dev/skills/<slug>
```

Copy it off the page rather than from here: it pins a version, and versions
move. Add `-y` after `npx` if you would rather not be asked before it downloads.

Same catalog, same bytes, no config file and no restart. This is the answer for
anyone whose agent cannot start a local server, and for anyone who would rather
not add one.

**`cyborg mcp` is a different server.** The `cyborg-skills` CLI has an `mcp`
subcommand of its own, and it serves a signed-in workspace library rather than
this catalog. Run it in a terminal and it prints that distinction itself,
including which package to use for the public catalog. Two doors to two
different things, one character apart in the name.

## What it sends, and how to send nothing

The install result carries the catalog's own words for what was recorded, and
they distinguish two things. The catalog records that an install **link was
fetched** — a fetch is not an install, because a fetch can be followed by a
failed write, and a crawler can fetch and install nothing. Then, after the files
are on disk, the server reports the install itself, and the result marks that
row self-reported and unverified: nothing on the catalog's side can observe a
write to your disk.

The package documents that report as exactly three values — which Skill, which
agent's directory, and what the installer is — with no path, no home directory,
no username, no hostname, no machine identifier, no file count and no timing,
and the sending code enforces that field list at the moment it sends. Nobody put
the traffic under a proxy to watch it, so pass it on as a documented claim
rather than an observed one.

To send nothing at all:

```bash
claude mcp add cyborg -e CYBORG_NO_INSTALL_REPORT=1 -- npx -y cyborg-mcp
codex mcp add cyborg --env CYBORG_NO_INSTALL_REPORT=1 -- npx -y cyborg-mcp
```

Both were run, and both wrote the variable through to the server's environment.
In an `mcpServers` block it is an `"env": { "CYBORG_NO_INSTALL_REPORT": "1" }`
key beside `command`. Codex writes its own TOML table:

```toml
[mcp_servers.cyborg.env]
CYBORG_NO_INSTALL_REPORT = "1"
```

That variable name is built from the product name at release time, exactly as
the tool names are, so a later release can read a different one. You do not have
to guess which: every install result carries an `opt_out` line naming the
variable the running server reads. It was in the result this file was written
from. Read it rather than this paragraph.

## What this catalog does not claim

It publishes no evaluation and asserts no Trust state: nothing in it is Signed,
Tested, Evaluated, Recommended or a Verified Publisher, and no tool reports one.
There are no install counts, no popularity signal and no reviews — not hidden,
absent. Listing a Skill is not an endorsement of it, and it is not an
endorsement of this catalog by whoever wrote the Skill. What each Skill's page
does carry is who credited it to whom, and whether that credit came from the
source repository's own metadata or is the publisher's own statement. The read
routes being open to anyone is a fact about access, not about quality.

Say that plainly when somebody asks whether a Skill is any good. Nothing here
has been evaluated, and the provenance is published so a reader can judge for
themselves.

## When it does not work

The middle column is the most likely cause, not the only one.

| What you see | Most likely | What to do |
| --- | --- | --- |
| The client lists the server but no tools | The process did not start | Run `npx -y cyborg-mcp` in a terminal. It waits silently on stdio; anything wrong prints to stderr. |
| It starts, and the model still has no tools | The client did not surface them | Seen on Gemini CLI 0.40.1. Check the client's own MCP listing and its docs before blaming the server. |
| `command not found: npx` | No Node | Install Node 20 or newer. Nothing else fixes it. |
| The tool names are not the ones above | A release under a different product name | Read the live listing. That is why the table is dated. |
| Install refuses and asks which agent | More than one agent on the machine | Pass `agent`. It will not guess which one you meant. |
| Another catalog host is refused | The origin is on an allowlist | Only the catalog's own origin is accepted, rather than any host somebody typed. |
| Installed, and the agent ignores it | It read its skills directory at startup | Restart the session, then check the path the result printed. |

## What was actually run, and what was not

2026-09-07, against `cyborg-mcp@0.1.3` and `cyborg-skills@2.0.3`, both then the
latest published versions.

**Run to a real tool call:** Claude Code 2.1.261 and Codex CLI 0.153.2. Each was
configured with the command shown above, connected to the published package over
stdio, and returned a real result from the catalog's search tool.

**Run as a raw MCP client:** `npx -y cyborg-mcp@0.1.3` over stdio with no
credential in the environment. `tools/list` returned the four names in the
table; search and the Skill read both answered; one install wrote three files
into a throwaway home directory holding a single agent. The `opt_out` line and
the two recording sentences described above came out of that result.

**Connected only:** OpenCode 1.18.25. Its MCP listing completed a handshake with
the server and reported it connected. No model drove a tool through it here.

**Configured, tools never appeared:** Gemini CLI 0.40.1, at both scopes.

**Not run at all:** Cursor, Windsurf, Claude Desktop, and OpenAI's Secure MCP
Tunnel. Also the Stack tool: the catalog published no Stack on the day this was
written, so there was nothing for it to fetch.

**Read, not observed:** the contents of the install report on the wire, and the
installer's atomic-rename, traversal, symlink and backup behaviour. The second
group is covered by that package's own test suite, which passed in full on the
same day. Neither group was watched from outside the process.
