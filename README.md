# Dev Skills - Agent Plugins

A collection of cross-platform AI agent skills for code review, scientific debugging, security audit, and Taro mini-app generation. Compatible with Claude Code, Cursor, VS Code Copilot, Codex, and Trae IDE.

These plugins bundle together skills to provide tailored workflows and instructions for development tasks. By giving the agent domain expertise and repeatable workflows, you drastically reduce mistakes and ensure agents reliably complete tasks following best practices.

## Available Skills

| Skill | Description | Example prompt |
|---|---|---|
| [code-review](skills/code-review/SKILL.md) | Performs general code review on MR/PR, commits, branches, or code diffs. Reviews quality, correctness, maintainability, performance, and best practices, and returns structured issues. | Review the code changes in this MR |
| [debugger](skills/debugger/SKILL.md) | Debugs complex issues requiring runtime evidence collection. Starts a debug server to collect logs via HTTP, following the scientific debugging process (hypothesis → instrumentation → reproduce → analyze → fix → verify). | Debug the login 500 error in production |
| [generate-mini-app](skills/generate-mini-app/SKILL.md) | Generates or modifies runnable code based on Taro for cross-platform mini-apps (WeChat, Alipay, Douyin). Provides end-to-end capability from requirements analysis to generating runnable projects. | Create a Taro mini-app for a shopping cart |
| [security-review](skills/security-review/SKILL.md) | Performs security scanning on code changes. Reports only verifiably exploitable risks introduced by the current changes. Covers untrusted input, authN/authZ, crypto, code execution, and sensitive data exposure. | Run a security audit on this PR |

## Installation

Copy the plugin to your agent's plugin directory. Refer to your agent's documentation for detailed instructions.

## Contributing

Please see [CONTRIBUTING.md](CONTRIBUTING.md) for more information.

## License

MIT License. See [LICENSE](LICENSE) for details.