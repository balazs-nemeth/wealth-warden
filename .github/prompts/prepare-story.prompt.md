---
name: prepare-story
description: Use this prompt to prepare a story for development.
---

# Workflow

1. Run the `ux-reviewer` agent to ensure the story meets UX standards and all strato components are correctly documented and the UX is approved.
   Use the `runSubagent` tool to run the `ux-reviewer` agent on the story.
2. Run the `architect` agent to ensure the story aligns with the architecture and design principles outlined in AGENTS.md and the project structure. Use the `runSubagent` tool to run the `architect` agent on the story.
3. Report any issues found during the UX review or architecture check back to the user for resolution. If the story passes both checks, mark it as "Approved" and inform the user that the story is ready for development.
