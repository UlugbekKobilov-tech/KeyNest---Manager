# KeyNest 2.0 — Product plan and implementation roadmap

## Product vision

KeyNest 2.0 is a simple, polished password manager for people who want to generate, assess, organize, and securely store their own passwords. It is a multi-user product: every person signs in with their own Google account and gets a private vault.

**Important distinction:** Google sign-in identifies the user. A separate Master Password unlocks the encrypted vault. The Google password is never used as the vault encryption key.

## Product principles

- Security before convenience; do not claim production security until independently reviewed and tested.
- Keep the interface focused. Remove redundant scale/compact controls and decorative features that do not help users manage passwords.
- Keep password strength analysis and password generation on the user's device.
- Store only encrypted vault content in the cloud. Never send plaintext vault passwords to Firestore.
- A user's vault is private to that user. Do not create a global plaintext password registry or expose whether another user has a particular password.
- Preserve the current live `main` site until the new version is reviewed and explicitly approved for release.

## Core product requirements

### 1. Google sign-in and account separation

- Sign in and sign out with Google using Firebase Authentication.
- Give each authenticated account its own vault and data namespace.
- Enforce access rules using Firebase Authentication identity and Firestore Security Rules.
- Google sign-in does not unlock or decrypt the vault.
- Ask for a separate Master Password when creating or unlocking a vault.
- Never store or transmit the Master Password.
- Define a recovery-key process and clearly explain that losing both the Master Password and recovery key means the encrypted vault cannot be recovered.

### 2. Password generator

- Generate passwords with the browser's cryptographically secure random APIs.
- Support configurable length (8–128 characters), uppercase, lowercase, numbers, and symbols.
- Allow users to exclude ambiguous characters.
- Offer a passphrase mode using a curated word list and cryptographically secure randomness.
- Copy generated passwords only after a deliberate user action.
- Do not persist generated password values in generation history by default.

### 3. Password strength checker

- Assess a password locally in the browser; do not send the typed password to the server or analytics.
- Show a 0–100 estimated strength score, a clear category, and actionable improvement tips.
- Evaluate length, character variety, common-password patterns, repeated characters, predictable sequences, and dictionary-like patterns.
- Explain that the percentage is an estimate, not a guarantee or a measured probability of being cracked.
- Avoid logging, analytics capture, or remote requests containing the password being checked.

Suggested display bands:
- 0–20: Very weak
- 21–40: Weak
- 41–60: Fair
- 61–80: Strong
- 81–100: Very strong

### 4. Encrypted personal vault

- Encrypt vault records in the browser before cloud storage.
- Use AES-GCM with a fresh, unique nonce for every encryption operation.
- Derive a vault encryption key from the Master Password using a vetted password-based KDF (prefer Argon2id through a reputable, maintained implementation; document parameters and assess platform compatibility).
- Never store the Master Password or raw derived key in localStorage, sessionStorage, Firestore, logs, or URLs.
- Keep decrypted vault data in memory only while unlocked; clear references and lock on timeout, sign-out, and explicit lock.
- Store only ciphertext and the minimum metadata needed to locate records.
- Add entries with title, username/email, website, password, notes, folder, favorite status, and timestamps. Treat metadata as sensitive too; encrypt it where practical.
- Show passwords masked by default, with explicit reveal and copy actions.
- Add, edit, delete, search, and favorite records.

### 5. Duplicate detection

- Detect duplicates within the current user's own vault after it is unlocked, by comparing decrypted values locally or using a carefully designed keyed comparison.
- Never compare users' passwords through a global plaintext database.
- Explain that KeyNest cannot guarantee a password is unique across the entire internet or across all users.
- Warn users when they are about to save a duplicate within their own vault.

### 6. Folder manager

- Create, rename, and delete folders.
- Move entries between folders.
- Search and filter by folder.
- Show entry counts.
- Confirm destructive actions and avoid accidental data loss.

### 7. Encrypted import and export

- Export a versioned encrypted backup file, not a plaintext JSON dump.
- Require the correct Master Password (or recovery mechanism) to restore a backup.
- Validate file size, schema, version, field types, and record limits before import.
- Make import transactional: validate and decrypt everything before replacing or merging existing vault data.
- Provide clear success and failure messages without exposing secrets.
- Do not silently overwrite existing vault contents.

### 8. Quick Access and keyboard shortcuts

Use shortcuts that avoid common browser and operating-system defaults. Display the shortcut guide inside the app and allow users to discover shortcuts from the interface.

