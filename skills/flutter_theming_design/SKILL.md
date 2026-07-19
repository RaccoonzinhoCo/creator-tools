# Flutter Theming & Visual Design Craft

The difference between an app that looks like it used Flutter's raw defaults everywhere and one that reads as deliberately designed — without needing a designer's mockup to work from.

## Always define a real ThemeData, never rely on Flutter's defaults

- Every app needs an explicit `ThemeData`/`ColorScheme` (via `ColorScheme.fromSeed(seedColor: ...)` at minimum, for Material 3) set on `MaterialApp.theme` — an app left on Flutter's completely unstyled defaults (default purple, default fonts, default everything) is an immediate "unfinished/generic" signal, even when every screen's functionality is correct.
- Derive colors for custom widgets FROM the theme (`Theme.of(context).colorScheme.primary`, `.secondary`, `.surface`, etc.), not hardcoded `Color(0xFF...)` literals scattered through widget code — hardcoded colors are how an app ends up with 6 slightly-different shades of the "same" blue, and make a future theme/dark-mode change require hunting down every literal instead of changing one seed color.
- Support dark mode by default (`ThemeData` + `darkTheme` + `themeMode: ThemeMode.system`) unless the spec explicitly says otherwise — this is close to a baseline user expectation now, and `ColorScheme.fromSeed` with `Brightness.dark` gets most of the work done for free.

## Consistent spacing and typography scale

- Use a small, fixed set of spacing values (e.g. 4/8/16/24/32) applied consistently via `SizedBox`/`Padding`/`EdgeInsets`, not arbitrary one-off numbers picked per widget — inconsistent spacing (12 here, 15 there, 20 somewhere else) is subtle but reads as visually "off" even to someone who can't articulate why.
- Use `Theme.of(context).textTheme` (`headlineMedium`, `titleLarge`, `bodyMedium`, etc.) for text styling instead of ad hoc `TextStyle(fontSize: 16, fontWeight: FontWeight.w500)` literals repeated across the app — this keeps the whole app's typography consistent and makes a font-scale change a one-place edit.

## Depth, hierarchy, and visual polish

- Use `Card`/`Material` elevation, subtle shadows, or a slightly different surface color (not just borders) to distinguish layered content (a card on a background, a bottom sheet over content) — flat, borderless, same-color-everywhere layouts are a common "looks like a wireframe, not a finished app" tell.
- Give interactive elements (buttons, list tiles, cards that respond to tap) a visible pressed/hover state — Material widgets provide this via `InkWell`/`ElevatedButton` automatically; don't wrap tappable content in a bare `GestureDetector` with no visual feedback on press, which makes the app feel unresponsive even when the tap handler works correctly.
- Icons and images need consistent sizing within their context (all list-tile leading icons the same size, all card thumbnails the same aspect ratio) — mismatched sizes within what should be a repeated/uniform pattern (a list, a grid) is one of the most visually noticeable inconsistencies in an otherwise-correct layout.

## Empty, loading, and error states are part of the design, not an afterthought

- Every screen that loads data needs a designed empty state (not just a blank screen) and a designed error state (not just Flutter's raw exception text) — these are usually the first states a user or reviewer actually sees (an app almost always starts with "nothing loaded yet"), so leaving them unstyled undermines the polish of the rest of the app.
