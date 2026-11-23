## ESP32 Skill

[![License](https://img.shields.io/badge/license-Apache--2.0-507ec6?style=flat&labelColor=282c34&logo=open-source-initiative&logoColor=white)](LICENSE)
[![Version](https://img.shields.io/badge/version-0.1.0-yellow?style=flat&labelColor=282c34)](skills/esp32/SKILL.md)

The [ESP32 skill](skills/esp32/SKILL.md) is a reusable AI agent skill for developing and debugging firmware on the ESP32 family of microcontrollers.
It helps AI agents identify the board and toolchain, check pin assignments, build firmware, and diagnose flashing or serial console issues and bugs.

---

### Quickstart

Copy the `skills/esp32/` directory to your project's skills directory.
Use `.claude/skills/` for Claude Code or `.agents/skills/` for Codex.
For agents that accept file references, you can point them directly to the [`SKILL.md`](skills/esp32/SKILL.md) file.
The skill follows the [Agent Skills format](https://agentskills.io/specification) so it should be compatible with most agents
harnesses like Claude Code, Codex, Antigravity, etc.

#### Example Request

> Use the ESP32 skill to check why my ESP32-S3 firmware builds but produces no serial output.

---

### License

The content of this repository is licensed under the Apache License, Version 2.0 (see [LICENSE](LICENSE)).
