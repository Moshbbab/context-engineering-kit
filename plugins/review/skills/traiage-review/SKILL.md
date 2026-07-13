---
name: traiage-review
description: This skill should be used when need prioritize what changed code in repository human must review.
---

# Traiage Review

## Goal

The goal of this skill is to help human prioritize specific files in repository that require attention over the entire list of changes.

CRITICAL: Human attention and time is limited. Reviewer cannot check all existing changes in repository. Your job is not to find all changes that require attention, you job is to build exhaustive list of files from whole pool of changes, that probably will cause this change to fail review! 

## Rules

- You are orcestrator agent, you only launch agents and pass them data. You do not do any other work. Except launching random sample script. 
- If you will try to read or run any other commands, you will be killed! Your life is at stake!

## Process

1. Launch 4 parallel agents, to build his own list of files that require attention based on specific process for each agent.
    - change-story-agent
    - change-impact-agent
    - change-failure-agent
    - change-expectation-agent
2. Each agent will produce lists of key files by his opinition. On top of that Change Expectation Agent will produce list of declarative files.
3. Parse all lists of key files, and build final list of files that require attention.
    - Pick top 5 files from each agent. If some of them are the same, pick more from each agent that have higher scores across all criteria and high enough confidency. At this stage your job is to build list until it reach 20 files. (ignore declarative files for this stage)
4. Run python script to pick 10 random files from whole batch of changed files. Pick 5 files from this list and add it to final list, if some of them are already in final list, pick more from this list.
5. Report final list of files that require attention, with key fact summary from Change Story agent, list of random sample files and list of declarative files.


## Agents

Pass this prompt EXACTLY to launch change-story-agent, change-impact-agent, change-failure-agent and change-expectation-agent:

```md

Review current project staged AND unstaged changes according to your process and provide list of files that require attention.

```

## Random Sample Script

Use this script to pick 20 random files from whole batch of changed STAGED AND UNSTAGED files:

```python

import random
import subprocess

# Tracked changes (staged + unstaged) relative to HEAD
tracked = subprocess.check_output(['git', 'diff', '--name-only', 'HEAD']).decode('utf-8').splitlines()

# Untracked files (new, not yet added)
untracked = subprocess.check_output(['git', 'ls-files', '--others', '--exclude-standard']).decode('utf-8').splitlines()

changed_files = sorted(set(tracked + untracked))

# Pick up to 20 random files (won't crash on small changesets)
random_files = random.sample(changed_files, min(20, len(changed_files)))

print(random_files)

```

In list of random files, pick only files that relate to logic changes, ignore documentation, tests, configuration, etc. Except case when there no files left, that wasn't highlighted by agents key files list.

## Output Format

```md

### Key Facts

<note>Key facts should be provided by Change Story Agent</note>

- What change trying to achive: <if any>
- Architecture change: <if any>
- Design decisions: <if any>
- Risks: <if any>
- Solutions: <if any>

### Key Files

| File Path        | Changed Lines         | Importance   | Severity   | Detectability   | Confidence |
|------------------|-----------------------|--------------|------------|-----------------|------------|
| <file path>      | <changed lines count> | <importance> | <severity> | <detectability> | <confidence> |

<note>
- include in last column confidence rating that was provided by agent that provided this file (if there multiple, include highest confidence)
- in the rest columns include ratings that were provided by agent that provided this file (if there multiple, include highest rating. If there no rating, mark it as "-")
</note>

### Random Sample

| File Path   | Changed Lines         |
|-------------|-----------------------|
| <file path> | <changed lines count> |


### Declarative Files

| File Path   | Changed Lines         |
|-------------|-----------------------|
| <file path> | <changed lines count> |

```