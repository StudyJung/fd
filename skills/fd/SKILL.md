---
name: fd
title: Fast Direct Command
description: /fd + option(-s:skills,-t:teams,-a:apis,-m:mcp,-w:wiki) + order + "args" + etcs.
tools: All
---

# RULE [P0 | ABSOLUTE | STRICT ORDER | NO SKIP | SILENT | PRODUCTION]
- P0: **EXECUTION**: run matched .md file once
- P1: **TEAMS(/fd -t)**: Ensure 100% inclusion of all comments and Do not reference previous reports
- P1: **OUTPUT**: no file log
- P1: **INPUT**: /fd <option> <order> ["args..."] [etcs...]
- P1: **PARSE**: `/fd -s& print "Hi Huns. test. " 1 23 456`
  (command: `/fd`, option: `-s`, task: `&`, order: `print`, "args": `"Hi Huns. test. "`, etcs:`1 23 456`)

## EXECUTE [P1 | ABSOLUTE | STRICT ORDER | NO SKIP | SILENT | PRODUCTION]
- P0: **XML**: must follow structure
- P0: **BASH**: must execute bash
- P0: **CALL**: call only when listed
- P0: **CURL**: no web fetch, only bash curl
- P1: **MATCH**: (Score >= 70: EXACT 100 | PREFIX 50+5n | PARTIAL 50+5n | FUZZY 80-5d ) → if score < 70: EXIT "No match found."
- P1: **EDGE**: args empty = null | case insensitive | tie = first wins

<execute name="fd">

  <const name="-s" value="skills" />
  <const name="-t" value="teams" />
  <const name="-a" value="apis" />
  <const name="-m" value="mcp" />
  <const name="-w" value="wiki" />
  
  <param name="command" required="true" position="1" value="/fd" />
  <param name="option" required="true" position="2" />
  <param name="order" required="true" position="3" />
  <param name="args" required="false" position="4" />
  <param name="etcs" required="false" position="5+" />

  BASH: `echo $command, $option, $order, $args, $etcs`

  IF option NOT IN [-s, -t, -a, -m, -w]:
    EXIT "Invalid option. Use -s, -t, -a, -m, -w."

  SWITCH option:
    CASE "-s": MATCH @.base/skills/{order}/SKILL.md → CALL BASH SKILL.md
    CASE "-t": MATCH @.base/teams/teammates_{excluding 's_' order}.json(Fallback:@.base/teams/teammates_base.json) → Agent TeamCreate → TaskCreate per stage(strict_order) → Execute {order}, {args}, {etcs}
    CASE "-a": MATCH @.base/apis/{order}/{args[0]}.md → CALL BASH CURL
	CASE "-m": MATCH Server from .mcp.json by {order} → CALL MCP tool: mcp__{server}__{order}({args}) → Do not repeatedly output request response messages
	CASE "-w": MATCH WIKI {order:default 'common'} Create, Ingest, Query, Lint
    DEFAULT: EXIT "Invalid option. Use -s, -t, -a, -m, -w."

</execute>