Suggested shortcuts:
- Ctrl + Shift + G: Generate a password
- Ctrl + Shift + K: Focus KeyNest search
- Ctrl + Shift + S: Save the current generated password
- Ctrl + Shift + N: Create a folder
- Ctrl + Shift + L: Lock the vault
- Esc: Close the active dialog

Implementation notes:
- Do not intercept shortcuts while the user is typing in an input, textarea, or editable control, except Esc for closing a dialog when safe.
- Prevent the browser default only when KeyNest actually handles the shortcut.
- Every shortcut must also have a visible clickable equivalent.

### 9. Interface and accessibility

- Minimal, professional interface with responsive desktop and mobile layouts.
- Light and dark themes.
- Clear navigation: Vault, Generator, and Settings.
- Remove compact-mode and arbitrary scale controls unless user testing proves they are needed.
- Accessible labels, keyboard focus states, readable contrast, and clear confirmation/error messages.
- Keep the UI fast and avoid unnecessary animation.

## Security architecture

- Firebase Authentication: account identity only.
- Firestore: encrypted vault payloads and minimal required metadata.
- Client-side cryptography: vetted Web Crypto APIs and a reputable Argon2id implementation.
- Firestore Security Rules: users can access only their own account-scoped documents.
- No plaintext passwords in logs, analytics, URLs, crash reports, or cloud documents.
- Threat-model cross-site scripting (XSS), malicious browser extensions, compromised devices, clipboard exposure, backup theft, account takeover, and weak Master Passwords.
- Use a strict Content Security Policy and safe DOM rendering; do not interpolate untrusted values into innerHTML.
- Add rate limits and abuse protections to account-facing services where appropriate.
- Security review and independent testing are required before public launch.

## Implementation roadmap

### Phase 0 — Protect and isolate the existing product
- Keep the live `main` branch untouched.
- Work only in `keynest-2.0-security-sprint-1` and subsequent reviewed branches.
- Preserve a backup of the current source and document existing plaintext-storage risks.
- Keep interim safety restrictions until encrypted storage is implemented.

### Phase 1 — Product foundation and authentication
- Establish the new minimal UI and navigation.
- Configure Firebase Authentication and Google sign-in.
- Set up Firestore with strict per-user Security Rules.
- Add account lifecycle, sign-out, and empty-vault states.
- Test that one account cannot read another account's documents.

### Phase 2 — Master Password and encrypted vault
- Implement and review key derivation, encryption, nonce handling, and key lifecycle.
- Add create/unlock/lock flows and recovery-key design.
- Implement encrypted record create/read/update/delete.
- Add auto-lock and memory-clearing behavior.
- Test wrong Master Password, tampered ciphertext, missing keys, and recovery scenarios.

### Phase 3 — Generator and strength checker
- Build configurable secure password and passphrase generation.
- Build local 0–100 strength estimation and improvement feedback.
- Add own-vault duplicate warnings.
- Add safe reveal/copy behavior and keyboard shortcuts.

### Phase 4 — Folders, backup, and usability
- Add folder management, search, favorites, and record editing.
- Implement versioned encrypted import/export with strict validation.
- Add shortcut guide, responsive design, and accessibility checks.

### Phase 5 — Verification and release readiness
- Run automated unit and integration tests.
- Test Firestore Security Rules with multiple test accounts.
- Test encryption/decryption, tampering, backup restore, and migration using dummy data.
- Perform browser/device compatibility checks and a security review.
- Prepare privacy policy, terms, support contact, and release notes.
- Deploy only after explicit approval; never auto-publish from the development branch.

## Acceptance criteria

The product is not ready for public launch until all of the following are true:
1. Two separate test accounts cannot access each other's vault documents.
2. Firestore contains no plaintext vault passwords or Master Passwords.
3. Wrong Master Password cannot decrypt the vault.
4. Each encryption uses a fresh nonce; modified ciphertext fails authentication.
5. Strength checking works locally and does not transmit the typed password.
6. Duplicate detection is limited to the current user's unlocked vault.
7. Encrypted backup can be exported and restored with the correct credentials.
8. Invalid or malicious import files are rejected without damaging existing data.
9. Keyboard shortcuts work without disrupting text entry or browser defaults.
10. The complete app passes functional, accessibility, and security review.

## Current branch status

The current development branch is an interim security-hardening step, not a finished password manager. It disables plaintext save/export/import paths and removes password-in-URL sharing while the encrypted vault is being built. Existing legacy localStorage records may still exist in a user's browser in plaintext; this branch hides them but does not automatically encrypt or delete them. Do not use the interim build to store real credentials.
