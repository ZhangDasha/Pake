# src-tauri/ - Tauri Desktop Application Module

[Root](../CLAUDE.md) > **src-tauri**

## Module Responsibility

The `src-tauri/` module contains the Rust-based desktop application built with Tauri v2. It wraps web content in a system WebView (WebKit on macOS/Linux, WebView2 on Windows) and provides native functionality like window management, system tray, notifications, and file downloads.

---

## Entry & Startup

| File | Description |
|------|-------------|
| `src/main.rs` | Binary entry point, calls `lib.rs` |
| `src/lib.rs` | **Core application logic** - plugin setup, menu, window lifecycle |
| `build.rs` | Build script for Tauri |

**Startup Flow**:
1. `main.rs` calls `lib.rs::run()`
2. `lib.rs::run_app()` initializes the application
3. Plugins are registered (window-state, single-instance, oauth, etc.)
4. Menu is configured (macOS only)
5. Window is created with configured settings
6. Injection scripts are loaded into the WebView
7. Application event handlers are attached

---

## External Interfaces

### Tauri Commands (Invoke Handlers)

Commands registered in `src/app/invoke.rs`:

| Command | Parameters | Description |
|---------|------------|-------------|
| `download_file` | url, filename, language | Download file from URL |
| `download_file_by_binary` | filename, binary, language | Save binary data as file |
| `send_notification` | title, body, icon | Show system notification |
| `update_theme_mode` | mode | Update app theme (dark/light) |
| `clear_cache_and_restart` | - | Clear browsing data and restart |

### Window Configuration

Window properties configured via `WindowConfig` in `src/app/config.rs`:

```rust
pub struct WindowConfig {
    pub url: String,
    pub hide_title_bar: bool,
    pub fullscreen: bool,
    pub width: f64,
    pub height: f64,
    pub resizable: bool,
    pub always_on_top: bool,
    pub dark_mode: bool,
    pub activation_shortcut: String,
    // ... and more
}
```

---

## Key Dependencies & Configuration

### Cargo.toml Dependencies

| Dependency | Version | Purpose |
|------------|---------|---------|
| `tauri` | ^2.10.2 | Core framework |
| `serde` | ^1.0.228 | Serialization |
| `tokio` | ^1.49.0 | Async runtime |
| `tauri-plugin-window-state` | ^2.4.1 | Remember window position/size |
| `tauri-plugin-oauth` | ^2.0.0 | OAuth authentication support |
| `tauri-plugin-http` | ^2.5.7 | HTTP client |
| `tauri-plugin-global-shortcut` | ^2.3.1 | Global hotkeys |
| `tauri-plugin-shell` | ^2.3.5 | Open URLs in default browser |
| `tauri-plugin-opener` | ^2.5.3 | Cross-platform open |
| `tauri-plugin-single-instance` | ^2.4.0 | Single instance enforcement |
| `tauri-plugin-notification` | ^2.3.3 | System notifications |

### Configuration Files

| File | Purpose |
|------|---------|
| `tauri.conf.json` | Default Tauri configuration |
| `tauri.macos.conf.json` | macOS-specific configuration |
| `tauri.windows.conf.json` | Windows-specific configuration |
| `tauri.linux.conf.json` | Linux-specific configuration |
| `pake.json` | Pake-specific runtime configuration |
| `capabilities/default.json` | Tauri v2 capabilities/permissions |
| `Cargo.toml` | Rust dependencies and build config |

---

## Data Models

### Core Structures (`src/app/config.rs`)

```rust
// Window configuration
pub struct WindowConfig {
    pub url: String,
    pub hide_title_bar: bool,
    pub fullscreen: bool,
    pub width: f64,
    pub height: f64,
    pub resizable: bool,
    pub url_type: String,  // "web" or "local"
    pub always_on_top: bool,
    pub dark_mode: bool,
    pub disabled_web_shortcuts: bool,
    pub activation_shortcut: String,
    pub hide_on_close: bool,
    pub incognito: bool,
    pub title: Option<String>,
    pub enable_wasm: bool,
    pub enable_drag_drop: bool,
    pub new_window: bool,
    pub start_to_tray: bool,
    pub force_internal_navigation: bool,
    pub zoom: u32,
    pub min_width: f64,
    pub min_height: f64,
    pub ignore_certificate_errors: bool,
}

// Platform-specific configuration
pub struct PlatformSpecific<T> {
    pub macos: T,
    pub linux: T,
    pub windows: T,
}

// Main Pake configuration
pub struct PakeConfig {
    pub windows: Vec<WindowConfig>,
    pub user_agent: UserAgent,
    pub system_tray: FunctionON,
    pub system_tray_path: String,
    pub proxy_url: String,
    pub multi_instance: bool,
}
```

