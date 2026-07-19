# Godot Sprite & Animation Craft

Getting a character/object to visually respond correctly to game state — the difference between a game that looks like placeholder art moving around and one that reads as intentional.

## AnimatedSprite2D / SpriteFrames setup

- Use `AnimatedSprite2D` with a `SpriteFrames` resource for anything with more than one visual state (idle, walk, jump, attack, hurt, death) — don't hand-roll frame-swapping with a plain `Sprite2D` and a timer; `SpriteFrames` gives you named animations, per-animation FPS, and looping control for free.
- Every animated character needs, at minimum: `idle`, a movement animation, and a reaction-to-damage animation if the game has combat/hazards — a character that only ever shows one pose regardless of state reads as unfinished even if the core mechanic works.
- Set `SpriteFrames`' looping explicitly per-animation: looping for `idle`/`walk`, non-looping for one-shot actions (`attack`, `hurt`, `death`) — connect `animation_finished` to transition back to `idle` (or to the death/game-over state) rather than letting a non-looping animation freeze on its last frame with nothing listening for completion.

## State-driven animation, not scattered play() calls

- Drive animation selection from a SINGLE place per character (typically inside the state machine's `_on_state_changed` or equivalent) that maps state -> animation name, not `play("walk")` calls scattered across movement/input-handling code — scattered calls are how a character ends up stuck mid-animation or flickering between two animations in the same frame.
- Guard against calling `play()` with the animation that's ALREADY playing every frame — `AnimatedSprite2D.play(name)` restarts the animation from frame 0 if called again while it's already playing that same animation, which for anything driven from `_physics_process` (e.g. "if moving, play walk") causes a visible stutter/reset every frame instead of a smooth loop. Check `animation != name` before calling `play()`.

## Flipping and facing direction

- Use `flip_h`/`flip_v` on the sprite node for a left/right-symmetric character instead of authoring separate left-facing and right-facing animation sets — cheaper to make, cheaper to maintain, and avoids the two sets silently drifting out of sync.
- If the character has asymmetric equipment/effects attached (a weapon on one specific hand, a UI health bar anchored to a side), those child nodes need their OWN position mirrored when `flip_h` toggles — flipping the parent sprite does not automatically re-mirror a child node's local offset.

## Juice: making motion read as intentional

- A landing after a jump/fall should have a brief visual response (a squash-and-stretch scale tween, a small dust-particle burst, or at minimum a 1-2 frame animation) — a character that lands completely rigidly, with zero visual acknowledgment of the impact, is one of the most common "feels unfinished" tells in a small game.
- Damage/hit feedback needs to be immediate and visible: a brief color flash (modulate to white/red for a few frames), a small knockback, or both — a hit that only changes an internal health number with no visual response is invisible to the player and reads as broken, not as "no feedback needed."
- Keep juice effects SHORT (a few frames to a few tenths of a second) and non-blocking — a squash/flash/particle effect should never delay or gate the actual gameplay logic (the player should still be able to act immediately after landing, not wait for a tween to finish).
