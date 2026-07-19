# Godot UI/UX Craft

Menus and HUDs are usually the first thing a player sees and the thing they interact with most between gameplay moments — a game with solid mechanics and a barebones/default-styled UI still reads as unfinished.

## Every screen needs a complete navigation loop

- Main menu -> gameplay -> pause -> (resume OR quit-to-menu) -> game-over/win -> (retry OR quit-to-menu) is the minimum complete loop for almost any small game. Before writing any UI code, list every screen and, for each one, exactly which button/input leads to which other screen — a screen with no way out (see godot_game_mechanics's "no orphaned states") is the single most common small-game UI bug.
- Pause should be reachable from gameplay via a consistent input (usually Escape/Start) and should actually halt gameplay (`get_tree().paused = true` plus `process_mode = Node.PROCESS_MODE_ALWAYS` on the pause menu itself so IT still responds to input while everything else is frozen) — a "pause menu" that pops up over gameplay that keeps running underneath it is not actually pausing.

## Control node layout

- Use `Control` node anchors/containers (`VBoxContainer`, `HBoxContainer`, `MarginContainer`, `CenterContainer`) instead of manually positioning every UI element with fixed pixel coordinates — fixed positioning breaks the moment the game runs at a different resolution/aspect ratio than whatever the layout was eyeballed against.
- Buttons and other interactive controls need a minimum comfortable hit area (roughly 40x40 px at the game's base resolution, larger for touch targets) — a button that's only as big as its label text is easy to miss-click, especially for anything meant to run on a touchscreen.

## Feedback for every interactive element

- Every button needs a visible state change on hover AND on press (Godot's `Button` gives you this for free via its theme's hover/pressed styleboxes — don't override them away without replacing them with something equally visible). A button that looks identical whether idle, hovered, or pressed gives the player no confirmation their input registered.
- A value that changes over time and matters to the player (health, score, ammo, a timer) should be visually represented continuously (a bar, a number that updates live), not something the player has to open a separate menu to check — if it's important enough to track, it's important enough to always be visible during the moment it matters.

## Menu default focus & keyboard/controller navigation

- Every menu needs an initial focused control set explicitly (`grab_focus()` on the intended default button when the menu opens) — without this, keyboard/controller navigation has no starting point and the player is stuck unable to navigate the menu at all without a mouse.
- Use `focus_neighbor_*` properties (or Godot's automatic focus-neighbor detection for simple grid/list layouts) so arrow-key/D-pad navigation between buttons goes in the visually obvious direction — don't leave this to chance on anything more complex than a single vertical button stack.

## Don't ship Godot's bare default theme for anything player-facing

- At minimum, set a project-wide `Theme` resource with an intentional font, button style, and color palette that matches the game's tone — the completely unstyled default Godot UI (grey boxes, default font) is an immediate, unmistakable "unfinished prototype" signal to a player, even if every mechanic underneath it is solid.
