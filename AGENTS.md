# AGENTS.md

Security-focused fork of [dscho/clipboard2markdown](https://github.com/dscho/clipboard2markdown), which is itself a fork of the original [euangoddard/clipboard2markdown](https://github.com/euangoddard/clipboard2markdown) with Turndown modernization on top. Upstream loads Turndown from `unpkg.com` at runtime with no version pin and no SRI — a supply-chain compromise would leak the user's clipboard, which is exactly what the page reads. This fork vendors dependencies and adds a CSP with `connect-src 'none'` so the app can safely handle sensitive paste content. **Preserve that posture in future changes.** No runtime CDN fetches, no network egress.

## Project

Static single-page app: paste HTML, get Markdown, locally in the browser. No build, no tests — open `index.html` directly.

Core files: `index.html`, `clipboard2markdown.js` (Turndown wrapper with Pandoc-style converter rules), `vendor/turndown.js`, `vendor/turndown-plugin-gfm.js`.

## Gotcha

In `clipboard2markdown.js`, the `pandoc` converter rules and the `escape()` regex cleanup are coupled. E.g. `br` emits `\\\n` and several `escape()` replacements exist specifically to collapse runs of those into paragraph breaks. Change one without the other and you'll get stray `\` in the output.

## Updating vendored dependencies

Pull directly from the npm registry tarball — it's the source of truth, and published versions are immutable. Do not route through unpkg, jsDelivr, or other CDN mirrors: each adds a party that has to be trusted not to substitute the file.

```
curl -sSf https://registry.npmjs.org/turndown/-/turndown-X.Y.Z.tgz | tar -xzO package/dist/turndown.js > vendor/turndown.js
curl -sSf https://registry.npmjs.org/turndown-plugin-gfm/-/turndown-plugin-gfm-X.Y.Z.tgz | tar -xzO package/dist/turndown-plugin-gfm.js > vendor/turndown-plugin-gfm.js
```

Latest version: `curl -sS https://registry.npmjs.org/<pkg>/latest | python3 -c 'import sys,json;print(json.load(sys.stdin)["version"])'`.

Skim the diff before committing — vendoring's value is that a malicious update becomes a visible code change instead of a silent CDN swap.
