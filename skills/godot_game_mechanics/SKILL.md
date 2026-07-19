# Godot Game Mechanics & Feel

Making the core loop actually feel good to play, not just technically function — the gap between "the mechanic works" and "the mechanic is fun."

## Character/enemy behavior: use an explicit state machine

- Any character with more than 2-3 behaviors (idle/move/jump/attack/hurt/dead, or patrol/chase/attack for an enemy) needs an explicit state enum + a single `current_state` variable + a `_process`/`_physics_process` that branches on it — NOT a tangle of independent boolean flags (`is_jumping`, `is_attacking`, `is_hurt`, ...) checked in combination. Boolean-flag soup is exactly how "the player attacks while jumping while already hurt" bugs happen — undefined combinations of flags that were never meant to coexist.
- Every state needs an explicit exit condition and an explicit set of states it's allowed to transition to — write this out (even just as a comment) before implementing, so "can the player attack while stunned?" has a definite planned answer instead of being decided by whichever code path happens to run first.

## Input handling

- Read input in `_unhandled_input` or `_input` for discrete actions (jump, attack, pause) and in `_physics_process` via `Input.is_action_pressed` for continuous/held actions (movement) — mixing these up causes either missed single-press inputs (checking `is_action_just_pressed` inside `_physics_process` can skip a frame's input under variable framerate) or movement that only updates on discrete input events (feels laggy/stepped instead of smooth).
- Buffer time-sensitive inputs (jump, attack) for a few frames — a jump pressed 1-2 frames before landing should still register as a jump on landing ("jump buffering"), and a jump pressed 1-2 frames after leaving a platform should still work ("coyote time"). Skipping both is a common, very noticeable source of a platformer "feeling unresponsive" even when the underlying physics is correct.

## Collision layers and masks

- Assign collision layers/masks DELIBERATELY per node type (player, enemy, enemy-projectile, player-projectile, environment, hazard) rather than leaving everything on the default layer — the default-layer-for-everything setup is how a player's own projectile ends up colliding with the player, or an enemy walks through a wall.
- A hitbox (deals damage) and a hurtbox (receives damage) on the same character should usually be SEPARATE `Area2D`/collision shapes with their own layer/mask pairing, not one shared shape — this is what lets a player's attack hitbox hit an enemy without the player's own hurtbox also being checked against that same attack.

## Physics tuning that reads as "good game feel"

- Jump arcs read better with ASYMMETRIC gravity: apply stronger gravity on the way down than the way up (or cut upward velocity sharply when the jump button is released early, for variable jump height) — pure symmetric gravity (same value up and down) is a common tell of unpolished platformer physics; real platformers almost never use it as-is.
- Acceleration/deceleration should not be instant — moving from 0 to max speed and max speed to 0 over a few frames (not one frame) reads as far more controlled and intentional than binary on/off velocity, even for a simple game.

## No orphaned states (ties to the Plan/judgment checklist)

- Every state a character/menu can enter needs a way OUT that the player can actually trigger — a pause menu with no resume input, a "game over" state with no restart path, a dialogue state with no way to advance/close it. This is checked again at the judgment-checklist stage, but get it right at design time: if you can't state how the player leaves a state, that state isn't finished being planned.
