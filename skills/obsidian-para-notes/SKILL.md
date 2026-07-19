---
name: obsidian-para-notes
description: Manage work notes in Obsidian using the PARA method (Projects, Areas, Resources, Archives). Automatically documents working sessions, discoveries, and context to a structured knowledge base. Use this skill whenever the user mentions "work notes", "PARA", documenting sessions, saving discoveries, or organizing knowledge in Obsidian. Also trigger when the user wants to capture learning, track project progress, or retrieve information from their work knowledge base.
compatibility:
  required_skills:
    - obsidian-cli
    - obsidian-markdown
---

# Obsidian PARA Work Notes

This skill manages a PARA-organized Obsidian vault for work documentation. It handles creating, updating, and retrieving work notes while maintaining a structured knowledge base.

## Vault Structure

The vault at `/home/obetsun/Documents/Work/obsidian/second_brain` uses this structure:

```
00_Inbox/                   Quick captures, raw data (organize later)
01_Projects/                Active projects with goals and lessons
02_Areas/                   Long-term responsibilities
03_Resources/               Knowledge base, code snippets, docs
04_Archive/                 Completed projects, old data
05_Templates/               Note templates
06_Daily_Logs/              Chronological daily logs
  ├── YYYY-MM-DD.md         Individual daily logs
  └── 06_Daily_Logs/MASTER-LESSONS-LEARNED.md   Index of all lessons across all projects
Home.md                     Mind map index with links to key nodes
```

**06_Daily_Logs/06_Daily_Logs/MASTER-LESSONS-LEARNED.md** is a critical file that serves as a searchable index of every mistake and lesson learned across all projects. This prevents repeating the same mistakes and makes cross-project knowledge easily discoverable.

## Core Workflows

### 1. Document a Working Session

When the user asks to document a working session or mentions they finished work on something:

1. **Identify the context:**
   - What project/area was worked on?
   - If unclear, ask: "Which project or area is this for?"
   - If it's a new project, create it first (see workflow 3)

2. **Create today's daily log** if it doesn't exist:
   ```bash
   obsidian create path="06_Daily_Logs/$(date +%Y-%m-%d).md" \
     content="---\ndate: $(date +%Y-%m-%d)\ntags: [daily-log]\n---\n\n# $(date +%Y-%m-%d)\n\n## Sessions\n\n" \
     silent
   ```

3. **Gather session information:**
   - Date/time: Use current timestamp
   - What was worked on: Project/area name
   - What was accomplished: Summary from conversation
   - What was learned: Key discoveries or insights
   - Next steps/blockers: From conversation context
   - Links: Relevant file paths, commands, or code mentioned

4. **Create session note in project:**
   ```bash
   obsidian create \
     path="01_Projects/{project-name}/Sessions/$(date +%Y-%m-%d)-session.md" \
     content="..." \
     silent
   ```
   
   Session note format:
   ```markdown
   ---
   date: YYYY-MM-DD
   time: HH:MM
   project: [[Project Name]]
   tags: [session, project-tag]
   ---
   
   # Session: {Brief Description}
   
   ## What was worked on
   {Project/area name and specific focus}
   
   ## Accomplishments
   - {What was completed}
   - {What progress was made}
   
   ## Discoveries & Learnings
   {Key insights, solutions found, gotchas discovered}
   
   ## Next Steps
   - [ ] {Action items}
   
   ## Blockers
   {Current obstacles, if any}
   
   ## Related
   - Files: `path/to/file.go:123`
   - Commands: `kubectl get pods -n namespace`
   - Links: [[Related Note]]
   ```

5. **Add session link to daily log:**
   ```bash
   obsidian append \
     path="06_Daily_Logs/$(date +%Y-%m-%d).md" \
     content="- [[01_Projects/{project-name}/Sessions/$(date +%Y-%m-%d)-session|{project-name}]]: {one-line summary}\n"
   ```

### 2. Save a Discovery or Learning

When the user mentions "save this discovery", "document this learning", or describes something they figured out:

1. **Identify what was discovered:**
   - Extract the key insight from conversation
   - Determine which project it relates to

