# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mobile Next - MCP server for Mobile Development and Automation that provides iOS, Android, Simulator, Emulator, and physical device automation through the Model Context Protocol (MCP). This server enables scalable mobile automation through a platform-agnostic interface, eliminating the need for distinct iOS or Android knowledge.

The server allows AI agents and LLMs to interact with native iOS/Android applications and devices through structured accessibility snapshots or coordinate-based taps based on screenshots.

## Development Commands

### Building and Development
- `npm run build` - Compile TypeScript to JavaScript in lib/ directory
- `npm run watch` - Watch for TypeScript changes and auto-compile
- `npm run clean` - Remove compiled lib/ directory

### Testing and Quality
- `npm test` - Run test suite with nyc coverage using mocha and ts-node
- `npm run lint` - Run ESLint on TypeScript files
- Test individual modules: `npx mocha --require ts-node/register test/android.ts`

### Running the Server
- `npm run start-http` - Start HTTP streamable server on port 10000
- `node lib/index.js` - Run compiled server (stdio mode, default)
- `node lib/index.js --port 8080` - Run HTTP server on custom port

## Architecture Overview

### Core Components

**Entry Point (`src/index.ts`)**
- CLI interface using commander.js
- Supports both stdio and HTTP streamable transports
- HTTP mode uses Express.js server with `/mcp` endpoint

**Server (`src/server.ts`)**
- Main MCP server implementation with tool registration
- Robot abstraction layer for device-agnostic operations
- Tool wrapper with error handling and ActionableError support
- PostHog analytics integration for usage tracking

**Device Management**
- **Android (`src/android.ts`)**: ADB-based automation using UIAutomator XML parsing
- **iOS (`src/ios.ts`)**: WebDriverAgent integration for physical devices
- **iOS Simulator (`src/iphone-simulator.ts`)**: xcrun simctl integration for simulators
- **Robot Interface (`src/robot.ts`)**: Platform-agnostic automation contract

### Platform-Specific Architecture

**Android Implementation**
- Uses ADB (Android Debug Bridge) for device communication
- UIAutomator XML hierarchy parsing for element discovery
- Supports both mobile devices and Android TV (DPAD controls)
- Platform tools detection via ANDROID_HOME environment variable

**iOS Implementation**
- Physical devices: go-ios + WebDriverAgent tunnel (port 8100)
- Simulators: xcrun simctl direct integration
- iOS 17+ requires tunnel for physical device communication
- WebDriverAgent provides screenshot and interaction capabilities

**Cross-Platform Abstractions**
- Robot interface defines common operations (tap, swipe, screenshot, etc.)
- Screen element representation with coordinates and accessibility data
- Unified button mapping and orientation control

### Tool Categories

**Device Selection Tools**
- `mobile_use_default_device` - Auto-select single connected device
- `mobile_list_available_devices` - Enumerate all available devices
- `mobile_use_device` - Select specific device by ID and type

**App Management Tools**
- `mobile_list_apps` - List installed applications
- `mobile_launch_app` - Launch app by package name
- `mobile_terminate_app` - Stop running application

**Interaction Tools**
- `mobile_take_screenshot` - Capture device screen with ImageMagick optimization
- `mobile_list_elements_on_screen` - Get accessibility hierarchy
- `mobile_tap_element` - Find and tap element by query
- `mobile_click_on_screen_at_coordinates` - Direct coordinate tapping
- `swipe_on_screen` - Gesture navigation with direction and distance
- `mobile_type_keys` - Text input with optional submission

**System Tools**
- `mobile_get_screen_size` - Device dimensions and scale factor
- `mobile_set_orientation` / `mobile_get_orientation` - Device rotation
- `mobile_press_button` - Hardware button simulation
- `mobile_get_log` - Device log extraction with filtering

## Key Dependencies and Libraries

- `@modelcontextprotocol/sdk` - MCP protocol implementation
- `commander` - CLI argument parsing
- `express` - HTTP server for streamable transport
- `fast-xml-parser` - Android UIAutomator XML parsing
- `zod-to-json-schema` - Schema validation and conversion

## Development Environment Setup

### Prerequisites
- Node.js v18+
- Platform-specific tools:
  - **Android**: Android SDK with platform-tools (ADB)
  - **iOS**: Xcode command line tools, go-ios for physical devices
  - **Optional**: ImageMagick for screenshot optimization

### Platform Tools Configuration
- **ANDROID_HOME**: Set to Android SDK root for ADB detection
- **GO_IOS_PATH**: Custom path to go-ios binary (defaults to 'ios' in PATH)

### Code Style and Linting
- Uses TypeScript with strict mode enabled
- ESLint configuration with @typescript-eslint and @stylistic plugins
- Tab-based indentation (configured in eslint.config.mjs)
- Double quotes for strings, single quotes for JSX
- Comprehensive style rules for spacing, brackets, and anti-patterns

## Testing Strategy

- Unit tests in `test/` directory mirror `src/` structure
- Platform-specific test files: `android.ts`, `ios.ts`, `iphone-simulator.ts`
- Image processing tests: `png.ts`
- Uses mocha with ts-node for TypeScript execution
- nyc for test coverage reporting

## Error Handling Patterns

- **ActionableError**: User-actionable errors with helpful messages
- **Robot Abstraction**: Consistent error handling across platforms
- **Tool Wrapper**: Automatic error catching and user-friendly responses
- **Platform Detection**: Graceful fallbacks when platform tools unavailable

## Mobile Automation Workflow

1. **Device Discovery**: Enumerate available devices (simulators, emulators, physical)
2. **Device Selection**: Choose target device based on availability
3. **App Management**: Launch target applications
4. **Element Discovery**: Use accessibility hierarchy or screenshots
5. **Interaction**: Perform taps, swipes, text input based on element data
6. **Validation**: Capture screenshots and verify expected states

## Build Output Structure

- Source: `src/` (TypeScript)
- Compiled: `lib/` (JavaScript, distributed)
- Entry point: `lib/index.js` (executable with shebang)
- Package binary: `mcp-server-mobile` command