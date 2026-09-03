# AGENTS.md

This repository holds what a Google CX Agent Studio agent needs in order to reply with Botcopy chat UI. Botcopy renders a structured payload, and that payload is composed at runtime by the CX Agent Studio agent rather than written ahead of time. The work here is configuration: setting that agent up so it composes a valid payload on every turn.

Copy from this repository and do not change it. Anyone maintaining it rather than building against it should ask their Botcopy contact.

## Files

| File | Contents | How to use it |
| :--- | :--- | :--- |
| `README.md` | Setup, variables, instruction authoring | Read from start to finish, check back for updates |
| `botcopy_message.json` | The widget schema | Paste whole, see Setup |
| `global-instructions.xml` | Rules the CX Agent Studio agent follows at runtime | Paste whole, see Setup |
| `docs/components.md` | Every field of every component | Look up |
| `docs/cookbook.md` | Payloads captured from a running agent | Look up |
| `docs/patterns.md` | Wrapping Dialogflow flows | Only if Dialogflow is still in play |

## Setup

Three settings decide whether rich replies work at all. `README.md` carries the console navigation for each one.

1. Set the application's model to the most capable Text Optimized option. The console default does not reliably fill the widget, and reports no error when it fails.
2. Paste `botcopy_message.json` into the Custom widget's Parameters field, whole and unmodified.
3. Paste `global-instructions.xml` into the application's global instructions, whole and unmodified.

`global-instructions.xml` is read by the CX Agent Studio agent at runtime rather than by you, so copy it across instead of following its composition rules yourself.

## Boundaries

- Never hand-edit `botcopy_message.json` or `global-instructions.xml`. Replace an older copy in full rather than patching it, and if either file looks wrong, tell your Botcopy contact instead of correcting it locally.
- Never compose a Botcopy payload by hand. A payload written here is not the payload the CX Agent Studio agent will emit at runtime, so it proves nothing about the configuration.
- Anything a visitor is meant to pick has to be a `suggestions`, `list`, `card`, or `carousel` component. Choices written as text in a message bubble are not selectable.

## Verify

Check every new reply shape in the CX Agent Studio simulator before you point a Botcopy bot at it. A malformed payload renders nothing and raises no error, so an untested shape looks the same as a working one right up until a visitor reaches it.
