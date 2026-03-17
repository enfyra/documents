# Server Logs Viewing

The Server Logs page (`/settings/admin/logs`) provides a modern interface for monitoring and inspecting backend log files.

## Access Requirements

- **Root Admin**: Full access
- **Permission**: `read` action on `/logs` route

## Overview

The logs interface displays:

- **Stats Cards**: Total files count, total size, and live monitoring status
- **File List**: Grid of log files with download buttons
- **Log Viewer**: Full-screen viewer with search and actions

## Log File Types

| Icon | Type | Description |
|------|------|-------------|
| 💀 Skull | `crash-*.log` | Fatal crashes, uncaught exceptions |
| ⚠️ Alert | `error-*.log` | Error-level logs (HTTP 500+) |
| 🌐 Globe | `access-*.log` | Access logs |
| 🐛 Bug | `debug-*.log` | Debug logs |
| 📄 File | `app-*.log` | General application logs |

## Viewing Log Content

1. Click on any log file card to open the viewer
2. The viewer displays log entries in JSON format
3. Use browser back button or close button to return to file list

### Log Viewer Actions

| Action | Description |
|--------|-------------|
| 🔍 Search | Search by Log ID or Correlation ID |
| 📋 Copy | Copy visible content to clipboard |
| ⬇️ Download | Download last 10,000 lines |
| 🔄 Reload | Refresh log content |
| ✕ Close | Return to file list |

## Searching Logs

### Smart ID Detection

The search input automatically detects ID type:

- **Log ID** (starts with `log_`): Find specific log entry
- **Correlation ID** (starts with `req_`): Find all logs for a request

### Examples

```
log_mmtajhqm_003e_0n5p    → Find specific log entry
req_1773672046211_abc123  → Find all logs for request
```

### Search Behavior

- Debounced search (500ms delay)
- Shows "No results" message if nothing found
- Clear search to return to full content

## Pagination

Log content is paginated:

- Default: 20 lines per page
- Click "Load More" button at the bottom to load the next page
- Continue loading until no more content (`hasMore: false`)
- Download for full content (up to 10,000 lines)

## Downloading Logs

Click the download button on any file card or in the viewer:

- Downloads last 10,000 lines
- Format: Plain text with JSON entries
- Filename matches the log file name

## Stats Dashboard

The stats cards at the top show:

| Stat | Description |
|------|-------------|
| Total Files | Number of log files |
| Total Size | Combined size of all files |
| Live | Monitoring status indicator |

## Tips

1. **Trace Requests**: Use correlation IDs from error responses to trace request lifecycle
2. **Check Error Logs First**: For 500 errors, check `error.log` before `app.log`
3. **Crash Investigation**: Empty `crash.log` = healthy server
4. **Download for Analysis**: Download large logs for offline analysis

## Responsive Design

The interface adapts to screen size:

- **Desktop**: 3-4 column grid for files, horizontal toolbar in viewer
- **Tablet**: 2 column grid, compact toolbar
- **Mobile**: Single column, stacked toolbar buttons