# Flutter Animation & Motion Craft

Small, well-placed motion is one of the highest-leverage ways to make an app feel polished rather than static — and one of the easiest to get wrong (overused, janky, or blocking interaction).

## Prefer implicit animations for simple state changes

- For a single property changing in response to state (a container resizing, a color changing, an opacity fading) use the implicit animation widgets — `AnimatedContainer`, `AnimatedOpacity`, `AnimatedCrossFade`, `AnimatedSwitcher` — instead of manually building an `AnimationController` + `Tween` + `AddListener` setup. Implicit animations need no controller lifecycle management (no risk of forgetting to dispose one) and are almost always enough for a UI-level transition.
- Reach for an explicit `AnimationController` only when you need something implicit animations can't do: precise control over timing/curves mid-flight, chaining/staggering multiple animations together, or repeating/reversing on a custom trigger.

## Every explicit AnimationController must be disposed

- An `AnimationController` created in a `State` class's `initState` MUST be disposed in `dispose()` (`controller.dispose()`), and its `vsync` should be the `State` itself via `SingleTickerProviderStateMixin` (single controller) or `TickerProviderStateMixin` (multiple) — a leaked controller keeps ticking after its widget is gone, which is both a real memory/CPU leak and a common source of "setState called after dispose" crashes.

## Page transitions

- Use `Navigator.push` with a real transition (`MaterialPageRoute` already provides a platform-appropriate one; a custom `PageRouteBuilder` for anything bespoke) rather than instantly swapping screens with no transition — an app where every screen change is an abrupt cut reads as noticeably less polished than one with even Flutter's default transition.
- Use `Hero` widgets for an element that visually continues from one screen to the next (a thumbnail becoming a full image, a list item becoming a detail-view header) — this single technique is one of the highest-impact, easiest to implement "feels professionally made" cues Flutter offers, and is easy to skip by default because it takes an explicit `Hero(tag: ..., child: ...)` on both screens.

## Curves, duration, and restraint

- Default to `Curves.easeInOut` (or `Curves.easeOut` for something entering, `Curves.easeIn` for something exiting) rather than `Curves.linear` — linear motion reads as mechanical/robotic; eased motion reads as natural, at effectively zero extra cost.
- Keep UI-transition durations short: roughly 150-300ms for most widget-level transitions, up to ~400-500ms for a full page transition. Longer than that makes the app feel sluggish and makes the user wait on a decoration rather than actual content.
- Don't animate everything — a screen where every single element independently fades/slides/scales in competes for attention and reads as busier and slower than a screen with one or two well-chosen animated elements and the rest appearing instantly. Motion should draw attention to what changed, not decorate every pixel.

## Never let an animation block interaction

- A loading spinner or transition animation should never be the ONLY way the user learns something is happening while also preventing them from doing anything else for longer than necessary — if an action can complete in the background, let the UI stay interactive (or show a lightweight inline indicator) rather than a full-screen blocking animation for routine operations.
