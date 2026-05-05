# AEGIS Framework 🛡️
**Autonomous Enterprise Guardrailed Iteration System**

AEGIS is a deterministic, state-machine-driven orchestration scaffold designed entirely for **GitHub Copilot and VS Code**. 

Unlike traditional Python-based agent frameworks, AEGIS operates purely through Markdown-based prompt engineering and Copilot Custom Instructions. It merges the continuous execution of the Ralph Loop, the evidence-first validation of the Anvil Loop, and strict **Business-Driven Development (BDD)** to ensure that all generated code directly validates business functionality.

---

## 🏗️ Architecture & Project Structure

This repository relies on VS Code's native Copilot features (`@workspace`, `#file`, and `.github/copilot-instructions.md`) to enforce a strict, multi-stage execution loop.

```text
aegis-framework/
├── .github/
│   ├── copilot-instructions.md      # Global Copilot guardrails (DOs and DONTs)
│   └── agents/
│       └── aegis-orchestrator.md    # The core state-machine prompt for Copilot
├── docs/
│   ├── bdd-contracts/               # Gherkin feature files (Business requirements)
│   └── ledger/                      # Markdown/JSON ledger of successful patterns
├── package.json                     # Target for Stage 7b (Self-Upgrade Engine)
└── README.md                        # This file
