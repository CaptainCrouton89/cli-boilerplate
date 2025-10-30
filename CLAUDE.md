# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This is a generic CLI boilerplate template built using the oclif framework. It provides a reusable foundation for building command-line tools with built-in auth and config management. The structure can be adapted for any API or service integration.

## Build/Development/Testing

**Build:**
```bash
npm run build
```
This compiles TypeScript from `src/` to `lib/` directory using `tsc` and `tsc-alias`.

**Local Development:**
```bash
npm link
```
After building, this makes the `cli` command available globally for testing. Update the `bin` name in `package.json` for your use case.

**Configuration:**
The CLI includes a reusable hierarchical auth/config system:
1. **Config file** (primary): `~/.config/[your-app]/config`
2. **Environment variables**: Automatically loaded from `.env.local`, `.env`, or process.env
3. **Prompt-based auth**: Interactive `auth:login` stores credentials securely

Auth is stored automatically by `auth:login` and `config:set` commands. **BaseCommand is already generic** – all Replicate-specific code has been removed. Just update `ConfigManager.APP_NAME` in `config-manager.ts` to your app name and you're ready to go.

## High-Level Architecture

### oclif Framework Structure

This CLI is built on oclif with a convention-based command structure:
- Commands are defined in `src/commands/` with directory structure matching command syntax
- `auth:login` maps to `src/commands/auth/login.ts`
- `config:get` maps to `src/commands/config/get.ts`
- The oclif config in `package.json` defines `topicSeparator: ":"` for command namespacing
- New commands auto-register in `oclif.manifest.json` on build

### Core Components

**BaseCommand** (`src/utils/command-base.ts`)
- Abstract base class for all commands with built-in auth handling
- **Fully generic** – no service-specific code
- Provides `getConfig()` for loading configuration from hierarchy
- Provides `promptForAuthentication()` for interactive auth
- Provides `handleAuthError()` for recovering from auth failures
- All command classes should extend this base (see auth/config commands for examples)

**Config Manager** (`src/utils/config-manager.ts`)
- Static class with all config operations
- Handles read/write to `~/.config/[app-name]/config` (JSON format)
- `config:get` and `config:set` commands expose config operations
- **Customizable**: Change `APP_NAME` constant to your app name (line 21)
- Supports hierarchical config loading (config file → env vars → dotenv)
- Methods: `load()`, `readToken()`, `writeToken()`, `get()`, `set()`, `delete()`

**Prompt Utilities** (`src/utils/prompt.ts`)
- Interactive prompting for user input
- Used by auth commands and other interactive flows
- Supports text input, password input, and yes/no prompts

### Command Categories

**Authentication** (`src/commands/auth/`)
- `auth:login` - Interactive authentication, saves credentials to config
- `auth:logout` - Removes stored credentials
- `auth:whoami` - Shows authenticated user/account info

**Configuration** (`src/commands/config/`)
- `config:get` - Display config values
- `config:set` - Update config (e.g., set credentials manually)

### Adding New Commands

To add new commands to your CLI:

1. Create a new file in `src/commands/[topic]/[command].ts`
2. Export a default class extending `BaseCommand`
3. Define static properties: `args`, `flags`, `description`
4. Implement the `run()` method
5. Rebuild with `npm run build`

### Key Conventions

**Command Structure:**
- Each command file exports a default class extending `BaseCommand`
- Static properties define `args`, `flags`, and `description`
- Commands use oclif's `Args` and `Flags` for type-safe parameter definitions
- Set `strict = false` on commands that accept dynamic/flexible arguments

**Error Handling:**
- Use `this.error()` to display user-friendly error messages and exit
- Config/auth loading happens at command execution time
- Validation errors should be collected and displayed before API calls

**Config Access:**
- Import `ConfigManager` from `src/utils/config-manager.ts`
- Use `ConfigManager.load()` to get config and environment variables
- Config file location is configurable in `ConfigManager`

## Customization Guide

**BaseCommand is already fully generic – no modifications needed!**

To adapt this boilerplate for your service:

1. **Change app name in `ConfigManager`** (`src/utils/config-manager.ts` line 21):
   ```typescript
   private static readonly APP_NAME = 'your-app-name'; // Change this
   ```
   This controls where config is stored: `~/.config/your-app-name/config`

2. **Update Package Metadata**:
   - Change `package.json` name, bin name, description
   - Update bin file name: `mv bin/cli bin/your-app`
   - Update `oclif.bin` and `oclif.dirname` in `package.json`

3. **Add your domain-specific commands** in `src/commands/`:
   - Extend `BaseCommand`
   - Follow the pattern in `auth/` and `config/` commands
   - Use `ConfigManager` to access/store config

4. **Update README** with your service-specific instructions

5. **Optional - Add service-specific config keys**:
   - Update `Config` interface in `ConfigManager` (line 6)
   - Add methods to ConfigManager for your specific keys
