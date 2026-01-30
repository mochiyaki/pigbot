# Pigbot Extension

A simple VS Code extension that manages Pigbot connection status via a status bar item.

## Requirements
- Visual Studio Code version 1.74.0 or higher
- [Pigbot](https://www.npmjs.com/package/@gguf/pigbot) installed; if not, install it via npm:

```bash
npm install -g @gguf/pigbot
```

## Features

- **Status Bar Item**: Shows connection status with three states:
  - `$(plug) Pigbot - Not connected (click to connect)`
  - `$(sync~spin) Connecting...` - Connection in progress
  - `$(check) Pigbot - Connected to Pigbot`

- **OS Detection**: Automatically detects the operating system and uses:
  - `pigbot status` on Windows (wsl)
  - `pigbot status` on other platforms

- **Auto-Connect**: Optional setting to automatically connect on startup (disabled by default)

## Usage

1. Click the Pigbot status bar item (bottom right) to connect
2. The extension will open a terminal and execute the `pigbot status` command
3. To enable auto-connect, go to Settings and enable `Pigbot: Auto Connect`

## Development

1. Install dependencies: `npm install`
2. Compile: `npm run compile`
3. Press F5 to launch the Extension Development Host

## License
MIT