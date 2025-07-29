# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mind the Time is a browser extension that tracks time spent on websites. It runs as a background script with popup, options, and summary interfaces. The extension uses browser local storage for persistence and provides detailed analytics with daily/weekly/monthly aggregations.

## Development Commands

Since this is a browser extension without a build system:

- **Testing**: Load unpacked extension in browser developer mode from `src/` directory
- **Debugging**: Use browser dev tools on background page, popup, and content pages
- **Manifest**: `src/manifest.json` defines extension structure and permissions

## Architecture Overview

### Core Background Scripts (loaded in order per manifest.json)
- `format-time.js` - Time formatting utilities 
- `new-day.js` - Daily data rollover and summary calculations
- `ticker-timer-modes.js` - Timer mode logic and badge display
- `tracking-events.js` - Core time tracking with event listeners
- `main.js` - Initialization, storage management, and shared utilities

### User Interface Components
- `popup/` - Browser action popup with timer controls and current site stats
- `options/` - Settings page for notifications, whitelist, and preferences  
- `summary/` - Detailed analytics dashboard with historical data and export

### Data Flow Architecture
1. **Event Detection**: Browser tab/window events trigger `tracking-events.js`
2. **Time Calculation**: Clock on/off cycles measure time spent per domain
3. **Storage Updates**: Domain data and totals stored in browser.storage.local
4. **Daily Rollover**: `new-day.js` aggregates data into daily/weekly/monthly summaries
5. **UI Updates**: All interfaces read from shared storage for real-time updates

### Storage Schema
- **Domain keys**: `domain.com` → seconds spent (current day)
- **Aggregated data**: `days[]`, `weekSums[]`, `monthSums[]`, `past7daySum`
- **Configuration**: `oButtonBadgeTotal`, `oNotificationsOn`, `oWhitelistArray`, etc.
- **State**: `today`, `totalSecs`, `timerMode`, `nextDayStartsAt`

## Key Technical Concepts

### Timer Modes
- **D (Default)**: Normal tracking with idle detection
- **G (Green)**: Video mode - continues timing during idle periods  
- **B (Blue)**: Time-only mode - tracks total time without domain specifics
- **O (Off)**: Disabled tracking

### Event Handling Pattern
All tracking uses "clock off then clock on" pattern:
1. Browser events trigger `maybe_clock_off_then_pre_clock_on()`
2. Previous timing session is saved to storage
3. New timing session starts for current active tab
4. Timeout-based re-clocking for ticker updates

### Cross-Page Communication
- Background page functions accessed via `browser.runtime.getBackgroundPage()`
- Storage changes trigger `browser.storage.onChanged` listeners
- Summary page refreshes on storage updates and new day events

### Daily Rollover Logic
- Configurable day start time (default midnight, can be offset for late-night users)
- Domain data moved to `days[]` array with metadata (weekNum, monthNum)
- Summary objects recalculated for weeks/months when boundaries are crossed
- 70-day retention limit with automatic cleanup

## Testing Approach

- **Manual Testing**: Load extension in browser, verify tracking across different sites
- **New Day Testing**: Uncommented test functions in `summary/index.js` for day boundary testing
- **Storage Inspection**: Use browser dev tools to examine `browser.storage.local`
- **Background Script Debugging**: Access background page via extension management
- **Cross-Browser**: Test in Firefox (primary) and Chrome (secondary support)

## Code Conventions

- ES6+ JavaScript with async/await patterns
- Mozilla Public License 2.0 
- Consistent error handling with try/catch and console.error()
- Global state in `gState` object, persistent data in browser.storage.local
- Function naming: snake_case for internal functions, camelCase for some newer code
- All background scripts share global scope and can call each other's functions