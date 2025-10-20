# Security Policy

## Supported Versions

- Active development occurs on the `main` branch
- This is a methodology/specification project - no executable code

## Reporting a Vulnerability

- Please open a GitHub issue with the label `security` to report suspected vulnerabilities
- For sensitive issues, open a minimal issue noting you'd like to share details privately

## Disclosure Policy

- Follow responsible disclosure principles
- Coordinate with maintainers before publicizing vulnerabilities
- Avoid posting payloads that could impact others if reused

## Scope

This is a **task management methodology** with documented workflows. Security concerns focus on:
- AI agents reading and executing workflows
- Data handling in task packages
- Access control in team environments
- Prompt injection risks

## Agentic AI Security

### Why This Matters

This system is designed for AI agents (e.g., Cline, Claude, Cursor, GitHub Copilot) to read workflows, task packages, and execute procedures. Like any system where agents process untrusted content, there are risks:

**Adversarial instructions** can be embedded in:
- Task descriptions and context files
- Workflow documentation
- Event log descriptions
- Markdown comments or hidden sections
- External links that agents might fetch

**Potential attacks** include:
- Exfiltrating sensitive data from task packages
- Disabling validation steps in workflows
- Modifying task priorities or dependencies maliciously
- Injecting false entries into event logs
- Redirecting agents to external malicious content

### Threat Model

**Attack Vectors:**

1. **Malicious Task Content**
   - Task cards with embedded agent instructions
   - Context files containing hidden directives
   - Ledger entries with adversarial descriptions

2. **Compromised Workflows**
   - Modified workflow steps that bypass validation
   - Instructions to skip human sign-off gates
   - Directives to expose sensitive information

3. **External References**
   - Links in task packages to malicious sites
   - Fetched content containing prompt injections
   - Documentation that redirects agent behavior

4. **Social Engineering**
   - Tasks requesting agents to reveal system details
   - Instructions to modify other tasks maliciously
   - Requests to bypass DRI accountability

### Core Mitigations

#### 1. Human-in-the-Loop (Primary Defense)

**Human sign-off required** at critical stages:
- ✅ Moving tasks to `next` stage (prioritization)
- ✅ Moving tasks to `integration` stage (technical review)
- ✅ Moving tasks to `archived` stage (final closure)

This ensures humans review:
- Task content before implementation
- Implementation results before release
- Final state before archival

#### 2. Append-Only Event Log

**Immutable audit trail:**
- Event logs are append-only (never edited)
- All actions are timestamped and attributed
- Malicious modifications are detectable
- Full provenance enables investigation

#### 3. Bounded Context

**Component isolation:**
- Tasks define explicit boundaries
- Agents work within bounded context windows
- Cross-task operations require explicit authorization
- Context limits prevent uncontrolled expansion

#### 4. Idempotency Rules

**Agents propose, don't apply:**
- Agents suggest changes (propose events)
- Humans approve before application
- No direct state modification by agents
- All changes are reviewable

#### 5. Read-Only by Default

**Principle of least privilege:**
- Agents should read workflows and task content
- Write operations require explicit approval
- File system access limited to task packages
- No system-level modifications

### Tool-Specific Guidance

#### Cline

- Use **Plan Mode** for task exploration and workflow review
- Switch to **Act Mode** only for approved implementations
- Keep `requires_approval=true` for:
  - Task creation (WF1)
  - Task movement between stages
  - Event log modifications
  - Index updates
- Review agent-proposed ledger entries before appending
- Validate task package structure after agent operations

#### Claude Code / Cursor / GitHub Copilot

- Disable auto-apply for task management operations
- Review all proposed changes before accepting
- Manually verify workflow step completion
- Check ledger entries for suspicious content
- Validate task movements match workflow rules

#### General AI Agent Usage

- **Never** let agents bypass human sign-off gates
- **Always** review event log entries before appending
- **Validate** task movements against workflow criteria
- **Check** for suspicious external links before following
- **Audit** agent actions via event logs regularly

### Data Handling Guidelines

#### What Belongs in Task Packages

✅ **Safe to include:**
- Task goals and acceptance criteria
- Implementation context and requirements
- Technical specifications and diagrams
- Links to internal documentation
- Non-sensitive configuration examples

❌ **Should NOT include:**
- API keys, tokens, or credentials
- Personally identifiable information (PII)
- Production database credentials
- Private encryption keys
- Sensitive customer data
- Internal IP or trade secrets

#### Handling Sensitive Information

If a task requires sensitive information:

1. **Reference, don't embed:**
   ```markdown
   # Bad
   API_KEY = "sk-abc123..."
   
   # Good
   See secure credential store: vault://prod/api-keys/service-name
   ```

2. **Use placeholder values:**
   ```markdown
   # Context
   Configure OAuth with client_id: <REDACTED - see 1Password>
   ```

3. **Link to secure storage:**
   ```markdown
   Database credentials: See AWS Secrets Manager under prod/db-creds
   ```

4. **Document in context, not in ledger:**
   - Context files can reference secure locations
   - Don't log sensitive info in event descriptions

### Access Control for Teams

#### Repository Permissions

- **Public repositories**: Anyone can read task content
  - Ensure no secrets in task packages
  - Review all commits before pushing
  - Run secret scanners regularly

- **Private repositories**: Control team access
  - Use GitHub teams for permission management
  - Protect main branch with required reviews
  - Audit access logs periodically

