# /fd + option(-s:skill,-t:teams,-m:mcp,-w:web,+&^%$:agent) + order + "args" + etcs

# RULES [P0 | ABSOLUTE | STRICT ORDER | NO SKIP | SILENT | PRODUCTION]
P0: **EXECUTION**: run matched .md file once
P1: **TEAMS(/fd -t)**: Ensure 100% inclusion of all comments and Do not reference previous reports
P1: **OUTPUT**: no file log
P1: **INPUT**: /fd <option> <order> ["args..."] [etcs...]
P1: **PARSE**: `/fd -s& print "Hi Huns~ test. " 1 23 456`
  (command: `/fd`, option: `-s`, task: `&`, order: `print`, "args": `"Hi Huns~ test. "`, etcs:`1 23 456`)

## EXECUTE [P1 | ABSOLUTE | STRICT ORDER | NO SKIP | SILENT | PRODUCTION]
P0: **XML**: must follow structure
P0: **BASH**: must execute bash
P0: **CALL**: call only when listed
P0: **CURL**: no web fetch, only bash curl
P1: **MATCH**: (Score >= 70: EXACT 100 | PREFIX 50+5n | PARTIAL 50+5n | FUZZY 80-5d ) → if score < 70: EXIT "No match found."
P1: **EDGE**: args empty = null | case insensitive | tie = first wins

<execute name="fd">

  <const name="-s" value="skills" />
  <const name="-t" value="teams" />
  <const name="-m" value="mcp" />
  <const name="-w" value="webs" />

  <param name="command" required="true" position="1" value="/fd" />
  <param name="option" required="true" position="2" />
  <param name="order" required="true" position="3" />
  <param name="args" required="false" position="4" />
  <param name="etcs" required="false" position="5+" />

  BASH: `echo $command, $option, $order, $args, $etcs`

  IF option NOT IN [-s*, -t*, -m*, -w*,]:
    EXIT "Invalid option. Use -s, -t, -m, or -w."

  IF $option == "-*&":
    RUN background_agent(run_in_background: true, subagent_type: "general-purpose")
  ELSEIF $option == "-*^":
    RUN background_agent(run_in_background: true, subagent_type: "Plan")
  ELSEIF $option == "-*%":
    RUN background_agent(run_in_background: true, subagent_type: "Explore")
  ELSEIF $option == "-*$":
    RUN background_agent(run_in_background: true, subagent_type: "claude-code-guide")
  ELSE:
    RUN immediately

  SWITCH option:
    CASE "-s*": MATCH @.claude/skills/{order}/SKILL.md → CALL BASH SKILL.md
    CASE "-t*": MATCH @.claude/teams/config/config.json → Agent TeamCreate → Read teammates_{order or etcs[0]}.json(Fallback:@.claude/teams/teammates_base.json) → TaskCreate per stage(strict_order) → Execute {order}, {args}, {etcs}
    CASE "-m*": MATCH Server from .mcp.json by {order} → CALL MCP tool: mcp__{server}__{order}({args}) → NO additional text output
    CASE "-w*": MATCH @.claude/webs/{order}/{args[0]}.md → CALL BASH CURL
    DEFAULT: EXIT "Invalid option. Use -s, -t, -m, or -w."

</execute>
