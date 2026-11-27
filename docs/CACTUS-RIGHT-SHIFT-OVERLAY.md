# Cactus in-game panel: Right Shift overlay

## User-facing behavior

- Cactus remains the game client; AI-Rescue does not modify or inject into Cactus.
- The user runs Cactus in **borderless fullscreen** mode.
- Pressing **Right Shift** toggles a compact AI-Rescue panel over the game.
- Pressing Right Shift again hides the panel and returns input to the game.
- The panel must not steal keyboard or mouse input while hidden.
- The panel must not pause/resume the bot: this is a visibility toggle, separate from the existing bot pause hotkey.
- The existing DDNet AI desktop window and its current pause shortcut continue to work.

## Important implementation boundary

Cactus is a closed-source third-party client, and this repository has no confirmed Cactus plugin/overlay API. Therefore this feature is an **external transparent desktop overlay**, not a panel rendered inside Cactus itself. It should look attached to the game, but it remains an AI-Rescue-owned window.

Borderless fullscreen is required for the normal desktop-compositor overlay path. Exclusive fullscreen may cover the overlay. Do not promise support for exclusive fullscreen.

## Implementation plan

1. Add a Windows-only, narrowly scoped Right Shift key-down listener. It must detect only the Right Shift virtual key (VK_RSHIFT, 0xA1), ignore key-up and auto-repeat, and never record or transmit other keystrokes. Keep the existing configurable pause hotkey unchanged.
2. Add a dedicated transparent, frameless, always-on-top overlay BrowserWindow. Keep it hidden until toggled. When hidden, make it click-through; when shown, accept pointer input only inside the panel.
3. Render a compact panel using the existing app design system and expose only the required bot status/actions through a minimal preload bridge. Do not load the full desktop shell into a game-sized transparent window.
4. Place the panel on the display containing the Cactus game window. If reliable game-window discovery is unavailable, use the primary display and make the limitation explicit rather than guessing coordinates.
5. On shutdown, unregister the keyboard hook and destroy the overlay. If the hook cannot be installed, show a clear error and leave the existing app shortcuts untouched.

## Acceptance checks

- Right Shift opens and closes the panel while Cactus is focused in borderless fullscreen.
- Left Shift alone does nothing.
- Shift auto-repeat does not rapidly flicker the panel.
- Hidden overlay is click-through and does not intercept game input.
- Visible panel buttons work; clicks outside the panel reach the game.
- Existing pause/resume hotkey still pauses/resumes the bot and is not repurposed.
- Hook failure, Cactus not running, multiple monitors, app restart, and shutdown are handled safely.
- Verify on Windows with Cactus in borderless fullscreen; do not mark this complete based only on code review.

## Current status

Specification only. The runtime keyboard hook, overlay window, panel UI, and in-game verification are not implemented in this change. This document records the agreed behavior and constraints so implementation can be added without pretending that a normal Electron global shortcut can capture a standalone Shift key.
