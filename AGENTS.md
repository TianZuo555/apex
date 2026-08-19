# Stylus & UserCSS Development Guidelines for Agents

This document contains rules, common pitfalls, and preventative guidelines when developing Stylus UserCSS stylesheets (e.g. `stylus/*.user.css`) in this repository.

---

## 1. Troubleshooting "Stylus failed to parse UserCSS: Could not find metadata."

When Stylus intercepts a URL and displays this error, it almost always points to one of the following root causes:

### Cause A: Uncommitted / Unpushed Remote URL (GitHub 404 Response)
- **Mechanism**: Stylus monitors browser navigation and intercepts **any URL ending in `.user.css` or `.user.styl`**.
- **What happens**: If a user navigates to `https://raw.githubusercontent.com/<user>/<repo>/<branch>/path/file.user.css` **before** the branch or file is committed and pushed to GitHub, GitHub responds with HTTP 404 text (`404: Not Found`).
- Stylus intercepts the 404 response body and tries to parse `"404: Not Found"` for `/* ==UserStyle== ... ==/UserStyle== */`. Because no header is present, it reports:
  ```
  Stylus failed to parse UserCSS:
  Could not find metadata.
  ```
- **Fix**:
  1. Commit and push the `.user.css` file to the remote GitHub repository branch before opening the raw URL.
  2. For local testing without pushing, install the file directly via local file drag-and-drop or by pasting into Stylus's **Write new style** (with the **"as UserCSS"** checkbox checked).

### Cause B: Using GitHub Web UI URL Instead of Raw URL
- **Problem**: Opening `https://github.com/<user>/<repo>/blob/master/path/file.user.css` serves an HTML web page (GitHub's code viewer).
- Stylus intercepts the HTML document and fails to find the UserCSS comment block.
- **Fix**: Always use the direct raw URL:
  `https://raw.githubusercontent.com/<user>/<repo>/<branch>/path/file.user.css`

### Cause C: Malformed Metadata Header or Byte Order Mark (BOM)
- Header **must** start at byte 0 with `/* ==UserStyle==\n` (no preceding characters, no UTF-8 BOM, no `/*! ==UserStyle==`).
- Header **must** close with `==/UserStyle== */`.

---

## 2. UserCSS Metadata Header Specifications

The metadata block format:

```css
/* ==UserStyle==
@name           Apex for X (Twitter)
@namespace      github.com/clearlysid/apex
@version        1.0.0
@description    A clean, minimal, distraction-free theme for Twitter / X inspired by the Apex Obsidian theme.
@author         clearlysid (https://github.com/clearlysid/apex)
@homepageURL    https://github.com/clearlysid/apex
@supportURL     https://github.com/clearlysid/apex/issues
@updateURL      https://raw.githubusercontent.com/clearlysid/apex/master/stylus/apex-twitter.user.css
@license        MIT
@preprocessor   uso
==/UserStyle== */
```

### Critical Metadata Constraints
1. **`@author` Format**:
   - Format: `Name [<email>] [(https://url)]`
   - **Never** put descriptive text inside parentheses like `@author Apex (inspired by ...)`. The parser converts any parenthesized string into a `new URL(...)`. Non-URL text in parentheses triggers a fatal `invalidURL` parse error.
2. **Mandatory Fields**:
   - `@name`, `@namespace`, and `@version` are mandatory.
   - `@version` must be valid SemVer (e.g., `1.0.0`).
3. **`@preprocessor` Options**:
   - Use `@preprocessor uso` for `/*[[variable]]*/` substitution tokens.
   - Use `@preprocessor default` for standard CSS variables (`var(--variable)`).

---

## 3. CSS & `@-moz-document` Syntax Rules

1. **NO `@import` inside `@-moz-document`**:
   - Standard CSS and the Stylus UserCSS engine strictly forbid `@import` rules nested inside `@-moz-document` or `@media` blocks.
   - Doing so causes immediate parse errors during installation (`@import not allowed here` / `Expected }`).
2. **Platform Content Security Policy (CSP)**:
   - Target sites (like `x.com` / `twitter.com`) often have strict CSP headers that block external Google Fonts `@import` stylesheets.
   - Always prioritize robust local and system fallback font stacks:
     ```css
     --apex-font-mono: "Geist Mono", ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", monospace;
     --apex-font-sans: "Geist", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
     ```
3. **Clean `@var select` Options**:
   - Keep option values concise single-line strings or CSS tokens.
   - Avoid embedding raw unescaped multiline CSS blocks directly into JSON select dictionaries.

---

## 4. Local Installation & Testing Workflow

When testing before pushing to remote:
1. **In Stylus Manager**:
   - Open Stylus extension popup -> **Manage** -> click **Write new style**.
   - Check the **"as UserCSS"** checkbox.
   - Paste the stylesheet contents and click **Save**.
2. **Or Drag & Drop**:
   - If granted file access in Chrome/Firefox extensions settings, drag `apex-twitter.user.css` directly into a browser tab.