#### Task Ownership

- Each task has one DRI (Directly Responsible Individual)
- DRI controls task content and progression
- Hand-offs require explicit acceptance (see WF-HANDOFF)
- Unauthorized modifications are violations

#### Concurrency Coordination

Before tooling exists, use social coordination:
1. Announce before modifying shared state
2. Single writer rule (one person per task)
3. Log all workflow interruptions
4. Document conflict resolution

### Workflow Integrity

#### Protected Operations

These operations **MUST** require human approval:

- ⚠️ Creating task packages (WF1)
- ⚠️ Moving tasks between stages (WF2-WF9)
- ⚠️ Modifying event logs (append only)
- ⚠️ Updating indexes (sync with reality)
- ⚠️ Changing task dependencies (cycle detection)
- ⚠️ Canceling or pausing tasks (affects dependents)

#### Validation Rules

Agents should **not** bypass:

- ✅ Dependency validation (no cycles)
- ✅ Stage entry/exit criteria
- ✅ Human sign-off requirements
- ✅ Event log immutability
- ✅ Index consistency checks
- ✅ Task package structure rules

#### Red Flags in Task Content

Watch for:

- 🚩 Instructions to skip workflow steps
- 🚩 Requests to modify other tasks without DRI approval
- 🚩 Directives to expose system information
- 🚩 Links to external, untrusted URLs
- 🚩 Instructions to bypass validation rules
- 🚩 Requests to execute commands without approval
- 🚩 Hidden instructions in comments or metadata

### Incident Response

If you detect malicious content or behavior:

1. **Immediate action:**
   - Stop the workflow execution
   - Document the incident in a new task
   - Review recent event logs for suspicious entries
   - Check other tasks for similar patterns

2. **Investigation:**
   - Trace the origin (who created the task/content?)
   - Check git history for unauthorized changes
   - Review all tasks by the same author
   - Examine agent action logs

3. **Remediation:**
   - Remove malicious content
   - Revert affected tasks to known-good state
   - Update workflows to prevent recurrence
   - Document learnings in learning stage

4. **Prevention:**
   - Add validation rules to catch similar patterns
   - Update agent guidelines
   - Enhance human review procedures
   - Share learnings with community

### Secret Scanning

Before making repositories public or sharing task packages:

**Manual checks:**
- Search for common patterns: API keys, tokens, passwords
- Review context files for credentials
- Check ledger descriptions for sensitive data
- Validate .gitignore excludes sensitive files

**Automated tools:**
- `gitleaks detect --source . --no-git`
- `trufflehog git file://.`
- GitHub secret scanning (if enabled)

**Best practices:**
- Never commit credentials to task packages
- Use secret management tools for sensitive data
- Add pre-commit hooks for secret detection
- Regularly audit repository content

### Operational Safeguards

#### For Public Repositories

- Enable branch protection on `main`
- Require PR reviews for workflow changes
- Use CODEOWNERS for sensitive files:
  ```
  .work/tasks/*.workflow.md @security-team
  .work/tasks/worker.contract.md @security-team
  ```
- Enable GitHub Advanced Security (if available)
- Run automated secret scans in CI

#### For Private Repositories

- Audit team member access regularly
- Enable audit log review
- Use deployment keys (not personal tokens)
- Rotate credentials if exposed
- Monitor unusual commit patterns

#### CI/CD Integration

```yaml
# Example GitHub Actions workflow
name: Security Checks
on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      
      - name: Run secret scan
        run: |
          docker run -v $(pwd):/scan zricethezav/gitleaks:latest \
            detect --source /scan --no-git --verbose
      
      - name: Validate task structure
        run: |
          python3 .work/scripts/validate-tasks.py
```

## Security Checklist

Before adopting this system:

- [ ] Review all task content for sensitive data
- [ ] Configure appropriate repository permissions
- [ ] Enable branch protection and required reviews
- [ ] Set up secret scanning (manual or automated)
- [ ] Train team on AI agent safety practices
- [ ] Document incident response procedures
- [ ] Establish human approval gates for critical operations
- [ ] Configure audit logging for task modifications

## Responsible AI Agent Usage

### Do:
- ✅ Use agents to read and understand workflows
- ✅ Let agents propose changes for human review
- ✅ Validate agent output against workflow rules
- ✅ Keep audit trails of agent actions
- ✅ Limit agent context to relevant task packages

### Don't:
- ❌ Let agents bypass human sign-off gates
- ❌ Allow agents to modify event logs directly
- ❌ Trust agent-generated content without review
- ❌ Follow external links without validation
- ❌ Grant agents system-level permissions

## Additional Resources

- **Prompt Injection Resources:**
  - [OWASP LLM Top 10](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
  - [Simon Willison on Prompt Injection](https://simonwillison.net/series/prompt-injection/)
  
- **Secret Management:**
  - [GitHub Secret Scanning](https://docs.github.com/en/code-security/secret-scanning)
  - [Gitleaks Documentation](https://github.com/gitleaks/gitleaks)
  
- **Agentic AI Safety:**
  - Cline SECURITY.md (examples section)
  - [AI Safety in Development Tools](https://www.anthropic.com/index/building-effective-agents)

---

**Remember:** This system involves AI agents reading and executing workflows. Security is a shared responsibility between the methodology, tooling, and human oversight. When in doubt, add human review.
