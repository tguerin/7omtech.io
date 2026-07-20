---
title: "Type-safe Rive, and letting the agent do the wiring"
subtitle: "Rive ViewModel Generator: stop hand-writing string lookups against your state machines, and hand your coding agent a contract it can't hallucinate."
date: 2026-07-20 08:00:00 +0100
categories: [Flutter Programming, Tooling]
image: /images/flutter-logo.jpeg
tags:
    - Flutter
    - Rive
    - Code Generation
    - AI Agents
    - CLI
---

If you read my [Mowgli post](https://7omtech.fr/2025/12/mowgli/), you saw this
little snippet buried in the animation section:

```dart
_viewModel // This is a viewmodel from Rive using databinding
  ..colorLevel = animation.type.bountyLevel.toDouble()
  ..triggerStart();
```

It looks clean. It reads like a proper API. But I lied to you by omission.
Because behind that `_viewModel`, the real code was closer to this:

```dart
_viewModel.number('colorLevel')!.value = animation.type.bountyLevel.toDouble();
_viewModel.trigger('triggerStart')!.trigger();
```

Magic strings. Everywhere. `'colorLevel'`, `'triggerStart'`, `'orientation'`,
`'isFolded'`… On a poker table with **dozens of Rive files**, each exposing its
own set of numbers, colors, booleans, enums and triggers, that's hundreds of
string keys scattered across the codebase.

And strings, as we all know, are where dreams go to die.

## The dirty secret of Rive data binding

Don't get me wrong: Rive data binding is fantastic. Motion designers build a
view model in the editor, expose the properties they want, and hand you a `.riv`
file. On the code side you just read and write those properties, and the whole
animation reacts. It's genuinely one of my favorite things about working with
Rive.

But the contract between the designer's file and your Dart code is… a bunch of
strings. And that means:

- **No autocomplete.** You have to open Rive (or dig through a doc nobody
  updated) to remember whether it's `colorLevel` or `color_level`.
- **No compile-time safety.** `_viewModel.number('colorLvl')` compiles perfectly
  fine and then crashes at runtime with a lovely null-check exception. Ask me how
  I know.
- **Refactoring hell.** The designer renames a property, and you find out in
  production, not in your IDE.
- **Type guessing.** Is that property a `number` or an `enum`? Get it wrong and,
  again, runtime crash.

At some point I realized we were treating the view model like an untyped bag of
keys, when it's really **a typed API waiting to be generated**. Rive already
knows every property, its type, its enums, its nested view models. That
information is right there in the file. So why on earth were we transcribing it
by hand?

So I built a tool to stop doing that.

## Rive ViewModel Generator

The idea is dead simple: **point it at a `.riv` file, get a typed Dart wrapper
back.** Every property becomes a real getter and setter, every enum becomes a
Dart `enum`, every nested view model becomes its own typed class, and you get a
reactive `Stream` per property for free.

There are two ways to use it, and they produce **identical output** because they
render from the exact same Mustache templates:

1. A **drag-and-drop web app**: throw one or more `.riv` files on it, download
   the generated Dart.
2. A **CLI** (`rive-gen`) for scripting and CI, and (as we'll see) for your
   coding agent.

You can play with the web version right here:

<iframe width="100%" height="600" src="https://tguerin.github.io/rive-viewmodel-generator/"></iframe>

## What the generated code looks like

Remember that stringly-typed mess? Here's what the generator turns it into. For
a number property, you get a getter, a setter, and a broadcast stream:

```dart
double get numberProperty => _viewModel.number('numberProperty')!.value;

set numberProperty(double value) =>
    _viewModel.number('numberProperty')!.value = value;

Stream<double> get numberPropertyStream { /* broadcast stream over the property */ }
```

The `'numberProperty'` string is now written **exactly once**, by a machine,
inside the generated file you never touch. Your app code just does:

```dart
final viewModel = TestViewModel.fromViewModel(stateMachine.viewModel('Test'));

// Getters and setters, fully typed
viewModel.orientation = Orientation.portrait;

// Reactive streams
viewModel.orientationStream.listen((orientation) {
  print('Orientation changed to: $orientation');
});

// Always dispose when done
viewModel.dispose();
```

`Orientation` there is a real Dart enum the generator extracted from the Rive
file. Autocomplete works. Rename detection works. And if the designer removes a
property, your project stops compiling in your IDE, not in front of your users.

It covers everything I actually hit on Mowgli:

- **Every property type**: boolean, number, string, color, trigger, enum, and
  nested view models.
- **Artboards & state machines**: typed accessors for both.
- **Named instances (presets)**: if your view model declares named instances in
  the editor, you get a typed enum plus `fromInstance` / `fromDefaultInstance`
  factories, so presets stop being string lookups too.
- **Lifecycle**: a `dispose()` that closes every stream controller. No leaks.

## The part I actually want to talk about: agents

Here's where it gets fun.

I didn't build a fancy MCP server. I didn't build a bespoke "AI integration".
I built a **plain, boring CLI**, just one command and a handful of flags:

```bash
npm install -g @rive-viewmodel/cli

rive-gen --input assets/hero.riv --output lib/generated
```

And it turns out that "plain and boring" is exactly what makes it perfect for a
coding agent like Claude Code. An agent lives in the terminal. A CLI is a
terminal command. There's nothing to teach it, no protocol to negotiate. It
just runs the thing.

The workflow I use now looks like this. The motion designer drops a fresh
`hero.riv` in the assets folder. I tell my agent:

> "The designer updated `hero.riv`. Regenerate its view model and wire the new
> `bountyLevel` property into `KoCenterWidget`".

And the agent:

1. Runs `rive-gen -i assets/hero.riv -o lib/generated --modern --interface`.
2. **Reads the generated Dart**, which now spells out, in types, every property
   the file exposes.
3. Writes the binding code against those types.
4. Runs the analyzer. If it typo'd a property, **it doesn't compile**, the agent
   sees the error, and fixes itself in the loop.

That last point is the whole thing. Think about what happens *without* the
generator: the agent has to guess the string key. It writes
`_viewModel.number('bountyLvl')`, that compiles fine, and the bug ships. LLMs are
*spectacularly* good at inventing plausible-looking string keys that don't
exist. Stringly-typed APIs are basically hallucination traps.

The generated view model flips that. It turns the designer's file into a
**machine-readable contract**. A wrong property name becomes a compile error
instead of a runtime crash. And a compile error is something the agent's own
toolchain catches and self-corrects, no human in the loop. You're not asking the
agent to be careful; you've made carelessness impossible to commit.

I've said it before and I'll say it again: **code generation isn't just for
humans anymore.** The types you generate are the guardrails your agent runs
inside.

## Wiring it into CI (and your agent's habits)

Because it's a CLI, it drops straight into whatever automation you already have.
A one-liner in a script, a step in CI to regenerate on every `.riv` change, a
pre-build hook. Pick your poison. My favourite is just telling the agent, once,
in the project's instructions: *"whenever a `.riv` file changes under `assets/`,
run `rive-gen` before touching any animation code".* After that it's automatic.

A couple of flags worth knowing:

```bash
# Legacy rive_native import (this is the default)
rive-gen -i assets/hero.riv -o lib/generated

# Modern package:rive import (0.14+)
rive-gen -i assets/hero.riv -o lib/generated --modern

# Modern import + implement the RiveViewModel interface
rive-gen -i assets/hero.riv -o lib/generated --modern --interface
```

Small heads-up: the web app and the CLI ship with **different defaults** (the web
app leans modern + interface on, the CLI leans legacy + interface off), so pass
the flags that match your setup rather than trusting the default. There's also an
optional [`rive_viewmodel`](https://pub.dev/packages/rive_viewmodel) pub package
that provides the `RiveViewModel` interface if you want your generated classes to
share a common type.

## What about Rive's own generator?

Fair question, and I want to be upfront about it. The Rive team is building their
own [`rive-code-generator`](https://github.com/rive-app/rive-code-generator-wip),
and if you've read my stuff you already know I'm a big fan of what they do (I said
it in the Mowgli post and I'll happily repeat it: Rive rocks). It's honestly funny
how much our two approaches rhyme. Both parse the `.riv`, both render the output
through **Mustache templates**. Great minds, or at least similarly lazy ones.

For the record, this wasn't me copying their homework. Rive's generator has been
around since late 2024, but for a long time it targeted the older world of state
machine inputs and text runs, not data binding. I started my little view model
generator back in June 2025, and Rive only landed data binding support in their own
generator at the end of October 2025, a few months later. We just both stared at the
same gap and reached for the same tool (hello again, Mustache).

Theirs is actually broader in scope than mine. It's built to be language-agnostic
through those templates (Dart and JSON out of the box), and it does things mine
doesn't: JSON representations of the file, diff files for version control, and
asset extraction with CDN metadata. If your needs go beyond Flutter, that's the
one to keep an eye on.

Where mine differs today is Flutter ergonomics. It leans hard into how a Flutter
dev actually wants to consume a view model: a reactive `Stream` per property, a
`dispose()` that cleans everything up, typed `fromInstance` factories for named
presets, the optional `RiveViewModel` interface, and output for both the legacy
`rive_native` and the modern `package:rive` imports. Add the drag-and-drop web app
and the `npm` CLI, and there's no native binary to build first.

Now the honest part: theirs is the **official** one, and the repo name still ends
in `-wip` for a reason. It's explicitly experimental for now. Once it ships properly
it'll be a great option, and well worth watching, if only because a generator
maintained by the Rive team is going to track the runtime more closely than anything
I keep alive on the side.

So I'm not going to tell you to drop one for the other. If you like how my tool
generates your view models, the per-property streams, the `dispose()`, the typed
factories, the Flutter-first feel, then use it. If the official one fits your
workflow better when it lands, use that instead. Two tools solving the same problem
is a good problem to have, and honestly it just means the itch was real.

## Try it

Everything is open source under a BSD-3 license:

- **Live demo:** [tguerin.github.io/rive-viewmodel-generator](https://tguerin.github.io/rive-viewmodel-generator)
- **CLI on npm:** [`@rive-viewmodel/cli`](https://www.npmjs.com/package/@rive-viewmodel/cli)
- **Source & issues:** [github.com/tguerin/rive-viewmodel-generator](https://github.com/tguerin/rive-viewmodel-generator)

If you're using Rive data binding in Flutter, do yourself a favour and stop
hand-writing those string lookups. And if you're pairing with a coding agent,
generate the types first. It's the cheapest way I know to keep the thing honest.

As always, the tool is young and I'd love your feedback. If a property type isn't
handled the way you'd expect, or you want another target language out of those
templates, open an issue or a PR. I read all of them.

**Type-safely yours,**\
_Thomas_
