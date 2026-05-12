# Xcode / Swift (Optional)

Skip this file entirely if you're not working on Apple platform projects. Per the global "language-specific rules go project-scoped" principle, the Swift coding-style and testing sections below should live in a Swift project's `.claude/rules/swift/` directory, not in `~/.claude/rules/`. Copy what you need on a per-project basis.

The **`block-xcodebuild-test.sh` hook** is the exception — it's installed globally because it protects against a macOS-level bug regardless of project.

---

## Swift Coding Style

- Prefer `let` over `var` — only use `var` when the compiler requires it
- Use `struct` with value semantics by default; `class` only when identity or reference semantics are needed
- Follow [Apple API Design Guidelines](https://www.swift.org/documentation/api-design-guidelines/): clarity at point of use, omit needless words
- Typed throws (Swift 6+): `throws(MyError)` not bare `throws`
- Swift 6 strict concurrency: `Sendable` types crossing isolation boundaries, `actor` for shared mutable state, structured concurrency (`async let`, `TaskGroup`) over GCD

## Swift Testing

Use **Swift Testing** (`import Testing`) for all new tests — not XCTest (XCTest only for UI tests):

```swift
@Test("validates email on creation")
func userCreationValidatesEmail() throws {
    #expect(throws: ValidationError.invalidEmail) {
        try User(email: "not-an-email")
    }
}

@Test("handles multiple formats", arguments: ["json", "xml", "csv"])
func validatesFormat(format: String) throws {
    let parser = try Parser(format: format)
    #expect(parser.isValid)
}
```

Each test gets fresh state — set up in `init`, tear down in `deinit`. No shared mutable state.

## Swift Security

- Use **Keychain Services** for sensitive data (tokens, passwords) — never `UserDefaults`
- Use environment variables or `.xcconfig` files for build-time secrets
- Never hardcode secrets — decompilation extracts them trivially
- App Transport Security (ATS) is on by default — do not disable it
- Validate all data from external sources (APIs, deep links, pasteboard) before use

## macOS Native-First

Before writing any helper or utility type, check if macOS already provides it:

| Need | Use this | Don't write |
|------|----------|-------------|
| Render loop | `CADisplayLink` | Custom `Timer` |
| Short audio | `AVAudioPlayer` | Custom mixing |
| Persistence | `@AppStorage` / `UserDefaults` | Custom INI/JSON |
| JSON | `Codable` + `JSONDecoder` | Hand-rolled parser |
| Caching | `NSCache` | Custom LRU |
| State observation | `@Observable` macro | Notification-center plumbing |
| Unit tests | Swift Testing (`@Test`, `#expect`) | XCTest for new code |

When in doubt: macOS probably already provides it. Check Apple developer docs first.

---

## xcodebuild Hook — Critical (Global)

**Problem:** Running `xcodebuild test` with UI tests from Terminal.app crashes it on macOS 26.x — a null-deref in `XCTAutomationSupport` triggered when the test harness injects into the foreground app (Terminal).

**Solution:** Create `~/.claude/hooks/block-xcodebuild-test.sh`:

```bash
#!/usr/bin/env bash
# Blocks `xcodebuild test` without scoping — crashes Terminal on macOS 26.x.
# Safe: build, build-for-testing, test-without-building, scoped test runs.

set -u

input="$(cat)"
cmd="$(printf '%s' "$input" | jq -r '.tool_input.command // empty')"

case "$cmd" in
  *xcodebuild*) ;;
  *) exit 0 ;;
esac

stripped="$(printf '%s' "$cmd" | sed -E 's/build-for-testing|test-without-building//g')"

printf '%s' "$stripped" | grep -qE '(^|[[:space:];&(]|\|\||&&)xcodebuild([[:space:]]|$)' || exit 0
printf '%s' "$stripped" | grep -qE '\btest\b' || exit 0

if printf '%s' "$cmd" | grep -qE -- '-(only|skip)-testing:'; then
  exit 0
fi

cat >&2 <<'MSG'
Blocked: `xcodebuild test` without scoping crashes Terminal.app on macOS 26.x
(XCTAutomationSupport bug when UI tests run from a Terminal-hosted shell).

Use one of:
  xcodebuild build                           # build-only verification
  xcodebuild build-for-testing               # compile tests without running
  xcodebuild test -only-testing:<Bundle>     # restrict to unit tests
  xcodebuild test -skip-testing:<UITests>    # exclude UI tests
  xcodebuild test-without-building           # run a pre-built bundle

For UI tests: ask the user to run them via Xcode's play button.
MSG
exit 2
```

Make it executable and wire it up:

```bash
chmod +x ~/.claude/hooks/block-xcodebuild-test.sh
```

Add to `~/.claude/settings.json` hooks (merge into existing `PreToolUse` array — already present if you used `setup/first-time-setup.md`):

```json
"PreToolUse": [
  {
    "matcher": "Bash",
    "hooks": [
      {
        "type": "command",
        "command": "$HOME/.claude/hooks/block-xcodebuild-test.sh",
        "timeout": 5
      }
    ]
  }
]
```

> Note: the hook only fires when `xcodebuild` appears as an invoked command (at line start or after a shell separator), not merely referenced in a filename or path. This prevents false positives when editing or reading hook files themselves.

## Project-Level Build Hook

Add to your project's `.claude/settings.json` to auto-quit the running app before builds — prevents codesign collisions when the app is already running:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.command // \"\"' | grep -qE 'xcodebuild|swift test' && pkill -x YourApp 2>/dev/null; exit 0"
          }
        ]
      }
    ]
  }
}
```

Replace `YourApp` with your app's process name (as shown in Activity Monitor).
