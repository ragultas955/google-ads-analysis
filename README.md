# Google Ads Monthly Report Skill

An agent skill that builds a dashboard-style, 8-slide monthly Google Ads performance deck (PPTX) comparing the reporting month with the previous month. It uses a deterministic Python builder, so the output is the same in Claude, Claude Code, ChatGPT, Codex and Gemini.

## What you get
Overall KPIs, campaign table, search term charts and top 20 table (brand terms excluded, competitors kept), age group pies and table, and location charts and table, each with key insights and month-over-month deltas.

## Folder layout
```
google-ads-monthly-report/
  SKILL.md                    agent instructions
  scripts/build_report.py     deterministic deck builder
  references/data-schema.md   CSV format the builder reads
  assets/sample_data/         synthetic example data
  requirements.txt
```

## Try it without any agent
```
pip install -r requirements.txt
python scripts/build_report.py --data-dir assets/sample_data --client "Example Brand" \
  --month "August 2026" --prev-month "July 2026" \
  --brand-terms "example brand,exampleretail" --competitors "rival one,rival two" \
  --currency "Rs " --out sample_report.pptx
```

## Install by tool
Skill folder locations differ by tool and version, so check each tool's current docs if a path below does not work.

- **Claude Code:** copy the folder to `~/.claude/skills/` (personal) or `.claude/skills/` (project).
- **Claude (claude.ai / app):** zip the folder and upload it under Settings > Capabilities > Skills.
- **Codex:** copy the folder into the tool's skills directory (for example `~/.codex/skills/`), or reference `SKILL.md` from your `AGENTS.md`.
- **Gemini CLI:** copy the folder into the Gemini skills directory (for example `~/.gemini/skills/`), or reference `SKILL.md` from your `GEMINI.md`.
- **ChatGPT:** upload the folder (or the zip) to a chat or custom GPT with Code Interpreter enabled, and say "Use the google-ads-monthly-report skill".

## Prompt example
> Use the google-ads-monthly-report skill. Client: <Client>. Reporting month: August 2026, compare with July 2026. Data: <attached exports or connected account>. Brand terms to exclude: <brand, variants>. Competitor terms: <optional>.

## Notes
Sample data is synthetic. Rendering is done by the PPTX viewer you open the file in; fonts are set to Arial for consistency.
