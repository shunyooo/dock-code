# dock-code Design Document

## Overview

dock-code is a VS Code fork optimized for multi-devcontainer workflows and coding agent collaboration.

The core idea: **one window, multiple projects (devcontainers), each with its own workspace state**.

## Motivation

Current IDEs force a 1:1 mapping between windows and remote connections. When working with multiple devcontainers simultaneously (common in microservice architectures or multi-repo workflows), this means juggling multiple windows with no unified view.

Meanwhile, coding agents like Claude Code are becoming central to the development workflow, but IDEs treat them as an afterthought — a terminal tab at best.

dock-code aims to be the IDE where:
- Multiple devcontainers coexist in a single window
- Coding agents are first-class citizens with dedicated UI
- Switching between projects is as fast as switching tmux windows

## Core Concepts

### Project

A **Project** is a single work context. It consists of:

- A connection to a devcontainer (multiple Projects can share the same container)
- An independent pane layout
- Independent agent sessions (e.g., Claude Code conversations)
- Independent file/editor state

Projects are displayed in a flat list in the left sidebar. No nesting.

```
┌───────────────┬────────────────────────────┐
│ Projects      │                            │
│               │  Selected Project's        │
│ ● flamel/     │  Pane Layout               │
│   rewriting   │                            │
│ ○ flamel/     │  ┌──────────┬─────────┐   │
│   bugfix-123  │  │ Claude   │ editor  │   │
│ ○ report/     │  │ Code     │         │   │
│   weekly      │  │ terminal │         │   │
│               │  ├──────────┴─────────┤   │
│ Status:       │  │ terminal / preview │   │
│ 1 running     │  └────────────────────┘   │
│ 2 idle        │                            │
└───────────────┴────────────────────────────┘
```

### Pane

A **Pane** is a subdivision within a Project. Panes can contain:

- Terminal (including Claude Code or other agents)
- Code editor
- Markdown preview
- Webview (HTML preview, graphs, etc.)

Panes are freely arrangeable. Keyboard shortcuts for pane navigation follow tmux conventions.

## Use Cases

### UC1: Multi-project overview
A developer has 3 devcontainers running (flamel, report, polaris). From the sidebar, they can see all projects and each project's agent status (running/idle/error) at a glance. Clicking a project switches the entire workspace to that project's layout.

### UC2: Parallel tasks in the same container
A developer is working on two separate tasks in the same codebase (feature development + bug fix). They create two Projects pointing to the same devcontainer, each with its own Claude Code session, open files, and pane layout.

### UC3: Agent-assisted development
Claude Code edits files in the background. The editor automatically opens and displays the changed files. The developer reviews changes in real-time without manually opening files.

### UC4: Rich content display
Agent outputs (Markdown reports, HTML graphs, data visualizations) are displayed in Webview panes alongside the code editor, providing a richer experience than terminal-only output.

### UC5: Keyboard-driven workflow
Developers switch between projects and panes entirely via keyboard shortcuts, similar to tmux window/pane navigation. No mouse required for core workflows.

## Architecture: What to Change in VS Code

### Core Changes (fork-only)

| Area | Current VS Code | dock-code Target |
|------|----------------|-----------------|
| Remote connection | 1 window = 1 connection | 1 window = N connections (per Project) |
| Workspace state | Single workspace per window | Per-Project workspace state (editors, terminals, layout) |
| Sidebar | Explorer, Search, Git, etc. | Add Project list as primary navigation |

The critical change is **multi-remote connection support**. VS Code's `RemoteAuthority` is currently window-scoped. dock-code needs to make it Project-scoped.

### Extension-level Changes (no core modification needed)

| Feature | Implementation |
|---------|---------------|
| Agent status display | Read status files/API, show in sidebar |
| Auto-open agent-edited files | File watcher + editor auto-open |
| tmux-style keybindings | Keybinding configuration |
| Rich reporting | Webview panel extension |

### Unchanged

- Text editor (Monaco)
- LSP integration (per-container, already works)
- Markdown preview
- Webview API
- Extension ecosystem
- Terminal emulator

## Development Roadmap

### Phase 0: Foundation (current)
- [x] Fork VS Code
- [x] Build successfully on Linux
- [ ] Build and run on macOS
- [ ] Minimal branding change (title: "dock-code")
- [ ] Verify upstream merge workflow with agent

### Phase 1: Project Sidebar
- [ ] Implement Project list sidebar
- [ ] Project CRUD (create, rename, delete)
- [ ] Project switching (save/restore pane layout)
- [ ] Per-Project terminal sessions

### Phase 2: Multi-Devcontainer
- [ ] Refactor RemoteAuthority to be Project-scoped
- [ ] Per-Project LSP connections
- [ ] Per-Project file explorer scope
- [ ] devcontainer auto-detection

### Phase 3: Agent Integration
- [ ] Agent status display in Project list
- [ ] Auto-open files edited by agent
- [ ] Agent output Webview panel
- [ ] tmux-style pane keybindings

### Phase 4: Polish
- [ ] Rich reporting (graphs, structured output)
- [ ] Mobile-friendly remote access
- [ ] Project templates
- [ ] Session persistence across restarts

## Upstream Merge Strategy

dock-code tracks the `microsoft/vscode` upstream. Monthly merges are performed (assisted by coding agents) to stay up-to-date with VS Code releases.

Principles:
- **Minimize core changes**: Keep fork diff as small as possible
- **Prefer extensions**: Implement features as extensions when VS Code API allows
- **Isolate changes**: Keep dock-code specific code in clearly separated modules
- **Agent-assisted merges**: Use Claude Code to resolve merge conflicts during upstream syncs

## Tech Stack

- **Base**: VS Code (Electron + TypeScript)
- **Build**: Node.js 22, npm, gulp
- **Target platforms**: macOS (primary), Linux
- **Required**: fnm/nvm for Node version management

## Getting Started

```bash
# Clone
git clone https://github.com/shunyooo/dock-code.git
cd dock-code

# Node 22
fnm install 22.22.1 && fnm use 22.22.1

# Build dependencies (Linux)
sudo apt-get install -y make g++ pkg-config libx11-dev libxkbfile-dev libsecret-1-dev libkrb5-dev

# Install & compile
npm install
npm run compile

# Run (macOS / Linux with display)
./scripts/code.sh

# Watch mode (for development)
npm run watch
# Then in another terminal:
./scripts/code.sh
```
