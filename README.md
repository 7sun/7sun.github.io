# 7sun.github.io

## Password Protection

This site uses [StatiCrypt](https://github.com/robinmoisson/staticrypt) for client-side password protection. The content is encrypted and decrypts in the visitor's browser.

## Editing Workflow

The unencrypted source is stored in `index.source.html` (local only, not committed).

### To Update Content:

1. **Edit the source file:**
   ```bash
   vim index.source.html
   # Or open in your preferred editor
   ```

2. **Re-encrypt:**
   ```bash
   npx staticrypt index.source.html -p pillowfort --short -o encrypted/index.html
   mv encrypted/index.html index.html
   rm -rf encrypted/
   ```

## Requirements

- Node.js/npm (for `npx` command)
- StatiCrypt is automatically downloaded via `npx` (no global install needed)
