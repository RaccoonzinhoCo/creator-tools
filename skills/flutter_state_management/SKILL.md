# Flutter State Management Craft

Choosing and applying a state management approach well enough that the app doesn't accumulate bugs from state living in the wrong place or updating unpredictably.

## Choosing an approach (don't default to the most complex option)

- For a genuinely small/TRIVIAL app (a handful of screens, state that doesn't need to cross more than 1-2 widget levels): plain `StatefulWidget` + `setState` is correct and sufficient. Reaching for Provider/Riverpod/Bloc on an app this small adds ceremony with no payoff — more files, more boilerplate, nothing gained.
- For a LIGHT/FULL app with state shared across multiple unrelated screens (auth state, a shopping cart, user preferences, theme): use `Provider`/`ChangeNotifierProvider` (simplest to reason about, least boilerplate) unless the contract specifically calls for Riverpod (compile-time-safe DI, better for larger apps) or Bloc (explicit event/state separation, best when the team/spec wants an audit trail of every state transition). Don't introduce Bloc's event-sourcing ceremony for an app that doesn't need it.
- Whatever is chosen, use it CONSISTENTLY across the whole app — a codebase mixing `setState` in some screens, `Provider` in others, and a singleton service accessed directly elsewhere is harder to reason about than any single approach used uniformly, even if that single approach isn't the "best" one for every individual screen.

## Scope state to where it's actually needed

- State that only one widget subtree cares about belongs in THAT subtree's own `StatefulWidget`/provider scope, not hoisted to app-wide global state "just in case" — global state that's actually local-scoped state in disguise causes unrelated widgets to rebuild on changes they don't care about, and makes the actual data flow harder to trace.
- A `ChangeNotifier`/state class should expose METHODS that mutate its own state internally (`cart.addItem(item)`), not raw mutable fields a widget reaches into and mutates directly (`cart.items.add(item)` from outside) — the latter means nothing reliably calls `notifyListeners()`, so dependent widgets silently don't rebuild.

## Common state bugs to avoid

- Don't call `notifyListeners()` (or `setState`) synchronously from inside a widget's `build()` method or from another widget's `didChangeDependencies` in a way that triggers immediately during the current build pass — this throws a "setState called during build" error. Defer with `WidgetsBinding.instance.addPostFrameCallback` if a state change genuinely needs to happen in response to a build.
- A `Provider`/state object that holds a `Future` or a loading flag needs an explicit initial/loading/error/data state (an enum or a sealed class with those cases), not just a nullable data field — `data == null` is ambiguous between "still loading" and "loaded, but genuinely empty," and the UI needs to distinguish those to avoid showing an empty-state message while a request is still in flight.
- Derived/computed values (a cart's total price, a filtered list) should be computed FROM the source state on read, not stored as a separately-maintained field that has to be kept in sync by every mutation — a manually-synced derived field is a recurring source of "the total didn't update" bugs the moment one mutation path forgets to recompute it.