---

## Testing & Quality

### Build Verification

```bash
# Development build
cd src-tauri && cargo build

# Release build
cd src-tauri && cargo build --release

# Check code
cd src-tauri && cargo clippy

# Format code
cd src-tauri && cargo fmt
```

### Platform-Specific Notes

- **macOS**: Requires Xcode command-line tools
- **Windows**: Requires MSVC build tools
- **Linux**: Requires webkit2gtk and other dependencies

---

## Common Tasks

### Adding a New Tauri Command

1. Define handler function in `src/app/invoke.rs`:
```rust
#[command]
pub async fn my_command(app: AppHandle, params: MyParams) -> Result<(), String> {
    // Implementation
}
```

2. Register in `src/lib.rs` invoke_handler:
```rust
.invoke_handler(tauri::generate_handler![
    // ... existing commands
    my_command,
])
```

3. Call from frontend:
```javascript
invoke('my_command', { params: 'value' });
```

### Adding Window Configuration

1. Add field to `WindowConfig` in `src/app/config.rs`
2. Add to TypeScript `WindowConfig` in `bin/types.ts`
3. Update merge logic in `bin/helpers/merge.ts`
4. Apply in window creation in `src/app/window.rs`

### Modifying Injection Scripts

Scripts are in `src/inject/` and injected at runtime:

| Script | Purpose |
|--------|---------|
| `component.js` | UI components (toast, drag region) |
| `event.js` | Event handling (shortcuts, downloads, navigation) |
| `style.js` | CSS overrides for various websites |
| `auth.js` | OAuth/authentication URL detection |
| `custom.js` | User injectable custom code |
| `theme_refresh.js` | Theme change detection |

---

## Module Subdirectories

### src/app/

Core application logic module.

| File | Purpose |
|------|---------|
| `mod.rs` | Module exports |
| `config.rs` | Configuration structures |
| `invoke.rs` | Command handlers |
| `menu.rs` | macOS menu bar |
| `setup.rs` | Plugin initialization |
| `window.rs` | Window creation |

### src/inject/

JavaScript injected into webpages.

| File | Purpose |
|------|---------|
| `component.js` | Toast notifications, drag region |
| `event.js` | Keyboard shortcuts, download handling, context menu |
| `style.js` | Website-specific CSS overrides |
| `auth.js` | OAuth URL detection |
| `custom.js` | User injection target |
| `theme_refresh.js` | Theme synchronization |

---

## Platform-Specific Behavior

### macOS

- Native menu bar with navigation and edit menus
- Title bar overlay (overlay style)
- Dock icon integration
- System tray support
- Universal binary support (Intel + Apple Silicon)

### Windows

- WebView2 rendering engine
- MSI installer
- System tray with minimize-to-tray
- Proxy configuration via browser args

### Linux

- WebKitGTK rendering
- DEB/AppImage/RPM packages
- Desktop file integration
- Custom icons per distribution

---

## Related Files

| File | Purpose |
|------|---------|
| `src/util.rs` | Utility functions (toast messages, file naming, language detection) |
| `src/build.rs` | Tauri build script |
| `capabilities/default.json` | Permission definitions |
| `pake.json` | Default runtime config template |

---

## FAQ

**Q: How do I enable developer tools?**
A: Use `--debug` flag when building, or press `Cmd+Option+I` (macOS) in debug builds.

**Q: Where does the app store data?**
A: In platform-specific config directories (~/Library/Application Support on macOS).

**Q: How do I inject custom JavaScript?**
A: Use the CLI `--inject file.js` option. Code will be added to `custom.js`.

**Q: Can I use custom CSS?**
A: Yes, use `--inject file.css`. CSS is injected into the page head.