2. **Add to project's Lessons Learned:**
   ```bash
   obsidian append \
     path="01_Projects/{project-name}/Lessons_Learned.md" \
     content="\n## {Discovery Title} - $(date +%Y-%m-%d)\n\n{Discovery details}\n\n**Context:** {What led to this discovery}\n\n**Impact:** {Why this matters}\n\n**Related:** [[Session link]] ^discovery-$(date +%Y%m%d)\n"
   ```

3. **Add to 06_Daily_Logs/MASTER-LESSONS-LEARNED.md:**
   Every significant discovery MUST be indexed here for cross-project discoverability:
   
   ```bash
   obsidian append \
     path="06_Daily_Logs/MASTER-LESSONS-LEARNED.md" \
     content="\n### {Discovery Title} - $(date +%Y-%m-%d)\n\n**Project:** [[01_Projects/{project-name}/README|{Project Name}]]\n**Category:** {category-tag} (e.g., kubernetes, security, debugging, ci-cd)\n\n{One-line summary of the lesson}\n\n**Details:** [[01_Projects/{project-name}/Lessons_Learned#^discovery-$(date +%Y%m%d)|Read full lesson]]\n\n---\n"
   ```
   
   The MASTER file acts as a searchable catalog. When someone searches for "kubectl" or "service account" issues in the future, this ensures they find ALL related lessons, not just those in the current project.

4. **Create or update Resource note (if applicable):**
   If the discovery is reusable knowledge (not project-specific):
   
   ```bash
   # Create resource note if it doesn't exist
   obsidian create \
     path="03_Resources/{topic-name}.md" \
     content="..." \
     silent
   ```
   
   Resource note format:
   ```markdown
   ---
   title: {Topic Name}
   tags: [resource, {category}]
   created: YYYY-MM-DD
   updated: YYYY-MM-DD
   ---
   
   # {Topic Name}
   
   {Brief description of what this resource covers}
   
   ## Key Points
   
   - {Important takeaway 1}
   - {Important takeaway 2}
   
   ## Examples
   
   {Code snippets, commands, or practical examples}
   
   ## Learned From
   
   - [[Project Name/Lessons_Learned#^discovery-id]]
   - [[Another Project/Lessons_Learned#^another-discovery]]
   
   ## Related Resources
   
   - [[Related Topic]]
   ```

5. **Link back in the Lessons Learned:**
   Add a reference to the Resource note in the project's Lessons_Learned.md

### 3. Create a New Project

When starting work on a new project:

1. **Create project directory structure:**
   ```bash
   mkdir -p "/home/obetsun/Documents/Work/obsidian/second_brain/01_Projects/{project-name}/Sessions"
   mkdir -p "/home/obetsun/Documents/Work/obsidian/second_brain/01_Projects/{project-name}/Resources"
   ```

2. **Create README.md:**
   ```bash
   obsidian create \
     path="01_Projects/{project-name}/README.md" \
     content="..." \
     silent
   ```
   
   README format:
   ```markdown
   ---
   title: {Project Name}
   status: active
   start_date: YYYY-MM-DD
   deadline: YYYY-MM-DD (if applicable)
   tags: [project, {relevant-tags}]
   area: [[Area Name]] (if applicable)
   ---
   
   # {Project Name}
   
   ## Goal
   
   {What is this project trying to achieve?}
   
   ## Context
   
   {Why is this project important? What's the background?}
   
   ## Timeline
   
   - Start: YYYY-MM-DD
   - Target completion: YYYY-MM-DD
   - Status: Active / On Hold / Completed
   
   ## Key Deliverables
   
   - [ ] {Deliverable 1}
   - [ ] {Deliverable 2}
   
   ## Sessions
   
   {Links to session notes will be added here automatically}
   
   ## Related
   
   - Area: [[Area Name]]
   - Resources: [[Resource 1]], [[Resource 2]]
   ```

