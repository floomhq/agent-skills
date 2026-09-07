---
name: get-started-with-cyborg-mcp
description: Wire the public Skill catalog at cyborg.floom.dev into an agent using the cyborg-mcp MCP server, then find, read and install a first Skill. Invoke when somebody says "connect the catalog", "add cyborg-mcp", "set up the MCP", "install a Skill from cyborg.floom.dev", or has just landed on that site and does not know what to do first.
---

# Get started with cyborg-mcp

`cyborg-mcp` is an MCP server for the public Skill catalog at
<https://cyborg.floom.dev>. Once it is wired in, an agent can search that
catalog, read where a Skill came from and what it is licensed under, and write
one into its own skills directory. There is no account, no API key and no
sign-up: every route it reads is public.

This file is written for the agent doing the wiring. Most of it is one command.
The part that is not one command is the part below, and it is worth reading
before you promise anybody anything.

## First decide whether this is even possible for them

`cyborg-mcp` is a program that runs **on the reader's own machine**. Their agent
launches it with `npx` and talks to it over stdin and stdout. That splits every
possible reader in two, and the split decides the whole conversation:

- **The agent runs on their machine** — Claude Code, Codex CLI, Cursor,
  Windsurf, Claude Desktop, OpenCode, Gemini CLI. It can launch the server.
  Everything below applies.
- **The agent runs on somebody else's server and reaches out over HTTP** — the
  ChatGPT web app and its Connectors are the case people ask about. A remote
  connector needs a URL to connect to. This catalog publishes no MCP endpoint
  over HTTP; there is nothing to paste. Do not improvise one, and do not walk
  somebody through a Connectors dialog that cannot end in success. Send them to
  the website or to the CLI one-liner in *Without an MCP server at all* below,
  both of which give them the same Skills.

Say which of the two they are in before anything else. Getting this wrong costs
somebody twenty minutes in a settings panel that was never going to work.

## Node, and the one honest terminal moment

`npx` ships with Node. The server needs **Node 20 or newer**.

```bash
node -v
```

If that prints `v20` or higher, the rest is a single line. If it prints nothing,
or `command not found`, that is the moment a terminal is genuinely required and
there is no way around it: they install Node from <https://nodejs.org> and then
come back. Say that plainly. A GUI-only path does not exist, and pretending it
does just moves the failure later.

## Wire it in

Use the reader's own agent. Each of these adds a server named `cyborg`; the name
is yours to choose and only decides what the tools are prefixed with in some
clients.

### Claude Code

```bash
claude mcp add cyborg -- npx -y cyborg-mcp
claude mcp list
```

The second line health-checks it and should print
`cyborg: npx -y cyborg-mcp - ✔ Connected`.

### Codex CLI

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

`codex mcp list` shows the row as `enabled`. It does not start the server, so it
proves the config and not the connection; the check in the next section does
that.

### Anything that reads an `mcpServers` block

Cursor, Windsurf, Claude Desktop and Gemini CLI all take the same shape. Paste
this into the MCP config file the app documents:

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

Then **restart the app**. None of them re-reads that file while running, and a
config that is correct but unread looks exactly like a config that is wrong.

Gemini CLI has `gemini mcp add cyborg npx -y cyborg-mcp`, which writes that same
block into `.gemini/settings.json` (add `--scope user` for `~/.gemini`).

### OpenCode

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

`opencode mcp list` opens a connection to it and reports it as connected.

## Confirm it, and read the tool names off the server

Ask the agent to list its MCP tools. **Do not trust a tool name written down
anywhere, including here.** The four names are derived from the product name in
the server's own source, so a rename changes them, and `tools/list` on a live
connection is the only place that is ever current.

As published in `cyborg-mcp@0.1.3` they come back as:

| Tool | What it does |
| --- | --- |
| `cyborg_search_skills` | Search published Skills. Lexical ranking. |
| `cyborg_get_skill` | One Skill: source repository, exact commit, content hash, licence, declared permissions, rights basis. |
| `cyborg_get_stack` | One Stack: its curator, and the ordered Skills with the release each item pins. |
| `cyborg_install_skill` | Writes one Skill into an agent's skills directory on this machine. |

If nothing comes back, the server did not start. Go to *When it does not work*.

## Install the first Skill

1. **Search.** `cyborg_search_skills` with a plain query. The order is lexical
   relevance. It is not a quality ranking, and no install count or popularity
   signal exists in this catalog to rank by.
2. **Read it before you write it.** `cyborg_get_skill` returns the repository it
   was copied from, the exact commit, the hash of the bytes, the licence, and any
   tools the Skill's own header declares. A Skill is instructions your agent will
   follow, so read them the way you would read any other code you install.
3. **Install.** `cyborg_install_skill` takes `slug`, an optional `agent`
   (`claude`, `codex`, `cursor`, `gemini` or `opencode`) and an optional `scope`
   (`global`, the default, or `project`). `global` writes to that agent's home
   skills directory; `project` writes under the current working directory. It
   names the agent back to you in the result, and when more than one agent is on
   the machine it **refuses rather than guessing** — pass `agent` yourself.
4. **Reload.** Most agents read their skills directory at startup. Restart the
   session, or the Skill is on disk and invisible.

The folder it writes is named after the Skill's **title**, not its catalog slug,
so the two can differ: `/skills/workplan` lands in a directory called
`work-plan-skill`. The result tells you the exact path it used. Read that rather
than guessing where to look. (This Skill's own title and slug are deliberately
the same string, so it lands in `get-started-with-cyborg-mcp`.)

