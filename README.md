# CLI Boilerplate

A generic CLI boilerplate template built with [oclif](https://oclif.io/) that provides a reusable foundation for building command-line tools. Includes built-in authentication and configuration management.

## Features

- **oclif framework** - Powerful CLI framework with built-in help, error handling, and command discovery
- **Authentication** - Interactive auth system with secure config file storage
- **Config Management** - Hierarchical config loading (config file, env vars, dotenv)
- **Ready-to-extend** - Clean structure for adding domain-specific commands
- **TypeScript** - Full TypeScript support with types and tsconfig

## Quick Start

### Installation & Setup

```bash
# Clone or copy this template
cp -r cli-boilerplate my-cli
cd my-cli

# Install dependencies
npm install

# Build the project
npm run build

# Link globally for testing
npm link
```

### Customization

**Quick start – only 3 changes needed:**

1. **Update ConfigManager app name** (`src/utils/config-manager.ts` line 21):
   ```typescript
   private static readonly APP_NAME = 'myapp'; // Change this
   ```

2. **Update package.json:**
   - Change `name` to your CLI name
   - Update `bin` entry (e.g., `"myapp": "./bin/myapp"`)
   - Update `description`, `repository`, `homepage`

3. **Rename the bin file:**
   ```bash
   mv bin/cli bin/myapp
   ```

That's it! **BaseCommand is already fully generic** – no code changes needed there.

Then:
- Add your domain-specific commands in `src/commands/` (extend `BaseCommand`)
- Update README with your service instructions
- See CLAUDE.md for detailed architecture and patterns

### Usage

After customization and building:

```bash
# Help
myapp --help
myapp [command] --help

# Authentication
myapp auth:login
myapp auth:logout
myapp auth:whoami

# Configuration
myapp config:get
myapp config:set key value
```

## Project Structure

```
.
├── src/
│   ├── commands/
│   │   ├── auth/           # Authentication commands (login, logout, whoami)
│   │   └── config/         # Config commands (get, set)
│   └── utils/
│       ├── command-base.ts # Base class for all commands
│       ├── config-manager.ts # Config file and env var loading
│       └── prompt.ts       # Interactive prompting utilities
├── bin/
│   └── cli                 # Executable entry point
├── lib/                    # Compiled JavaScript (generated)
├── tsconfig.json           # TypeScript configuration
├── package.json
└── README.md
```

## Adding Commands

Create a new command file `src/commands/[topic]/[name].ts`:

```typescript
import { Args, Command, Flags } from '@oclif/core';
import { BaseCommand } from '../../utils/command-base.js';

export default class MyCommand extends BaseCommand {
  static override description = 'Command description';

  static override args = {
    arg1: Args.string({ required: true, description: 'First argument' }),
  };

  static override flags = {
    flag1: Flags.string({ description: 'A flag' }),
  };

  async run(): Promise<void> {
    const { args, flags } = await this.parse(MyCommand);

    // Your command logic here
    this.log(`Hello ${args.arg1}`);
  }
}
```

Then rebuild:
```bash
npm run build
myapp [topic]:[name] --help
```

## Configuration Hierarchy

ConfigManager loads values in this order (highest priority first):

1. **dotenv files**: `.env.local` or `.env` in current directory
2. **Environment variables**: `APP_*` prefix (or custom, see ConfigManager)
3. **Config file**: `~/.config/[app-name]/config` (JSON format)

Example accessing config in a command:

```typescript
// Load all config with hierarchy applied
const config = await ConfigManager.load();

// Access specific values
const token = ConfigManager.readToken();
const customValue = ConfigManager.get('some-key');

// Set values (stored in config file)
await ConfigManager.writeToken('new-token');
ConfigManager.set('some-key', 'value');
```

## Authentication Example

The `auth:login` command is ready to use:

```bash
# Interactive login
myapp auth:login

# Check authentication status
myapp auth:whoami

# Logout
myapp auth:logout

# Manual token management
myapp config:set token my-secret-token
myapp config:get token
```

## Building & Publishing

### Build locally
```bash
npm run build
npm link   # Test globally
```

### Publish to npm
```bash
npm version patch  # or minor/major
npm publish
git push --follow-tags
```

## Development

### Watch for changes
```bash
npm run build -- --watch
```

### Test commands
```bash
# After npm link
myapp --help
myapp [topic]:[name]
```

## Documentation

- **`CLAUDE.md`** - Architecture, component details, and customization patterns
- **`src/utils/config-manager.ts`** - Config management API and hierarchy
- **`src/utils/command-base.ts`** - BaseCommand methods and auth handling
- **`src/commands/auth/`** - Ready-to-use auth command examples
- **`src/commands/config/`** - Ready-to-use config command examples

## License

MIT

---

**Getting Started Checklist:**

- [ ] Update `ConfigManager.APP_NAME` in `src/utils/config-manager.ts` (line 21)
- [ ] Update `package.json` (name, bin, description, repository, homepage)
- [ ] Rename `bin/cli` → `bin/[your-app-name]`
- [ ] Run `npm run build && npm link` to test locally
- [ ] Test auth: `your-app auth:login` → `your-app auth:whoami`
- [ ] Add domain-specific commands in `src/commands/` (extend `BaseCommand`)

**That's it!** BaseCommand is already generic and ready to use.