3. **Create Lessons_Learned.md:**
   ```bash
   obsidian create \
     path="01_Projects/{project-name}/Lessons_Learned.md" \
     content="---\nproject: [[{project-name}/README]]\ntags: [lessons, {project-name}]\n---\n\n# Lessons Learned: {Project Name}\n\n{Discoveries and learnings will be documented here as the project progresses}\n" \
     silent
   ```

4. **Update Home.md** to add project link:
   ```bash
   obsidian append \
     path="Home.md" \
     content="  - [[01_Projects/{project-name}/README|{Project Name}]]\n"
   ```

### 4. Capture Quick Thought to Inbox

When the user wants to quickly save something without full organization:

```bash
obsidian create \
  path="00_Inbox/$(date +%Y%m%d-%H%M%S)-{brief-slug}.md" \
  content="---\ncaptured: $(date +%Y-%m-%d %H:%M:%S)\n---\n\n{content}\n" \
  silent
```

Later, these can be organized into proper projects/areas/resources.

### 5. Retrieve Information

When the user asks "what did I learn about X" or wants to recall context:

1. **Search the vault:**
   ```bash
   obsidian search query="{search-term}" limit=20
   ```

2. **Look in specific locations based on query type:**
   - **Project-specific:** Check `01_Projects/{project-name}/Lessons_Learned.md`
   - **General knowledge:** Check `03_Resources/`
   - **Timeline:** Check `06_Daily_Logs/` with date range
   - **Quick captures:** Check `00_Inbox/`

3. **Read relevant notes:**
   ```bash
   obsidian read file="{note-name}"
   ```

4. **Synthesize and present:** Summarize findings from the notes found

### 6. Archive a Project

When a project is completed:

1. **Update project status:**
   ```bash
   obsidian property:set \
     path="01_Projects/{project-name}/README.md" \
     name="status" \
     value="completed"
   
   obsidian property:set \
     path="01_Projects/{project-name}/README.md" \
     name="completion_date" \
     value="$(date +%Y-%m-%d)"
   ```

2. **Move to archive:**
   ```bash
   mv "/home/obetsun/Documents/Work/obsidian/second_brain/01_Projects/{project-name}" \
      "/home/obetsun/Documents/Work/obsidian/second_brain/04_Archive/{project-name}"
   ```

3. **Update Home.md:** Remove from active projects, add to archive section if it exists

## Standard Templates

The skill uses these templates for consistency:

### Daily Log Template
```markdown
---
date: YYYY-MM-DD
tags: [daily-log]
---

# YYYY-MM-DD

## Sessions

{Links to working sessions}

## Quick Notes

{Brief observations, thoughts}

## Tasks Completed

- {What got done today}

## Decisions Made

{Important decisions and their rationale}

## Problems Encountered

{Issues faced and how they were addressed}
```

### Project README Template
See workflow 3 above.

### Session Note Template
See workflow 1 above.

### Lessons Learned Template
```markdown
---
project: [[project-name/README]]
tags: [lessons, project-name]
---

# Lessons Learned: {Project Name}

{Discoveries and learnings documented chronologically as they happen}

## {Discovery Title} - YYYY-MM-DD

{Discovery details}

**Context:** {What led to this}
**Impact:** {Why it matters}
**Related:** [[Session link]]

^discovery-id
```

### Resource Note Template
See workflow 2 above.

## Important Guidelines

### When to Ask vs. Infer

**Ask the user when:**
- Which project a session belongs to (if ambiguous)
- Whether to create a new project vs. use existing one
- Project deadlines and goals (for new projects)
- Whether a discovery should go in Resources (general) or stay project-specific

**Infer from conversation:**
- Session accomplishments (from what was discussed)
- What was learned (from discoveries in the conversation)
- Next steps and blockers (from context)
- Relevant links (files, commands mentioned)

### Linking Strategy

- **Use wikilinks** for all internal vault references: `[[Note Name]]`
- **Use block references** for linking to specific discoveries: `[[Note#^block-id]]`
- **Cross-link liberally:** Sessions → Daily Log → Project → Lessons → Resources → MASTER-LESSONS-LEARNED
- **Maintain Home.md** as the central hub with links to all active projects and key areas

