# Dexter Dialogue Portrait Asset Archive

This archive collects **all generated Dexter portrait images currently available from this conversation**.

## Purpose
These assets are intended to be used as **speaker portraits** for when **Dexter speaks to the player** in dialogue boxes, tutorial popups, hint windows, or other reactive UI moments.

## What is included
- **4 selfie-style concept images**
- **4 front-facing portrait exploration images**
- **10 dialogue portraits in batch 1**
- **10 dialogue portraits in batch 2**
- **9 dialogue portraits in batch 3**

Total included image files: **37 PNG assets**.

## Archive structure
- `01-selfie-explorations/` — early concept images with a selfie/text-message feel
- `02-front-facing-explorations/` — refined front-facing portrait experiments
- `03-dialogue-portraits-batch-1/` — first full dialogue portrait set
- `04-dialogue-portraits-batch-2/` — second full dialogue portrait set
- `05-dialogue-portraits-batch-3/` — third dialogue portrait set

## Naming convention
Files were renamed with consistent `dexter-...` names for easier integration into a game project.

Examples:
- `dexter-dialogue-b1-alert.png`
- `dexter-dialogue-b2-surprised.png`
- `dexter-dialogue-b3-thinking.png`

## Suggested use
Swap Dexter's portrait whenever the player advances dialogue.
Useful mapping ideas:

- neutral/default -> `dexter-dialogue-b1-neutral.png`
- annoyed -> `dexter-dialogue-b1-annoyed.png` or `dexter-dialogue-b3-annoyed.png`
- smug/sassy -> `dexter-dialogue-b1-smug.png` or `dexter-dialogue-b3-smug.png`
- worried -> `dexter-dialogue-b3-worried.png`
- alert/warning -> `dexter-dialogue-b1-alert.png` or `dexter-dialogue-b3-barking.png`
- happy/success -> `dexter-dialogue-b2-happy.png`, `dexter-dialogue-b2-joyful.png`, or `dexter-dialogue-b3-gentle-smile.png`
- thinking/explaining -> `dexter-dialogue-b2-pensive.png` or `dexter-dialogue-b3-thinking.png`

## Note
This package includes all currently available generated image files from the session, renamed and organized for use as Dexter speaking portraits.
