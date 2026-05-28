Before writing any code, enforce the following standards:

**1. Plan first**
Produce a complete implementation plan covering architecture, data flow, and edge cases. Present it and get explicit approval before touching any code.

**2. Failure planning**
Identify all potential failure points upfront. Design error handlers, fallbacks, and retry logic as first-class concerns — not afterthoughts.

**3. Research thoroughly**
Investigate relevant tools, APIs, and capabilities (e.g., Claude API features, SDK behaviors, skill integrations) before assuming defaults. Do not guess — verify.

**4. Python best practices**
Follow PEP 8 (79-char line limit), use type hints on all public functions, structured logging via a logger (not print), environment-based config (.env / os.environ), and meaningful custom exceptions. No shortcuts.

**5. Production-grade standards**
Code must be reliable, observable, and maintainable:
- Structured logging at appropriate levels (DEBUG / INFO / WARNING / ERROR)
- Graceful degradation — never silent failures
- Clear separation of concerns — one file = one service, one function = one responsibility
- No hardcoded secrets or magic numbers
- All public functions have docstrings

**6. Lightweight dependencies**
Choose libraries wisely — use lightweight options by default, and only bring in heavier ones when truly necessary. Prefer stdlib or already-present deps before adding new ones.

**7. User-facing error messages (UI log panels)**
Any message that appears in a UI activity log, status panel, or dialog must be written for a non-technical user:
- State what happened in plain English — no exception class names, no Python tracebacks, no internal path details
- Tell the user what action to take — no developer actions (e.g. "run create_mapping.py", "check the config", "re-run with --flag")
- No technical identifiers: no column letters (col B, col L), no row numbers, no formula names (SUMIF, IFERROR), no internal sheet/variable references
- If a file is missing: name the file and say where to put it
- If the file is open elsewhere: say "close it in Excel and try again"
- If something is corrupt/unexpected: say "please contact your administrator" or "re-run the process"
- Raw exception text (`{exc}`) must never be shown in UI log messages — catch and rephrase

Apply these standards to every task in this conversation from this point forward.
