---
name: auto-commit
description: |
  Automatically generate meaningful commit messages in Conventional Commits format (with scope)
  and create git commits. Use this skill when the user explicitly requests to commit changes.
---

# Auto Commit

Generate and create git commits with Conventional Commits format messages, including scope based on the affected components.

## Workflow

Follow these steps sequentially when this skill is invoked:

### 1. Analyze Current Changes

Check the repository status and changes:

```bash
git status
git diff --staged
git diff
```

Understand:
- Which files are modified, added, or deleted
- What changes are staged vs unstaged
- The nature and scope of changes

### 2. Present Changes to User

Show the user:
- List of changed files categorized by status (staged/unstaged)
- Brief summary of what changed

Ask the user which files they want to include in the commit using AskUserQuestion tool.

### 3. Stage Selected Files

Based on user selection, stage the appropriate files:

```bash
git add <selected-files>
```

### 4. Generate Commit Message

Analyze the staged changes to create a Conventional Commits message with scope:

**Format**: `type(scope): description`

**Types**:
- `feat`: New feature or functionality
- `fix`: Bug fix
- `refactor`: Code refactoring without behavior change
- `docs`: Documentation changes
- `style`: Code style/formatting changes
- `test`: Test additions or modifications
- `chore`: Build process, dependencies, or tooling changes
- `perf`: Performance improvements

**Scope Selection**:
Determine scope from the primary affected component/directory:
- `queue` - Changes in queue/
- `models` - Changes in models/
- `service` - Changes in service/
- `middleware` - Changes in middlewares/
- `routes` - Changes in routes/
- `utils` - Changes in utils/
- `config` - Changes in config/
- `app` - App-level changes (app.js, server.js)
- Component/feature name for specific features

For changes spanning multiple areas, use the primary affected component or a general scope like `app`.

**Description**:
- Write in imperative mood ("add feature" not "added feature")
- Be concise but descriptive (1-2 sentences)
- Focus on "what" and "why", not "how"
- Keep it under 72 characters for the first line

**Examples**:
- `feat(queue): add retry mechanism for failed activations`
- `fix(service): resolve race condition in container status updates`
- `refactor(models): simplify docker info query logic`
- `docs(README): update installation instructions`

### 5. Create Commit

Execute the commit with the generated message, always including the Co-Authored-By line:

```bash
git commit -m "$(cat <<'EOF'
type(scope): description

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
EOF
)"
```

Verify the commit was created successfully:

```bash
git log -1 --oneline
```

### 6. Ask About Push

After successful commit, ask the user if they want to push to the remote repository using AskUserQuestion tool.

### 7. Optional Push

If the user wants to push:

```bash
git push
```

If the branch doesn't track a remote branch:

```bash
git push -u origin <branch-name>
```

## Important Notes

- **Always use HEREDOC format** for commit messages to ensure proper formatting
- **Never skip the Co-Authored-By line** - it's required for all commits
- **Always verify commit success** with `git log` after committing
- **Check branch tracking** before pushing to use correct push command
- **Follow the existing commit style** of the repository if it differs from these guidelines
- **Keep commit messages concise** - aim for under 72 characters for the first line
- **One logical change per commit** - if changes span multiple concerns, consider asking the user about creating multiple commits
