## Usage

`/test-webapp-ui <url_or_description> [test-focus]`

Where:

- `<url_or_description>` is either a URL or description of the app
- `[test-focus]` is optional specific test focus (defaults to core functionality)

## Context

- Target: $ARGUMENTS
- Uses browser-use MCP tools for UI testing

## Process

1. **Setup** - Identify target app (report findings at each step):
   - If URL provided: Use directly
   - If description provided,
     - **Try make first**: If no URL provided, check for `Makefile` with `make start` or `make dev` or similar
     - **Consider VSCode launch.json**: Look for `launch.json` in `.vscode` directory for run configurations
     - Otherwise, check IN ORDER:
       a. **Running apps in CWD**: Match `lsof -i` output paths to current working directory
       b. **Static sites**: Look for index.html in subdirs, offer to serve if found
       c. **Project configs**: package.json scripts, docker-compose.yml, .env files
       d. **Generic running**: Check common ports (3000, 3001, 5173, 8000, 8080)
   - **Always report** what was discovered before proceeding
   - **Auto-start** if static HTML found but not served (with user confirmation)
2. **Test** - Interact with core UI elements based on what's discovered
3. **Cleanup** - Close browser tabs and stop any servers started during testing
4. **Report** - Summarize findings in a simple, actionable format

## Output Format

1. **Discovery Report** (if not direct URL):
   ```
   Found: test-react-app/index.html (static React SPA)
   Status: Not currently served
   Action: Starting server on port 8002...
   ```
2. **Test Summary** - What was tested and key findings
3. **Issues Found** - Only actual problems (trust until broken)
4. **Next Steps** - If any follow-up needed

## Notes

- Test UI as a user would, analyzing both functionality and design aesthetics
- **Server pattern** to avoid timeouts:
  ```bash
  # Start server without blocking (returns immediately)
  (cd test-app && exec python3 -m http.server 8002 > /dev/null 2>&1) &
  
  # Get PID for cleanup
  SERVER_PID=$(lsof -i :8002 | grep LISTEN | awk '{print $2}')
  
  # ... do testing ...
  
  # Clean up
  kill $SERVER_PID 2>/dev/null || true
  ```
- Focus on core functionality first, then visual design
- Keep browser sessions open only if debugging errors or complex state
- **Always cleanup**: Close browser tabs with `browser_close_tab` after testing
- **Server cleanup**: Always kill any servers started during testing using saved PID

## Visual Testing Focus

- **Layout**: Spacing, alignment, responsive behavior
- **Design**: Colors, typography, visual hierarchy
- **Interaction**: Hover states, transitions, user feedback
- **Accessibility**: Keyboard navigation, contrast ratios

## Common App Types

- **Static sites**: Serve any index.html with `python3 -m http.server`
- **Node apps**: Look for `npm start` or `npm run dev`
- **Python apps**: Check for uvicorn, Flask, Django
- **Port conflicts**: Try next available (8000→8001→8002)
