<p align="center">
  <img src="Cline-PlatformIO-MCP-Server-Logo.png" alt="PlatformIO MCP Server Logo" width="200"/>
</p>

# PlatformIO MCP Server

A board-agnostic [Model Context Protocol](https://modelcontextprotocol.io) (MCP) server for [PlatformIO](https://platformio.org) embedded development. Enables AI agents like [Cline](https://github.com/cline/cline) and [Claude Code](https://docs.anthropic.com/en/docs/claude-code) to interact with PlatformIO's ecosystem of 1,000+ development boards across 30+ platforms.

## Features

- Universal board support: works with any PlatformIO-supported board (ESP32, Arduino, STM32, nRF52, RP2040, etc.)
- Complete development workflow: init, build, upload, and monitor
- Library management: search, install, and list from the PlatformIO registry
- Device discovery: detect connected boards automatically
- Board-agnostic: no hardcoded configs, supports all PlatformIO platforms out of the box

## Supported Platforms

PlatformIO supports 30+ embedded platforms including:

| Vendor | Platforms |
|---|---|
| Espressif | ESP32, ESP8266 |
| Arduino | Uno, Mega, Nano, Due |
| STMicroelectronics | STM32, STM8 |
| Nordic | nRF51, nRF52 |
| Raspberry Pi | RP2040 (Pico) |
| Teensy | All Teensy boards |
| Atmel | AVR, SAM, megaAVR |
| NXP | i.MX RT, LPC |
| Microchip | PIC32 |
| TI | MSP430, TIVA |
| RISC-V | SiFive, GAP |

See the full list at [PlatformIO Boards](https://docs.platformio.org/en/latest/boards/).

## Prerequisites

- Node.js >= 18.0.0
- PlatformIO Core CLI ([install guide](https://platformio.org/install/cli))

```bash
# Install PlatformIO via pip
pip install platformio

# Or via Homebrew on macOS
brew install platformio

# Verify
pio --version
```

## Installation

```bash
git clone https://github.com/jl-codes/platformio-mcp.git
cd platformio-mcp
npm install
npm run build
```

## Configuration

### Cline

Add to your Cline MCP settings:

```json
{
  "mcpServers": {
    "platformio": {
      "command": "node",
      "args": ["/path/to/platformio-mcp/build/index.js"],
      "env": {}
    }
  }
}
```

### Claude Code

Add to `~/.claude/settings.json` (global) or `.claude/settings.json` (project):

```json
{
  "mcpServers": {
    "platformio": {
      "command": "node",
      "args": ["/path/to/platformio-mcp/build/index.js"]
    }
  }
}
```

## MCP Tools

The server exposes 12 tools:

### Board Discovery

#### `list_boards`

Lists all available PlatformIO boards with optional filtering.

| Parameter | Required | Description |
|---|---|---|
| `filter` | no | Filter by platform, framework, or MCU (e.g. "esp32", "arduino", "stm32") |

#### `get_board_info`

Gets detailed information about a specific board.

| Parameter | Required | Description |
|---|---|---|
| `boardId` | yes | Board ID (e.g. "esp32dev", "uno", "nucleo_f401re") |

### Device Management

#### `list_devices`

Lists all connected serial devices. No parameters.

### Project Operations

#### `init_project`

Initializes a new PlatformIO project.

| Parameter | Required | Description |
|---|---|---|
| `board` | yes | Board ID |
| `projectDir` | yes | Project directory path |
| `framework` | no | Framework (e.g. "arduino", "espidf", "mbed") |
| `platformOptions` | no | Additional platform options |

#### `build_project`

Compiles the project and generates firmware binary.

| Parameter | Required | Description |
|---|---|---|
| `projectDir` | yes | Path to project directory |
| `environment` | no | Specific environment from platformio.ini |

#### `clean_project`

Removes build artifacts.

| Parameter | Required | Description |
|---|---|---|
| `projectDir` | yes | Path to project directory |

#### `upload_firmware`

Uploads compiled firmware to a connected device.

| Parameter | Required | Description |
|---|---|---|
| `projectDir` | yes | Path to project directory |
| `port` | no | Upload port (auto-detected if omitted) |
| `environment` | no | Specific environment from platformio.ini |

#### `upload_filesystem`

Uploads a SPIFFS/LittleFS filesystem image from the project's `data/` directory to a connected device.

| Parameter | Required | Description |
|---|---|---|
| `projectDir` | yes | Path to project directory |
| `port` | no | Upload port (auto-detected if omitted) |
| `environment` | no | Specific environment from platformio.ini |

#### `start_monitor`

Returns the command for starting serial monitor.

| Parameter | Required | Description |
|---|---|---|
| `port` | no | Serial port |
| `baud` | no | Baud rate (e.g. 115200) |
| `projectDir` | no | Project directory |

### Library Management

#### `search_libraries`

Searches the PlatformIO library registry.

| Parameter | Required | Description |
|---|---|---|
| `query` | yes | Search query |
| `limit` | no | Maximum results (default: 20) |

#### `install_library`

Installs a library from the PlatformIO registry.

| Parameter | Required | Description |
|---|---|---|
| `library` | yes | Library name or ID |
| `projectDir` | no | Project directory (global if omitted) |
| `version` | no | Specific version (e.g. "^6.21.0") |

#### `list_installed_libraries`

Lists installed libraries.

| Parameter | Required | Description |
|---|---|---|
| `projectDir` | no | Project directory (global if omitted) |

## Cline Customization

Cline has several customization features that work well with PlatformIO projects.

### Rules

[Cline Rules](https://docs.cline.bot/customization/cline-rules) let you set persistent instructions that apply to every task. Use them to tell Cline about your project conventions, preferred board targets, or coding standards for embedded C/C++. Rules can be global or project-scoped via `.clinerules`.

### Workflows

[Cline Workflows](https://docs.cline.bot/customization/workflows) are Markdown files that define multi-step processes. Place them in `.clinerules/workflows/` and invoke them with `/` in the chat input.

Example (`build-upload-monitor.md`):

```markdown
# Build, Upload, Monitor

1. Build the PlatformIO project in the current directory
2. If the build succeeds, upload the firmware to the connected device
3. Provide the serial monitor command so I can observe the output
```

### Skills

[Cline Skills](https://docs.cline.bot/customization/skills) are modular instruction sets that load on demand. They package development expertise into a `SKILL.md` with optional bundled docs and templates. Skills live in `.cline/skills/` (project) or `~/.cline/skills/` (global).

### Hooks

[Cline Hooks](https://docs.cline.bot/customization/hooks) are executable scripts that run at key moments in the task lifecycle (task start, before/after tool use, etc.). Use them for validation and guardrails:

- Ensure a successful build exists before allowing firmware upload
- Run static analysis or binary size checks after compilation
- Inject board configuration and project state at task start

### Kanban

For multi-component firmware projects, use the Kanban to break work into tasks and track progress across boards, features, and integration milestones. Useful when a project spans firmware, filesystem uploads, library integration, and hardware bring-up.

## Examples

### Create and upload an ESP32 project

```typescript
// List ESP32 boards
await listBoards("esp32");

// Initialize project
await initProject({
  board: "esp32dev",
  framework: "arduino",
  projectDir: "/path/to/esp32-blink"
});

// Build
await buildProject("/path/to/esp32-blink");

// Upload firmware
await uploadFirmware("/path/to/esp32-blink");

// Monitor
await startMonitor();
```

### Search and install libraries

```typescript
const libraries = await searchLibraries("ArduinoJson", 10);

await installLibrary("ArduinoJson", {
  projectDir: "/path/to/my-project",
  version: "^6.21.0"
});
```

## Development

```bash
npm run dev          # Development mode with auto-reload
npm test             # Run tests
npm run test:watch   # Run tests in watch mode
npm run lint         # Lint code
npm run format       # Format code
```

## Project Structure

```
platformio-mcp/
├── src/
│   ├── index.ts          # MCP server entry point
│   ├── types.ts          # Type definitions and Zod schemas
│   ├── platformio.ts     # PlatformIO CLI wrapper
│   ├── tools/
│   │   ├── boards.ts     # Board discovery
│   │   ├── devices.ts    # Device listing
│   │   ├── projects.ts   # Project init
│   │   ├── build.ts      # Build and clean
│   │   ├── upload.ts     # Firmware and filesystem upload
│   │   ├── monitor.ts    # Serial monitor
│   │   └── libraries.ts  # Library management
│   └── utils/
│       ├── validation.ts # Input validation
│       └── errors.ts     # Error handling
├── package.json
├── tsconfig.json
└── README.md
```

## Troubleshooting

### PlatformIO not found

1. Install: `pip install platformio`
2. Verify: `pio --version`
3. Ensure `pio` or `platformio` is in your PATH

### Board not found

Board IDs are case-sensitive. List available boards with `pio boards` or search at [PlatformIO Boards](https://docs.platformio.org/en/latest/boards/).

### Upload failures

- Check that the device is connected and powered
- Try a different USB cable
- Verify the port with `list_devices`
- Reset the device
- Close other programs using the serial port

### Build errors

- Check source code for syntax errors
- Ensure required libraries are installed
- Verify `platformio.ini` configuration
- Clean and rebuild: `pio run -t clean`

## Contributing

Contributions welcome. Open an issue or submit a pull request.

## License

MIT. See [LICENSE](LICENSE).

## Links

- [PlatformIO](https://platformio.org) | [Boards](https://docs.platformio.org/en/latest/boards/) | [Libraries](https://registry.platformio.org) | [Community](https://community.platformio.org)
- [Model Context Protocol](https://modelcontextprotocol.io)
- [Cline](https://github.com/cline/cline) | [Cline Docs](https://docs.cline.bot)
