# Application Workflow

Use this workflow when saved Aclip knowledge can improve active work.

Agents may proactively consult Aclip during complex work such as:

- architecture or technical design
- research synthesis
- product or workflow planning
- implementation choices with meaningful tradeoffs
- debugging or postmortem analysis
- writing documents that should reuse prior reading

Do not consult Aclip for every small question. Avoid lookup for trivial commands, purely local syntax fixes, or tasks where saved knowledge is unlikely to change the answer.

## Proactive Lookup

When the active task may benefit from saved knowledge, infer a small set of topics, tags, or classes and query Aclip:

```bash
aclip list --topic <topic>
aclip list --tag <tag>
aclip list --class methodology
aclip list --class reference
aclip list --class case_study
aclip list --class tech_update
```

If candidates look relevant, inspect the strongest ones:

```bash
aclip show <id>
```

Use saved content as context for the current work. Do not treat the saved resource as automatically correct; compare it with the current codebase, user requirements, and present constraints.

## Reporting Use

When saved resources materially influence the answer, say so explicitly:

```text
Based on saved Aclip resources:
- <title or id>: <point applied>

Applied to this task:
1. <decision or recommendation>
2. <decision or recommendation>
```

If no relevant resources are found, state that briefly and continue from the available context.

## Boundaries

Do not invent saved knowledge. If Aclip does not return a relevant saved resource, do not claim one exists.

Do not use live URLs as a substitute for saved Aclip content in this workflow. If the user gives a new URL, follow capture and initial analysis first.

Do not let Aclip lookup replace normal engineering verification. For code changes, still inspect the repository and run appropriate tests.
