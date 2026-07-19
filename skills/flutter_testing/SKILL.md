# Flutter Testing Craft

Tests that actually catch regressions, without wasting effort testing things Flutter/Dart already guarantee.

## What's worth testing in a generated app

- Widget tests for anything with real logic: form validation, conditional rendering based on state, a list that filters/sorts, navigation triggered by user action. A widget test that only asserts "the widget builds without throwing" and checks nothing about its actual behavior is close to worthless — assert on the OUTCOME of an interaction (tap a button, expect a specific text/state change), not just that construction succeeded.
- Unit tests for any non-trivial pure logic living outside widgets (a price calculator, a validator function, a data-transformation method) — these are cheaper to write and run than widget tests and should be preferred whenever the logic doesn't actually need a widget tree to exercise it.
- Skip testing framework behavior Flutter itself already guarantees (a `Text` widget displays its string, a `Container`'s color property is applied) — that's testing Flutter, not the app.

## Widget test structure

- Use `testWidgets` with `WidgetTester`, wrap the widget under test in a `MaterialApp`/`Scaffold` (most widgets need an ancestor `Directionality`/`MediaQuery`/theme to render correctly in a test), and use `tester.pumpAndSettle()` after an interaction that triggers an animation, or plain `tester.pump()` if `pumpAndSettle` would hang on an intentionally-infinite animation (a loading spinner).
- Find widgets by `find.byType`, `find.text`, or (for anything that could reasonably match multiple widgets, or where the widget's exact type/text is likely to change) `find.byKey` with an explicit `Key` assigned in the widget itself — a test built entirely on `find.text('Submit')` breaks the moment that button's copy changes for an unrelated reason.

## Async and state in tests

- Await every `tester.tap()`/`tester.enterText()` that triggers async work, and call `await tester.pump()` (or `pumpAndSettle()`) afterward before asserting — asserting immediately after an interaction that starts a `Future` will observe the PRE-completion state, not the result, and produce a flaky or simply wrong test.
- Mock external dependencies (network calls, platform channels) rather than letting a widget test make a real network request — a test suite that depends on network availability is not a reliable regression check, and slows down the whole suite for no benefit specific to the widget being tested.

## Don't over-test a TRIVIAL app

- For a genuinely small/TRIVIAL app (a counter, a single-screen utility), one or two widget tests covering the actual interactive behavior is proportionate — writing an extensive test suite for an app this size is disproportionate effort for the risk being mitigated, the same "don't add process where nothing is gained" reasoning Creator already applies to TRIVIAL's Execute stage.
