# bin/ - CLI Module

[Root](../CLAUDE.md) > **bin**

## Module Responsibility

The `bin/` module contains the Command-Line Interface (CLI) tool that drives the Pake build process. It provides the user-facing command for converting web pages into desktop applications.

---

## Entry & Startup

| File | Description |
|------|-------------|
| `cli.ts` | Main entry point - parses command-line arguments and initiates build |
| `dev.ts` | Development mode entry with hot-reload support |
| `helpers/cli-program.ts` | Commander.js program configuration with all CLI options |

**Startup Flow**:
1. User runs `pake <url> [options]`
2. `cli.ts` parses arguments via Commander.js
3. Options are processed by `options/index.ts`
4. `BuilderProvider` selects platform-specific builder
5. Builder prepares environment and executes Tauri build

---

## External Interfaces

### CLI Commands

```bash
# Basic usage
pake https://example.com --name MyApp

# Advanced options
pake https://example.com \
  --name MyApp \
  --icon ./icon.png \
  --width 1280 \
  --height 800 \
  --hide-title-bar \
  --inject custom.css,custom.js
```

### Key Parameters (from `types.ts`)

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `name` | string | auto-generated | Application name |
| `icon` | string | empty | Path to application icon |
| `width` | number | 1920 | Window width in pixels |
| `height` | number | 1080 | Window height in pixels |
| `hideTitleBar` | boolean | false | Hide window title bar (macOS overlay) |
| `multiArch` | boolean | false | Build universal binary (macOS) |
| `targets` | string | platform-specific | Build output format |
| `inject` | string[] | [] | CSS/JS files to inject |
| `debug` | boolean | false | Enable debug output |
| `proxyUrl` | string | empty | Proxy URL for network requests |

---

## Key Dependencies & Configuration

### Package Dependencies (from `package.json`)

- **commander**: ^14.0.3 - CLI argument parsing
- **chalk**: ^5.6.2 - Terminal colors
- **ora**: ^9.3.0 - Loading spinners
- **prompts**: ^2.4.2 - Interactive prompts
- **execa**: ^9.6.1 - Process execution
- **fs-extra**: ^11.3.3 - File system operations
- **icon-gen**: ^5.0.0 - Icon generation
- **sharp**: ^0.34.5 - Image processing

### Build Configuration

- **rollup.config.js**: Bundles TypeScript to `dist/cli.js`
- **tsconfig.json**: TypeScript compilation with path aliases (`@/*` → `bin/*`)

---

## Data Models

### Core Types (`types.ts`)

```typescript
// CLI options passed by user
interface PakeCliOptions {
  name?: string;
  icon: string;
  width: number;
  height: number;
  // ... 30+ options
}

// Extended options with identifier
interface PakeAppOptions extends PakeCliOptions {
  identifier: string;
}

// Window configuration passed to Rust
interface WindowConfig {
  url: string;
  hide_title_bar: boolean;
  width: number;
  height: number;
  // ... window properties
}

// Complete Pake configuration
interface PakeConfig {
  windows: WindowConfig[];
  user_agent: PlatformSpecific<string>;
  system_tray: PlatformSpecific<boolean>;
  proxy_url: string;
  multi_instance: boolean;
}
```

---

## Testing & Quality

### Test Files

| Test File | Description |
|-----------|-------------|
| `tests/unit/file-finding.test.js` | File path resolution tests |
| `tests/integration/workflow-paths.test.js` | Integration workflow tests |
| `tests/index.js` | Main test runner |

### Running Tests

```bash
# Run all tests
pnpm test

# Run specific test category
pnpm test -- --unit
pnpm test -- --integration
pnpm test -- --e2e
```

---

## Common Tasks

### Adding a New CLI Option

1. **Add to types**: Update `PakeCliOptions` in `types.ts`
2. **Set default**: Add default value in `defaults.ts`
3. **Register option**: Add to Commander in `helpers/cli-program.ts`
4. **Process option**: Handle in `options/index.ts` or `helpers/merge.ts`
5. **Pass to Rust**: If needed, update `src-tauri/src/app/config.rs`

### Adding Platform-Specific Build Logic

Each platform has a dedicated builder:
- `MacBuilder.ts` - macOS (.dmg, .app, universal)
- `WinBuilder.ts` - Windows (.msi)
- `LinuxBuilder.ts` - Linux (.deb, .appimage, .rpm)

Override methods in the platform builder as needed.

---

## Related Files

| File | Purpose |
|------|---------|
| `defaults.ts` | Default values for all CLI options |
| `options/icon.ts` | Icon downloading and processing |
| `helpers/merge.ts` | Merges user options with Tauri config |
| `helpers/tauriConfig.ts` | Loads default Tauri configuration |
| `utils/name.ts` | App name generation and sanitization |
| `utils/platform.ts` | Platform detection utilities |
| `utils/url.ts` | URL validation and domain extraction |

---

## FAQ

**Q: How do I debug the CLI?**
A: Use `pake <url> --debug` to enable verbose logging.

**Q: Where are built applications output?**
A: In the project root (e.g., `MyApp.dmg`, `MyApp.exe`).

**Q: How do I add custom CSS/JS injection?**
A: Use `--inject file.css,file.js` with relative or absolute paths.

**Q: What's the difference between app and dmg targets?**
A: `app` creates only the .app bundle (faster), `dmg` creates a disk image installer.
