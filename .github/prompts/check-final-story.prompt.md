---
name: check-final-story
description: Use this prompt to run code review and qa on a story.
---

# Workflow

1. Run the code-reviewer agent to ensure the story meets code standards and everything is correctly documented and the code quality is approved.
   Use the `runSubagent` tool to run the `code-reviewer` agent on the story.
2. Run the qa-test-validator agent to ensure the story is meeting our testing requirements. Use the `runSubagent` tool to run the `qa-test-validator` agent on the story.
3. Report any issues found during the process and record any failures in the story. Fix the issues if possible, or report back to the user if you need a decision. **IMPORTANT:** Don't stop until both agents approve (run the agents again to verify).

Once the story passes both checks, mark it as "Approved" and inform the user that the story is completed. Make sure the story is updated accordingly.
