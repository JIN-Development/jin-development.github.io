# Tutorial: Parallel Development with Claude Code Sub-Agents and Git Worktrees

## Overview

This tutorial demonstrates a powerful workflow for evaluating multiple implementation approaches in parallel using Claude Code sub-agents combined with git worktrees. This approach enables you to explore different solutions simultaneously, compare them side-by-side, and choose the best one - all without conflicts or switching branches.

**What you'll learn:**
- How to use git worktrees for parallel development
- How to launch multiple Claude Code sub-agents simultaneously
- How to structure tasks for parallel execution
- How to evaluate and merge the best approach
- Best practices for managing multiple worktrees

**When to use this workflow:**
- Evaluating multiple architectural approaches
- Comparing different implementation strategies
- Prototyping alternative solutions
- A/B testing different designs
- Learning by implementing the same feature multiple ways

**Prerequisites:**
- Git repository (local or cloned)
- Claude Code installed and configured
- Basic understanding of git branches
- Familiarity with Claude Code agents

---

## Table of Contents

1. [Understanding the Components](#understanding-the-components)
2. [When to Use This Workflow](#when-to-use-this-workflow)
3. [Step-by-Step: Parallel Development Workflow](#step-by-step-parallel-development-workflow)
4. [Real-World Example](#real-world-example)
5. [Best Practices](#best-practices)
6. [Troubleshooting](#troubleshooting)
7. [Advanced Techniques](#advanced-techniques)
8. [Cleanup and Maintenance](#cleanup-and-maintenance)

---

## Understanding the Components

### What Are Git Worktrees?

Git worktrees allow you to have multiple working directories from the same repository, each with a different branch checked out simultaneously.

**Traditional Git Workflow:**
```
your-project/               # Can only work on one branch at a time
├── .git/
├── src/
└── ...
```

**With Worktrees:**
```
your-project/              # Main worktree (main branch)
├── .git/
├── src/
└── ...

your-project-approach1/    # Worktree 1 (approach-1 branch)
├── src/
└── ...

your-project-approach2/    # Worktree 2 (approach-2 branch)
├── src/
└── ...
```

**Key Benefits:**
- ✅ Multiple branches checked out simultaneously
- ✅ No branch switching required
- ✅ Isolated working directories prevent conflicts
- ✅ Shared `.git` repository (efficient)
- ✅ Easy side-by-side comparison

### What Are Claude Code Sub-Agents?

Claude Code sub-agents are autonomous agents you can launch to perform specific tasks independently. They can work in parallel without interfering with each other.

**Key Features:**
- Execute tasks autonomously
- Work in specified directories
- Return results when complete
- Can run multiple agents simultaneously
- Isolated from each other

### The Synergy: Worktrees + Sub-Agents

Combining worktrees with sub-agents creates a powerful workflow:

1. **Worktrees** provide isolated working directories
2. **Sub-agents** work independently in each worktree
3. **Parallel execution** speeds up evaluation
4. **No conflicts** because each agent has its own directory
5. **Easy comparison** by examining different worktrees

---

## When to Use This Workflow

### Ideal Scenarios

**✅ Use this workflow when:**

1. **Multiple valid approaches exist**
   - "Should we use REST or GraphQL?"
   - "Which state management library is best?"
   - "What's the optimal caching strategy?"

2. **Architectural decisions needed**
   - Comparing design patterns
   - Evaluating framework choices
   - Testing different folder structures

3. **Performance comparisons required**
   - Different optimization strategies
   - Alternative algorithm implementations
   - Various rendering techniques

4. **Learning and exploration**
   - Understanding trade-offs
   - Hands-on comparison
   - Building knowledge through experimentation

5. **Stakeholder presentations**
   - Showing multiple options for review
   - Demonstrating pros/cons
   - Getting team feedback

### When NOT to Use This Workflow

**❌ Avoid this workflow when:**

- Single clear implementation path exists
- Task is trivial or straightforward
- Time-sensitive urgent fix needed
- Only one developer requirement
- Resources are limited (disk space, processing power)

---

## Step-by-Step: Parallel Development Workflow

### Phase 1: Planning and Preparation

#### Step 1: Define Your Approaches

Before starting, clearly define what you want to compare.

**Example: Displaying SVG Images in Markdown**

Approaches to evaluate:
1. Inline SVG embedded in markdown
2. Separate SVG file with figure shortcode
3. Hardcoded shortcode with embedded SVG
4. Shortcode wrapper + separate file
5. Generic parameterized shortcode

Document each approach with:
- Brief description
- Expected pros/cons
- Files that will be modified/created
- Success criteria

#### Step 2: Prepare Your Repository

Ensure your repository is in a clean state:

```bash
# Check status
git status

# Commit any pending changes
git add .
git commit -m "Prepare for parallel development experiment"

# Ensure you're on main/master
git checkout main
```

**Pro Tip:** Create a snapshot commit before starting so you can easily roll back if needed.

### Phase 2: Create Worktrees

#### Step 3: Create Worktrees for Each Approach

Create a worktree for each approach you want to evaluate:

```bash
# Syntax: git worktree add <path> -b <branch-name>

git worktree add ../my-project-approach1 -b approach-1-inline-svg
git worktree add ../my-project-approach2 -b approach-2-separate-file
git worktree add ../my-project-approach3 -b approach-3-shortcode
git worktree add ../my-project-approach4 -b approach-4-hybrid
git worktree add ../my-project-approach5 -b approach-5-parameterized
```

**What this does:**
- Creates new directory: `../my-project-approach1`
- Creates new branch: `approach-1-inline-svg`
- Checks out the branch in that directory
- Links to the same `.git` repository

#### Step 4: Verify Worktrees

```bash
git worktree list
```

**Expected output:**
```
/path/to/my-project                    abc123 [main]
/path/to/my-project-approach1          abc123 [approach-1-inline-svg]
/path/to/my-project-approach2          abc123 [approach-2-separate-file]
/path/to/my-project-approach3          abc123 [approach-3-shortcode]
/path/to/my-project-approach4          abc123 [approach-4-hybrid]
/path/to/my-project-approach5          abc123 [approach-5-parameterized]
```

### Phase 3: Launch Parallel Sub-Agents

#### Step 5: Prepare Agent Instructions

For each approach, prepare detailed instructions for the sub-agent.

**Template for Agent Instructions:**

```
You are Agent [N] working on [APPROACH NAME].

CRITICAL WORKING DIRECTORY INSTRUCTIONS:
- Your working directory is: /absolute/path/to/project-approachN
- You are on branch: approach-N-branch-name
- DO NOT change directories or work in any other location
- All file paths should be relative to this working directory

YOUR TASK:
[Clear description of what to implement]

STEP-BY-STEP INSTRUCTIONS:
1. [Specific action]
2. [Specific action]
3. [Specific action]
...

IMPORTANT NOTES:
- Stage changes (git add) but DO NOT commit
- Report back with summary of what was done
- Work ONLY in the specified directory

EXPECTED RESULT:
[Description of the final state]
```

#### Step 6: Launch All Agents Simultaneously

In Claude Code, launch all agents in a **single message** using multiple Task tool calls:

**Example prompt to Claude Code:**

```
Please launch 5 sub-agents in parallel to implement these approaches:

Agent 1: Implement inline SVG approach in ../my-project-approach1
Agent 2: Implement separate file approach in ../my-project-approach2
Agent 3: Implement shortcode approach in ../my-project-approach3
Agent 4: Implement hybrid approach in ../my-project-approach4
Agent 5: Implement parameterized approach in ../my-project-approach5

[Include detailed instructions for each agent]
```

**What Claude Code does:**
- Sends a single message with 5 Task tool calls
- Launches all 5 agents simultaneously
- Each agent works independently in its own worktree
- No conflicts because they're in different directories

#### Step 7: Monitor Agent Progress

The agents will work in parallel and report back when complete. You'll see output like:

```
Agent 1 Report: Inline SVG approach completed successfully...
Agent 2 Report: Separate file approach completed successfully...
Agent 3 Report: Shortcode approach completed successfully...
Agent 4 Report: Hybrid approach completed successfully...
Agent 5 Report: Parameterized approach completed successfully...
```

### Phase 4: Evaluation and Testing

#### Step 8: Test Each Approach

Navigate to each worktree and test the implementation:

```bash
# Test Approach 1
cd ../my-project-approach1
npm run build    # or hugo server, or whatever your test command is
npm test

# Test Approach 2
cd ../my-project-approach2
npm run build
npm test

# Repeat for all approaches...
```

**For Hugo projects:**
```bash
# Run Hugo servers on different ports to compare side-by-side
cd ../my-project-approach1
hugo server --buildDrafts --port 1313

# In another terminal
cd ../my-project-approach2
hugo server --buildDrafts --port 1314

# In another terminal
cd ../my-project-approach3
hugo server --buildDrafts --port 1315
```

Now you can visit:
- http://localhost:1313 - Approach 1
- http://localhost:1314 - Approach 2
- http://localhost:1315 - Approach 3

Compare them side-by-side in different browser tabs!

#### Step 9: Review Code Quality

Examine each implementation:

```bash
# View changes for each approach
cd ../my-project-approach1
git diff main

cd ../my-project-approach2
git diff main

# etc.
```

**Evaluation Criteria:**
- Code quality and maintainability
- Performance characteristics
- File structure and organization
- Dependencies added
- Complexity vs. simplicity
- Reusability and scalability
- Best practices compliance

#### Step 10: Compare Approaches

Create a comparison matrix:

| Criteria | Approach 1 | Approach 2 | Approach 3 | Approach 4 | Approach 5 |
|----------|-----------|-----------|-----------|-----------|-----------|
| Simplicity | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| Maintainability | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Reusability | ⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Performance | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Best Practices | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

### Phase 5: Merge and Cleanup

#### Step 11: Choose the Best Approach

Based on your evaluation, select the approach that best meets your needs.

**Decision factors:**
- Technical excellence
- Team preferences
- Project requirements
- Long-term maintainability
- Scalability needs

#### Step 12: Commit in the Chosen Worktree

Navigate to the chosen worktree and commit the changes:

```bash
cd ../my-project-approach5  # Example: chose approach 5

# Verify changes are staged
git status

# Commit with descriptive message
git commit -m "Implement parameterized shortcode for SVG images

Add reusable shortcode that accepts parameters for displaying
SVG diagrams across the site.

Benefits:
- Generic and reusable for all SVG files
- Clean markdown syntax
- Accessible with alt text support
- Follows Hugo best practices"
```

#### Step 13: Merge into Main

```bash
# Switch to main branch (in main worktree)
cd /path/to/my-project
git checkout main

# Merge the chosen approach
git merge approach-5-parameterized --no-edit
```

**Expected output:**
```
Updating abc123..def456
Fast-forward
 layouts/shortcodes/svg-image.html         |   9 +++
 static/images/diagrams/diagram.svg        | 114 +++++++++++++++
 content/tutorials/my-tutorial.md          |   2 +
 3 files changed, 125 insertions(+)
```

#### Step 14: Clean Up Worktrees

Remove the worktrees you no longer need:

```bash
# From the main worktree
cd /path/to/my-project

# List all worktrees
git worktree list

# Remove unused worktrees
git worktree remove --force ../my-project-approach1
git worktree remove --force ../my-project-approach2
git worktree remove --force ../my-project-approach3
git worktree remove --force ../my-project-approach4
git worktree remove ../my-project-approach5  # Already merged, no force needed
```

**Note:** Use `--force` when worktrees have uncommitted changes (since you're discarding them).

#### Step 15: Delete Branches

```bash
# Delete the branches you don't need
git branch -D approach-1-inline-svg
git branch -D approach-2-separate-file
git branch -D approach-3-shortcode
git branch -D approach-4-hybrid
git branch -D approach-5-parameterized
```

#### Step 16: Verify Cleanup

```bash
# Check no worktrees remain
git worktree list
# Should only show main worktree

# Check branches
git branch
# Should only show main (and any other existing branches)

# Check status
git status
# Should show clean working tree
```

---

## Real-World Example

### Scenario: Implementing SVG Display in Hugo

**Goal:** Find the best way to display SVG diagrams in Hugo markdown files.

**Approaches to evaluate:**
1. Inline SVG - Embed SVG directly in markdown
2. Separate file - SVG file + Hugo figure shortcode
3. Hardcoded shortcode - Custom shortcode with embedded SVG
4. Hybrid - Shortcode wrapper + separate SVG file
5. Parameterized - Generic shortcode accepting parameters

### Implementation

#### Step 1: Create Worktrees

```bash
git worktree add ../java-developer-approach1 -b approach-1-inline-svg
git worktree add ../java-developer-approach2 -b approach-2-separate-file
git worktree add ../java-developer-approach3 -b approach-3-shortcode
git worktree add ../java-developer-approach4 -b approach-4-hybrid
git worktree add ../java-developer-approach5 -b approach-5-parameterized
```

#### Step 2: Launch 5 Agents in Parallel

**Prompt to Claude Code:**

```
Please launch 5 sub-agents in parallel to implement different approaches
for displaying SVG diagrams in Hugo:

Agent 1 (Inline SVG):
- Working directory: /path/to/java-developer-approach1
- Task: Extract SVG from reverse_proxy.html and embed directly in markdown
- Insert after line 40 in 06.1-secure-with-self-signed-certificate.md
- Include CSS animations
- Stage changes but don't commit

Agent 2 (Separate File):
- Working directory: /path/to/java-developer-approach2
- Task: Save SVG as static/images/diagrams/reverse-proxy-architecture.svg
- Add figure shortcode in markdown
- Stage changes but don't commit

[Continue for agents 3, 4, 5...]
```

Claude Code sends **one message** with **5 Task tool calls**, launching all agents simultaneously.

#### Step 3: Agents Work in Parallel

Each agent works independently:
- Agent 1 in `java-developer-approach1/` on branch `approach-1-inline-svg`
- Agent 2 in `java-developer-approach2/` on branch `approach-2-separate-file`
- Agent 3 in `java-developer-approach3/` on branch `approach-3-shortcode`
- Agent 4 in `java-developer-approach4/` on branch `approach-4-hybrid`
- Agent 5 in `java-developer-approach5/` on branch `approach-5-parameterized`

**No conflicts** because each is in its own directory!

#### Step 4: Test All Approaches

```bash
# Terminal 1
cd ../java-developer-approach1
hugo server --buildDrafts --port 1313

# Terminal 2
cd ../java-developer-approach2
hugo server --buildDrafts --port 1314

# Terminal 3
cd ../java-developer-approach3
hugo server --buildDrafts --port 1315

# Terminal 4
cd ../java-developer-approach4
hugo server --buildDrafts --port 1316

# Terminal 5
cd ../java-developer-approach5
hugo server --buildDrafts --port 1317
```

Visit all 5 in browser tabs and compare side-by-side!

#### Step 5: Evaluate and Choose

**Comparison:**

| Aspect | Approach 1 | Approach 2 | Approach 3 | Approach 4 | Approach 5 |
|--------|-----------|-----------|-----------|-----------|-----------|
| Markdown size | +119 lines | +1 line | +1 line | +1 line | +1 line |
| Files created | 0 | 1 | 1 | 2 | 2 |
| Reusability | ❌ None | ⚠️ Manual | ✅ Yes | ✅ Yes | ✅✅ Best |
| Maintenance | ❌ Hard | ✅ Easy | ✅ Easy | ✅ Easy | ✅✅ Easiest |
| Flexibility | ❌ None | ⚠️ Limited | ❌ None | ⚠️ Limited | ✅✅ Full |

**Decision:** Approach 5 - Parameterized shortcode wins!

**Reasoning:**
- Clean markdown (just one line)
- Reusable for unlimited SVG files
- Flexible parameters (src, title, alt, class)
- Professional Hugo pattern
- Easy to maintain centrally

#### Step 6: Merge Approach 5

```bash
cd ../java-developer-approach5
git commit -m "Add custom SVG shortcode for displaying diagrams"

cd /path/to/java-developer
git checkout main
git merge approach-5-parameterized
```

#### Step 7: Cleanup

```bash
git worktree remove --force ../java-developer-approach1
git worktree remove --force ../java-developer-approach2
git worktree remove --force ../java-developer-approach3
git worktree remove --force ../java-developer-approach4
git worktree remove ../java-developer-approach5

git branch -D approach-1-inline-svg
git branch -D approach-2-separate-file
git branch -D approach-3-shortcode
git branch -D approach-4-hybrid
git branch -D approach-5-parameterized
```

**Result:** Clean repository with the best approach merged into main!

---

## Best Practices

### Planning Phase

1. **Define clear evaluation criteria** before starting
2. **Limit approaches** to 3-5 maximum (too many becomes overwhelming)
3. **Document approach goals** in a planning document
4. **Set time limits** for each phase to avoid analysis paralysis

### Worktree Management

1. **Use consistent naming** - `project-approach1`, `project-approach2`, etc.
2. **Create worktrees in parent directory** - Keeps main project clean
3. **Use descriptive branch names** - `approach-1-inline-svg` vs `test1`
4. **Always use absolute paths** when specifying worktree locations

### Sub-Agent Instructions

1. **Be extremely specific** about working directory
2. **Use absolute paths** in agent instructions
3. **Instruct agents not to commit** - You want to review first
4. **Include verification steps** - Agents should test their work
5. **Request detailed reports** - What was done, what files changed

### Testing and Evaluation

1. **Test all approaches thoroughly** before deciding
2. **Use different ports** for simultaneous testing (web servers)
3. **Create comparison matrices** to visualize trade-offs
4. **Get team input** if working in a team environment
5. **Document your decision** and reasoning for future reference

### Cleanup

1. **Always clean up worktrees** after merging
2. **Delete unused branches** to keep git history clean
3. **Verify cleanup** with `git worktree list` and `git branch`
4. **Keep the winning branch** briefly if you want to reference it later

---

## Troubleshooting

### Issue: Worktree Creation Fails

**Error:**
```
fatal: '/path/to/worktree' already exists
```

**Solution:**
```bash
# Remove existing directory
rm -rf /path/to/worktree

# Or use a different path
git worktree add ../project-approach1-v2 -b approach-1-v2
```

### Issue: Agent Working in Wrong Directory

**Symptom:** Agent reports "file not found" or modifies wrong files

**Solution:**
- Ensure you specified absolute path in agent instructions
- Verify path with `pwd` in that directory
- Re-launch agent with corrected path

### Issue: Cannot Remove Worktree

**Error:**
```
fatal: 'worktree' contains modified or untracked files
```

**Solution:**
```bash
# Force remove (if you don't need the changes)
git worktree remove --force ../project-approach1
```

### Issue: Merge Conflicts

**Symptom:** Git reports conflicts when merging

**Solution:**
```bash
# View conflicts
git status

# Manually resolve conflicts in files
# Then:
git add .
git commit
```

**Prevention:** Ensure all approaches start from the same base commit.

### Issue: Agents Complete Out of Order

**Symptom:** Some agents finish much faster than others

**This is normal!** Different approaches have different complexity.

**What to do:**
- Review completed agents as they finish
- Start testing the finished ones while others work
- Don't wait for all to complete before starting evaluation

### Issue: Running Out of Disk Space

**Symptom:** Worktree creation fails due to space

**Solution:**
```bash
# Worktrees share .git directory, but files are duplicated
# Consider:
# 1. Clean up node_modules in worktrees
find . -name "node_modules" -type d -prune -exec rm -rf '{}' +

# 2. Reduce number of parallel approaches
# 3. Remove worktrees as you eliminate approaches
```

---

## Advanced Techniques

### Technique 1: Staged Evaluation

Instead of evaluating all at once, evaluate in stages:

**Round 1:** Quick prototypes (2-3 approaches)
- Choose the most promising
- Discard obvious losers

**Round 2:** Refined implementations (2 finalists)
- Polish the top candidates
- Detailed comparison

**Round 3:** Final selection
- Production-ready implementation
- Full testing and documentation

### Technique 2: Hybrid Approaches

You can create a new approach by combining elements from multiple:

```bash
# Create new worktree
git worktree add ../project-approach6 -b approach-6-hybrid

# Manually cherry-pick ideas from approaches 2 and 5
cd ../project-approach6
# Implement hybrid manually or with agent
```

### Technique 3: Incremental Refinement

Ask agents to refine previous attempts:

```bash
# Approach 5 had an error, fix it in place
# Launch new agent in same worktree
"Fix the template parsing error in approach 5 worktree..."
```

### Technique 4: A/B Testing in Production

Keep multiple worktrees for A/B testing:

```bash
# Deploy approach A
cd ../project-approachA
npm run build
# Deploy to server-A

# Deploy approach B
cd ../project-approachB
npm run build
# Deploy to server-B

# Compare metrics and user feedback
```

### Technique 5: Documentation Generation

Use agents to document each approach:

```bash
"Agent 6: Create comparison documentation
- Read all 5 approach implementations
- Generate comparison table
- Document pros/cons
- Create recommendation report"
```

---

## Cleanup and Maintenance

### Post-Merge Cleanup Checklist

After merging your chosen approach:

- [ ] Remove all unused worktrees
- [ ] Delete all approach branches
- [ ] Update documentation with decision rationale
- [ ] Run final tests on main branch
- [ ] Push to remote repository
- [ ] Archive or document the experiment for future reference

### Cleanup Script

Create a script to automate cleanup:

```bash
#!/bin/bash
# cleanup-approaches.sh

echo "Cleaning up approach worktrees and branches..."

# Remove worktrees (force remove, we've already merged what we need)
for i in {1..5}; do
  git worktree remove --force "../project-approach$i" 2>/dev/null && \
    echo "Removed worktree approach$i" || \
    echo "Worktree approach$i not found (already removed)"
done

# Delete branches
for i in {1..5}; do
  git branch -D "approach-$i" 2>/dev/null && \
    echo "Deleted branch approach-$i" || \
    echo "Branch approach-$i not found (already deleted)"
done

echo "Cleanup complete!"
git worktree list
git branch
```

Usage:
```bash
chmod +x cleanup-approaches.sh
./cleanup-approaches.sh
```

### Preserving Experiments

If you want to keep approach branches for future reference:

```bash
# Don't delete branches, just tag them
git tag archive/approach-1-inline-svg approach-1-inline-svg
git tag archive/approach-2-separate-file approach-2-separate-file
# etc.

# Now you can delete branches but keep tags
git branch -D approach-1-inline-svg
git branch -D approach-2-separate-file

# Access archived approaches later
git checkout tags/archive/approach-1-inline-svg
```

---

## Summary

### The Parallel Development Workflow

1. **Plan** - Define approaches and evaluation criteria
2. **Create Worktrees** - One per approach with unique branches
3. **Launch Agents** - Simultaneously in one message
4. **Evaluate** - Test all approaches side-by-side
5. **Merge** - Choose best approach and merge to main
6. **Cleanup** - Remove worktrees and branches

### Key Benefits

✅ **Speed** - Parallel execution is faster than sequential
✅ **Isolation** - No conflicts between approaches
✅ **Comparison** - Easy side-by-side evaluation
✅ **Learning** - Understand trade-offs deeply
✅ **Quality** - Choose the best solution, not just first solution

### When to Use

✓ Multiple architectural approaches exist
✓ Need to compare implementations
✓ Learning and exploration
✓ Team needs to see options
✓ Decision has long-term impact

### Pro Tips

💡 **Start small** - Try 2-3 approaches first
💡 **Clear criteria** - Know how you'll evaluate before starting
💡 **Time box** - Set limits to avoid perfectionism
💡 **Document** - Record decision rationale for future
💡 **Clean up** - Always remove worktrees after merging

---

## Further Reading

- [Git Worktree Documentation](https://git-scm.com/docs/git-worktree)
- [Claude Code Agent Documentation](https://github.com/anthropics/claude-code)
- [Parallel Task Execution Patterns](https://en.wikipedia.org/wiki/Parallel_computing)
- [Software Architecture Decision Records](https://adr.github.io/)

---

## Appendix: Quick Reference Commands

### Worktree Commands

```bash
# Create worktree
git worktree add <path> -b <branch-name>

# List worktrees
git worktree list

# Remove worktree
git worktree remove <path>
git worktree remove --force <path>  # With uncommitted changes

# Clean up deleted worktrees
git worktree prune
```

### Branch Commands

```bash
# List branches
git branch
git branch -a  # Include remote branches

# Delete branch
git branch -d <branch-name>   # Safe delete (only if merged)
git branch -D <branch-name>   # Force delete

# Delete multiple branches
git branch -D branch1 branch2 branch3
```

### Comparison Commands

```bash
# Compare branch to main
git diff main..<branch-name>

# Compare two branches
git diff branch1..branch2

# View files changed
git diff --name-only main..<branch-name>

# View stats
git diff --stat main..<branch-name>
```

### Testing Multiple Servers

```bash
# Hugo
hugo server --buildDrafts --port <port-number>

# Node.js
PORT=<port-number> npm start

# Python
python -m http.server <port-number>

# General web server
# Each approach uses different port: 1313, 1314, 1315, etc.
```

---

**Created:** 2025-01-15
**Last Updated:** 2025-01-15
**Version:** 1.0
**Workflow Status:** Production-Ready ✅
