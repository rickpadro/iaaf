# Archetype: CLI Tool

## Default Stack Recommendation

| Layer | Default | Alternative | When to Switch |
|-------|---------|-------------|----------------|
| Runtime | Node.js | Go, Rust, Python | Go for performance + single binary, Rust for systems-level, Python for scripting/data |
| Language | TypeScript | Go, Rust, Python | Match the target audience's ecosystem |
| CLI Framework | Commander.js | yargs, oclif, Clipanion | oclif for plugin systems, yargs for complex options |
| Interactive Prompts | @inquirer/prompts | prompts, Clack | Clack for beautiful UIs, prompts for simplicity |
| Terminal Output | chalk + ora | picocolors + nanospinner | picocolors for smaller bundle |
| Configuration | cosmiconfig | conf, rc | conf for simple key-value, cosmiconfig for flexible (JSON, YAML, .rc) |
| HTTP Client | undici (built-in) | axios, got | axios if team is familiar |
| Testing | Vitest | Jest | — |
| Build | tsup | esbuild, unbuild | unbuild for library-style CLIs |
| Package Manager | pnpm | npm | npm if publishing to npm registry (simpler) |
| Distribution | npm registry | Homebrew, GitHub Releases | Homebrew for Mac users, GitHub Releases for binaries |

## Alternative Stacks

### Go CLI
| Layer | Technology |
|-------|-----------|
| CLI Framework | Cobra |
| Terminal Output | lipgloss + bubbletea |
| Configuration | Viper |
| Testing | go test + testify |
| Distribution | goreleaser (multi-platform binaries) |

### Python CLI
| Layer | Technology |
|-------|-----------|
| CLI Framework | Typer (or Click) |
| Terminal Output | Rich |
| Configuration | pydantic-settings |
| Testing | pytest |
| Distribution | pip / pipx |

## Default Directory Structure (Node.js/TypeScript)

```
src/
  index.ts                         # Entry point — parse args, route to commands
  commands/
    init.ts                        # `mycli init` — project initialization
    generate.ts                    # `mycli generate` — code/file generation
    config.ts                      # `mycli config` — manage configuration
    [command].ts                   # One file per command
  lib/
    config.ts                      # Load/save configuration (cosmiconfig)
    api.ts                         # External API client (if applicable)
    templates.ts                   # Template loading/rendering
    utils.ts                       # Shared utilities
    errors.ts                      # Custom error classes
  templates/                       # Template files (if generating files)
    component.hbs
    config.hbs
  types/
    index.ts
tests/
  commands/
    init.test.ts
    generate.test.ts
  lib/
    config.test.ts
  fixtures/                        # Test fixtures (sample configs, files)
bin/
  mycli.js                         # Executable entry point (#!/usr/bin/env node)
tsconfig.json
tsup.config.ts                     # Build config
package.json
```

## Common Patterns

### Command Structure (Commander.js)

```typescript
// src/index.ts
#!/usr/bin/env node
import { Command } from 'commander'
import { initCommand } from './commands/init'
import { generateCommand } from './commands/generate'
import { configCommand } from './commands/config'

const program = new Command()
  .name('mycli')
  .description('Description of what the CLI does')
  .version('1.0.0')

program.addCommand(initCommand)
program.addCommand(generateCommand)
program.addCommand(configCommand)

program.parse()
```

```typescript
// src/commands/init.ts
import { Command } from 'commander'
import { input, select, confirm } from '@inquirer/prompts'
import chalk from 'chalk'
import ora from 'ora'

export const initCommand = new Command('init')
  .description('Initialize a new project')
  .argument('[name]', 'project name')
  .option('-t, --template <template>', 'template to use', 'default')
  .action(async (name, options) => {
    // If name not provided, prompt for it
    const projectName = name || await input({ message: 'Project name:' })
    const template = await select({
      message: 'Select a template:',
      choices: [
        { name: 'Default', value: 'default' },
        { name: 'Minimal', value: 'minimal' },
        { name: 'Full', value: 'full' },
      ],
    })

    const spinner = ora('Creating project...').start()
    try {
      // Do the work
      await createProject(projectName, template)
      spinner.succeed(chalk.green(`Project ${projectName} created!`))
    } catch (error) {
      spinner.fail(chalk.red('Failed to create project'))
      process.exit(1)
    }
  })
```