**Cross-linking pattern for graph view:**
- **Projects link to Resources:** When a project uses a resource or pattern
- **Daily logs link to projects:** Each session entry links to the project
- **Lessons link to Resources:** When creating/updating a resource from a lesson
- **MASTER-LESSONS-LEARNED links to Projects:** Every entry references its source project
- **Resources link back to Projects:** Show where knowledge came from

This creates a rich graph where you can visually trace knowledge flow: Daily work → Projects → Lessons → Resources, with MASTER-LESSONS-LEARNED as a central hub.

### Atomic Notes Principle

**One topic per note.** Each note should cover a single concept, discovery, or resource:

✅ **Good:** `03_Resources/Kubectl-Port-Forward-Tekton.md` covering just kubectl port-forwarding with Tekton
❌ **Bad:** `03_Resources/Kubernetes-Commands.md` covering 20 different kubectl patterns

**Why this matters:**
- Makes linking more precise (link to the exact concept, not a giant document)
- Easier to find and reuse specific knowledge
- Creates a better graph view (nodes represent clear concepts)
- Supports future refactoring (split/merge notes as knowledge evolves)

**When creating Resources:** If a discovery could be split into multiple independent tips, create separate notes and link them together under a parent topic note.

### Keep It Lightweight

The goal is to capture knowledge without creating overhead:

- **Sessions:** Brief summaries, not exhaustive transcripts
- **Daily logs:** Links + one-line summaries, not full duplicates
- **Lessons:** Key insights, not everything that happened
- **Resources:** Reusable patterns, not project-specific details

### Vault Targeting

Always specify the vault explicitly since the user might have multiple Obsidian vaults open:

```bash
obsidian vault="second_brain" {command}
```

### Error Handling

If Obsidian CLI commands fail:
1. Check if Obsidian is running: suggest `obsidian version` to test
2. Verify vault name with `obsidian vaults`
3. Fall back to direct file operations if Obsidian is closed

## Examples

### Example 1: Documenting a debugging session

User: "I just finished debugging that Konflux pipeline issue. Can you document this session?"

Response flow:
1. Identify it's about debugging Konflux pipelines (likely existing project or new one)
2. Ask if unclear: "Is this for an existing project, or should I create a new one?"
3. Create today's daily log if missing
4. Create session note with:
   - What was worked on: Konflux pipeline debugging
   - Accomplishments: Fixed ImagePullBackOff issue in build-container task
   - Learnings: Service account permissions were missing for registry access
   - Next steps: Document the fix in the pipeline README
   - Links: kubectl commands used, file paths touched
5. Link session in daily log
6. If significant discovery, ask: "This service account permission issue seems like something that could happen again. Should I add it to Resources as a troubleshooting tip?"

### Example 2: Quick capture

User: "Save this to my notes: kubeconfig contexts can be switched with kubens"

Response:
1. Create quick capture in Inbox with timestamp
2. Optionally suggest: "I've saved that to your Inbox. Would you like me to organize it into a Kubernetes resource note right away, or leave it for later?"

### Example 3: Retrieving past learnings

User: "What did I learn about Tekton pipeline debugging?"

Response:
1. Search vault for "Tekton pipeline debugging"
2. Check Projects with "tekton" or "pipeline" in name
3. Look in 03_Resources/ for Tekton-related notes
4. Read relevant Lessons_Learned.md files
5. Synthesize: "Based on your notes, here's what you've learned about Tekton pipeline debugging: ..."

## When This Skill Applies

Trigger this skill when the user:
- Says "document this session" or "save this to my notes"
- Mentions "work notes", "PARA", or "Obsidian knowledge base"
- Describes finishing work and wants to capture it
- Discovers something and wants to remember it
- Asks "what did I learn about X"
- Wants to create a new project
- Needs to organize quick captures from Inbox
- References their "second brain" or work documentation

## When This Skill Does NOT Apply

Don't use this skill for:
- General Obsidian usage unrelated to work notes (use obsidian-cli or obsidian-markdown instead)
- Creating personal notes, journals, or non-work content
- Plugin development or theme customization
- Obsidian configuration changes

This skill is specifically for managing work knowledge using the PARA method.
