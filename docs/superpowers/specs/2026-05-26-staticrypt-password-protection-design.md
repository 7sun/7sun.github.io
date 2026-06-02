# StatiCrypt Password Protection Design

**Date:** 2026-05-26  
**Project:** 7sun.github.io (Japan trip itinerary)  
**Goal:** Add simple password protection to the static site using StatiCrypt

## Overview

Implement client-side password protection for the entire index.html page using StatiCrypt, a JavaScript-based encryption tool. The solution encrypts the HTML file locally and serves the encrypted version via GitHub Pages, with decryption happening in the visitor's browser upon correct password entry.

## Requirements

- Protect entire page content behind a password prompt
- Use password: "pillowfort"
- Remember password across visits (localStorage)
- Minimal UI (browser-native prompt acceptable)
- Works with existing GitHub Pages hosting
- No backend or server infrastructure required

## Architecture

### Components

1. **StatiCrypt npm package** — CLI tool for encrypting static HTML files with AES-256
2. **Encrypted index.html** — Self-contained file with encrypted content + decryption logic
3. **Source file management** — Backup system to preserve unencrypted source for editing
4. **Git workflow** — Commit encrypted version to main branch

### Encryption Flow

```
index.html (unencrypted source)
    ↓
npx staticrypt index.html pillowfort -o index.html
    ↓
index.html (encrypted with embedded decrypt JS)
    ↓
git add + commit + push
    ↓
GitHub Pages serves encrypted version
```

### Visitor Flow

**First Visit:**
1. Browser loads encrypted index.html
2. StatiCrypt JS executes immediately
3. Checks localStorage for saved password → not found
4. Shows password prompt
5. User enters "pillowfort"
6. JS decrypts content using password as key
7. Replaces page with decrypted HTML
8. Stores password in localStorage

**Return Visit:**
1. Browser loads encrypted index.html
2. StatiCrypt checks localStorage → password found
3. Auto-decrypts without prompting
4. Renders content immediately

## Implementation Details

### Installation

```bash
# Install StatiCrypt globally
npm install -g staticrypt

# Or use npx (no global install needed)
npx staticrypt --help
```

### Source File Management Strategy

**Approach: Local backup file (not committed)**

1. Before first encryption: `cp index.html index.source.html`
2. Add `index.source.html` to `.gitignore`
3. **Editing workflow:**
   - Edit `index.source.html`
   - Encrypt to `index.html`: `npx staticrypt index.source.html pillowfort -o index.html`
   - Commit encrypted `index.html`

### Encryption Command

```bash
npx staticrypt index.html pillowfort -o index.html
```

**Flags:**
- `-o index.html` — Output to same filename (replaces original)
- Default behavior includes localStorage support (remembering password)
- Minimal UI by default (simple prompt)

### .gitignore Updates

Add to `.gitignore`:
```
index.source.html
node_modules/
```

## Error Handling

| Scenario | Behavior |
|----------|----------|
| Wrong password | Alert: "Bad password!" → prompt reappears |
| No localStorage | Prompt on every visit (still functional) |
| JavaScript disabled | Message: "Please enable JavaScript" |
| Missing password | Prompt remains until valid entry |

## Testing Plan

1. **Local testing before commit:**
   ```bash
   # Encrypt the file
   npx staticrypt index.html pillowfort -o index.html
   
   # Open in browser
   open index.html
   ```

2. **Test cases:**
   - ✓ Password prompt appears
   - ✓ Wrong password shows error
   - ✓ Correct password ("pillowfort") decrypts content
   - ✓ All styling and JavaScript in itinerary work correctly
   - ✓ Close browser, reopen → no prompt (localStorage working)
   - ✓ Incognito mode → prompt appears (fresh session)

3. **Production testing:**
   - Push to GitHub Pages
   - Visit `https://<username>.github.io`
   - Verify password prompt and decryption work live

## Content Update Workflow

When itinerary content changes:

```bash
# 1. Edit the source file
vim index.source.html

# 2. Re-encrypt
npx staticrypt index.source.html pillowfort -o index.html

# 3. Test locally
open index.html

# 4. Commit and push
git add index.html
git commit -m "Update itinerary content"
git push origin main
```

## Security Considerations

**Strengths:**
- Content is genuinely encrypted (AES-256)
- Cannot be viewed without password
- No server-side infrastructure to compromise

**Limitations:**
- Password is used as encryption key (weak passwords = weak encryption)
- Client-side only — sophisticated attackers could try to brute-force
- Password visible in command history (use `history -d` or prefix with space)
- Source backup file must stay local (don't commit unencrypted version)

**Risk Assessment:** Acceptable for personal itinerary shared with friends/family. Not suitable for highly sensitive data.

## Rollback Plan

If issues arise:

1. **Decrypt current version:**
   ```bash
   # Open in browser, enter password, view source, save as index.html
   ```

2. **Revert to unencrypted:**
   ```bash
   cp index.source.html index.html
   git add index.html
   git commit -m "Remove password protection"
   git push origin main
   ```

3. **Git history fallback:**
   ```bash
   git log --oneline  # Find commit before encryption
   git checkout <commit-hash> -- index.html
   ```

## Success Criteria

- ✓ Entire page protected by password
- ✓ Password "pillowfort" decrypts successfully
- ✓ Password remembered across visits
- ✓ Works on GitHub Pages without additional hosting changes
- ✓ Source file safely backed up locally
- ✓ Content updates can be made and re-encrypted easily
