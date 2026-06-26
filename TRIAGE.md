Triage every issue in the report at {report_path} against the source code at {source_path}.
Load each of the 3 available skills one at a time. For each skill, independently triage every issue from scratch — no context sharing between skills or between issues. Each issue gets a fresh, independent evaluation.
For each issue × skill combination:
- Apply the skill's gates in order against the source code
- If any gate fails, record the verdict per that skill's rules and stop
- If all pass, assign severity per that skill's rules
Output each verdict to:
TRIAGE/SKILL-1/{issue-id}.md
TRIAGE/SKILL-2/{issue-id}.md
TRIAGE/SKILL-3/{issue-id}.md
Format:
# {issue-id}: {title}

**Verdict**: {Critical/High/Medium/Low/Informational/Invalid}

**Why?** {2-5 sentences grounded in the code, naming which gates passed/failed}

**Caveats**: {empty if none}
Create all directories and files. When done, show a summary table of all issues × all 3 skills.
