# Flutter Widget-Level Pitfalls

Widget-construction mistakes that compile fine but cause real runtime bugs — layout overflow, stale state, or a widget that silently doesn't update when it should. These are below the level of "app architecture" (already covered by the contract/plan) and above the level of "syntax" (already covered by `flutter analyze`) — the gap those two checks don't reach.

## Layout overflow

- A `Row`/`Column` whose children can exceed the available space needs `Expanded`/`Flexible` on the growable child, or the OTHER children need a fixed/intrinsic size — never leave an unconstrained `Text` or `Row` inside another `Row` without one of these.
- `ListView`/`GridView` inside a `Column` needs `Expanded` (or `shrinkWrap: true` if it's genuinely meant to size to its content) — a bare unconstrained scrollable inside a `Column` throws at runtime, not at analyze time.
- Prefer `Expanded`/`Flexible` over a hardcoded pixel width/height for anything that should adapt to screen size — a fixed size that works on the design's assumed screen size will overflow on a smaller one.

## State and rebuilds

- `setState` must only be called on state that actually changed — calling it unconditionally on every event handler causes unnecessary full-subtree rebuilds; scope state to the smallest `StatefulWidget` that needs it, not the whole screen.
- A `TextEditingController`/`AnimationController`/`FocusNode` created in a `State` class MUST be disposed in `dispose()` — a missing `dispose()` call is a real memory leak, not a style nit.
- Never call `setState` (or anything that touches `BuildContext`) after an `await` without first checking `if (!mounted) return;` — the widget may have been disposed while the async call was in flight, and using its context/calling setState after that throws.
- Use `const` constructors for every widget whose constructor arguments are themselves all compile-time constants — this is a real, measurable rebuild-cost optimization, not just a lint preference; apply it by default, not only when the linter flags it.

## Lists and keys

- Every widget in a list built from a dynamic collection (`ListView.builder`, `.map()` into a `Column`'s children, etc.) needs a stable `Key` (`ValueKey`/`ObjectKey`) tied to the underlying data's identity, not its list index — an index-based or missing key causes Flutter to misattribute state (e.g. a `TextField`'s content) to the wrong item after a reorder/insert/delete.

## Async + BuildContext

- Don't use `BuildContext` across an `await` boundary without a `mounted` check first (see above) — this includes `Navigator.of(context)`, `ScaffoldMessenger.of(context)`, `Theme.of(context)`, all of it.
- Capture anything derived FROM `context` (like `ScaffoldMessenger.of(context)`) BEFORE the `await`, not after — calling `.of(context)` after an await is exactly the pattern that throws once the widget's gone.

## Common widget-specific traps

- `Image.network`/`Image.asset` needs an `errorBuilder` for anything user-facing — an unhandled image load failure shows Flutter's default red error box, not a graceful fallback.
- `Form`/`TextFormField` validation should use a `GlobalKey<FormState>` and call `.currentState!.validate()` explicitly on submit — don't rely on `onChanged`-triggered validation alone for a submit-time check.
- A `Scaffold`'s `body` that itself contains another `Scaffold` (e.g. nesting a full-screen widget inside another screen) is almost always wrong — use `Navigator.push` for a new screen instead of nesting Scaffolds.