### Configuration Management

```typescript
// src/lib/config.ts
import { cosmiconfig } from 'cosmiconfig'

// Searches for: .myclircrc, .myclirc.json, .myclirc.yaml, mycli.config.js, package.json#mycli
const explorer = cosmiconfig('mycli')

export async function loadConfig() {
  const result = await explorer.search()
  return result?.config ?? {}
}

export async function saveConfig(config: object) {
  // Write to ~/.myclirc.json (global) or ./.myclirc.json (local)
  await fs.writeFile(configPath, JSON.stringify(config, null, 2))
}
```

### Error Handling

```typescript
// src/lib/errors.ts
export class CLIError extends Error {
  constructor(message: string, public exitCode = 1) {
    super(message)
  }
}

// In command handler
try {
  await doSomething()
} catch (error) {
  if (error instanceof CLIError) {
    console.error(chalk.red(`Error: ${error.message}`))
    process.exit(error.exitCode)
  }
  // Unexpected error
  console.error(chalk.red('Unexpected error. Please report this bug.'))
  console.error(error)
  process.exit(1)
}
```

### Progress Indicators

```typescript
// Spinner for single operations
const spinner = ora('Installing dependencies...').start()
spinner.succeed('Dependencies installed')

// Progress bar for multi-step
import cliProgress from 'cli-progress'
const bar = new cliProgress.SingleBar({}, cliProgress.Presets.shades_classic)
bar.start(files.length, 0)
for (const file of files) {
  await processFile(file)
  bar.increment()
}
bar.stop()
```

## Build Order

1. **Scaffolding**: Init project, TypeScript + tsup config, create `bin/` entry point, `package.json` with `bin` field.
2. **Command parser**: Set up Commander.js with root command, help, version. Test with `npx tsx src/index.ts --help`.
3. **First command**: Build the primary command end-to-end (e.g., `init` or `generate`).
4. **Interactive prompts**: Add Inquirer prompts for missing arguments. Add chalk for colored output, ora for spinners.
5. **Configuration**: Set up cosmiconfig for loading config files. Add `config set/get` command.
6. **Remaining commands**: Build all other commands one at a time.
7. **Error handling**: Custom error classes, graceful error messages, exit codes.
8. **Testing**: Unit tests for core logic, integration tests for commands (test CLI output).
9. **Build**: Configure tsup to bundle to single file. Test the built output.
10. **Distribution**: Configure `package.json` for npm publish. Add README with usage examples. Publish.

## Distribution Patterns

### npm Registry

```json
// package.json
{
  "name": "mycli",
  "version": "1.0.0",
  "bin": {
    "mycli": "./dist/index.js"
  },
  "files": ["dist"],
  "scripts": {
    "build": "tsup src/index.ts --format esm --dts",
    "prepublishOnly": "npm run build"
  }
}
```

```bash
# Users install with:
npm install -g mycli
# or run without installing:
npx mycli init
```

### GitHub Releases (for Go/Rust binaries)

```yaml
# .github/workflows/release.yml
# On tag push: build for linux/mac/windows, attach to GitHub Release
# Users download binary and add to PATH
```

## Common Pitfalls

- **Don't forget the shebang.** `#!/usr/bin/env node` at the top of the entry point. Without it, the CLI won't run on Unix systems.
- **Don't block on user input when piped.** Detect if stdin is a TTY. If not (piped input), don't show interactive prompts — read from stdin or use flags.
- **Don't swallow errors.** Always show a useful error message AND exit with non-zero code on failure. Other tools may chain your CLI.
- **Don't forget `--help`.** Commander.js generates it automatically, but make sure descriptions are useful.
- **Don't print to stdout and stderr interchangeably.** stdout for data output (so it can be piped). stderr for messages, spinners, progress.
- **Don't hardcode paths.** Use `os.homedir()`, `process.cwd()`, and `path.join()`. Support Windows, Mac, and Linux.

## Skills for Build Phase

| Skill | When |
|-------|------|
| `/deep-research` | Comparing CLI frameworks, distribution strategies |

## See Also

- `knowledge/building-blocks/testing-patterns.md` — Unit testing CLI commands
- `knowledge/building-blocks/deployment-patterns.md` — npm publish, GitHub Releases
