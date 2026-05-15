# Recommended Resources and References

> [!NOTE]
> GitHub and Microsoft documentation URLs can change over time. If any link below has moved, use the relevant docs home page or site search to locate the current article by title.

---

## Official GitHub Documentation

### Copilot and Agents

- [GitHub Docs — Copilot overview](https://docs.github.com/en/copilot)
- [GitHub Docs — Use GitHub Copilot agents](https://docs.github.com/en/copilot/how-tos/use-copilot-agents)
- [GitHub Docs — About MCP](https://docs.github.com/en/copilot/concepts/about-mcp)
- [GitHub Docs — Extend the coding agent with MCP](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/extend-coding-agent-with-mcp)
- [GitHub Docs — Managing Copilot Memory](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/copilot-memory)

### GitHub Actions

- [GitHub Docs — Automatic token authentication and `GITHUB_TOKEN`](https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication)
- [GitHub Docs — Workflow syntax (permissions, environments, jobs)](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [GitHub Docs — Managing environments for deployment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)
- [GitHub Docs — About OIDC in Actions](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)
- [GitHub Docs — Storing workflow data as artifacts](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/storing-workflow-data-as-artifacts)

### Repository Settings and Governance

- [GitHub Docs — About CODEOWNERS](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)
- [GitHub Docs — About protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)

---

## Useful Practice Resources

- [GitHub Skills courses for Actions and workflow automation](https://skills.github.com)
- [GitHub Actions example repositories](https://github.com/actions)
- [MCP Registry](https://github.com/modelcontextprotocol/servers)
- [Official Model Context Protocol docs](https://modelcontextprotocol.io/introduction)

---

## How to Use These Resources Efficiently

1. **Read once for concepts** — understand what each control or feature does.
2. **Convert concepts into GitHub controls** — for every concept, identify the matching permission, workflow rule, review rule, or approval gate.
3. **Build tiny example repos** — one workflow per high-risk topic is worth more than reading the same page twice.
4. **Rehearse scenario questions** using the [cheat-sheet tables](./cheat-sheets.md) and per-topic exam questions.

---

## Final Exam Strategy

- Think **GitHub control plane first**: issues, PRs, Actions, environments, reviews, logs, artifacts.
- Prefer **least privilege** over convenience.
- Prefer **human approval** for high-risk or production actions.
- Prefer **durable audit trails** over chat-only reasoning.
- Prefer **single constrained agents** unless specialization clearly improves outcomes.
- When a question presents speed vs safety, the exam rewards the answer that keeps safety, observability, and accountability intact.

> [!IMPORTANT]
> A strong GH-600 answer is usually the one that combines **clear role boundaries**, **minimal permissions**, **durable state/auditability**, and **human oversight for risky actions**.