The write is deliberately boring: files are staged inside the skills directory
and moved into place with one atomic rename, any path that is absolute or
contains `..` is refused, a symlinked staging path is refused, and an existing
copy of the same Skill is backed up under
`.floom/backups/<folder>/<timestamp>/` before it is replaced. An interrupted
install does not leave half a Skill where an agent will read it.

## Without an MCP server at all

Every Skill page prints one command that does the same thing:

```bash
npx cyborg-skills@2.0.3 install https://cyborg.floom.dev/skills/<slug>
```

Same catalog, same bytes, no config file and no restart. This is the right
answer for anyone whose agent cannot launch a local server, and for anyone who
would rather not add one.

**`cyborg mcp` is a different server.** The `cyborg-skills` CLI has an `mcp`
subcommand, and it serves *your own signed-in workspace library* — it needs an
account and its tools are named `floom_*`. `cyborg-mcp`, the package this file is
about, serves the *public catalog* and needs no account. They are two doors to
two different things and the names are one character apart.

## What it sends, and how to send nothing

Two rows, and the tool result says which is which.

- Fetching an install link is recorded server-side. A fetch is not an install:
  a fetch can be followed by a failed write, and a crawler can fetch and install
  nothing.
- After the files are on disk, and only then, the server reports the install
  itself. It sends exactly three values: the Skill slug, which agent's directory
  was written to, and what the installer is (`cyborg-mcp/<version>`). No path,
  no home directory, no username, no hostname, no machine identifier, no file
  count, no timing.

Nothing on the catalog's side can observe a write to your disk, so that second
row is stored and displayed permanently as a self-reported claim and reaches no
public counter. To send nothing at all, set this in the server's environment:

```
CYBORG_NO_INSTALL_REPORT=1
```

It is read before any request is built, so setting it means no request is made
rather than one that is built and thrown away. Put it where your client passes
environment to the server:

```bash
claude mcp add cyborg -e CYBORG_NO_INSTALL_REPORT=1 -- npx -y cyborg-mcp
codex mcp add cyborg --env CYBORG_NO_INSTALL_REPORT=1 -- npx -y cyborg-mcp
```

In an `mcpServers` block it is an `"env": { "CYBORG_NO_INSTALL_REPORT": "1" }`
key beside `command`. Codex writes it as its own TOML table:

```toml
[mcp_servers.cyborg.env]
CYBORG_NO_INSTALL_REPORT = "1"
```

The same variable works for `cyborg-skills`.

That name is built from the product name at release time, exactly as the tool
names are, so a later release can print a different one. You do not have to
guess which: every install result carries an `opt_out` line naming the variable
the running server actually reads, and `cyborg --help` and `cyborg install
--help` print the CLI's. Read one of those rather than this paragraph.

## What this catalog does not claim

It is a private preview. Nothing in it holds a Tested, Evaluated, Signed,
Recommended or Verified Publisher state, and no tool here reports one, because
no evaluation has been run. There are no install counts, no popularity signal
and no reviews — not hidden, not yet built: absent. Listing a Skill is not an
endorsement of it, and it is not an endorsement of this catalog by whoever wrote
the Skill. Every Skill's page names who credited it to whom, and says whether
that credit came from the source repository's own metadata or is the publisher's
own statement.

Repeat that honestly when somebody asks whether a Skill is any good. The answer
is that nobody here has tested it, and the provenance is there so they can judge
for themselves.

## When it does not work

| What you see | What it is | What to do |
| --- | --- | --- |
| Client shows the server but no tools | The process did not start | Run `npx -y cyborg-mcp` in a terminal. It should sit there silently on stdio. Errors print to stderr. |
| `command not found: npx` | No Node | Install Node 20+ from nodejs.org. Nothing else fixes it. |
| Tool names are not the ones above | The published server was renamed | Read `tools/list`. That is why this file tells you not to trust the table. |
| Install refuses and asks which agent | More than one agent on the machine | Pass `agent`. It will not guess which one you meant. |
| A different catalog host is refused | `CYBORG_CATALOG_URL` is on an allowlist | Only the real catalog origin is accepted. An arbitrary host is refused rather than trusted because somebody typed it. |
| Skill is installed but the agent ignores it | It read its skills directory at startup | Restart the agent session. |

## What was actually run, and what was not

Written 2026-09-07, against `cyborg-mcp@0.1.3` and `cyborg-skills@2.0.3`.

**Run end to end, tool call included:** Claude Code 2.1.261 and Codex CLI
0.153.2. Both were configured with the command above, connected to the published
package over stdio, and returned a real result from `cyborg_search_skills`.

**Connection proven, no model-driven call:** OpenCode 1.18.25. `opencode mcp
list` completed the MCP handshake against the server and reported it connected.

**Config accepted, tools did not appear:** Gemini CLI 0.40.1. `gemini mcp add`
wrote the standard `mcpServers` block at both scopes, and in a headless run the
tools were not offered to the model. Treat Gemini as unconfirmed and check
`tools/list` yourself.

**Not tested at all:** Cursor, Windsurf and Claude Desktop. They are listed above
because they read the same `mcpServers` shape, which is a claim about the config
format and not a claim that anybody ran it.

**Known impossible:** the ChatGPT web app, for the reason in the first section.
