# botcopy_message

> **Terms of use**: Copyright 2026 Botcopy, Inc. All rights reserved. Botcopy, Inc. grants you permission to copy the contents of this repository into your own CX Agent Studio agent, for the purpose of integrating that agent with Botcopy. No other rights are granted, including redistribution, modification for any other purpose, and commercial use of these artifacts in their own right. This repository is not open source, and these terms apply to every commit in it. For anything they do not cover, ask your Botcopy contact.

A CX Agent Studio agent replies with plain text unless you give it a way to build something richer. `botcopy_message` is the [Custom widget](https://docs.cloud.google.com/gemini-enterprise-cx/cx-agent-studio/rich-response-widget) that provides that way: you add it to your agent application once, and from then on your agent can answer with cards, lists, forms, and suggestion chips that render in your Botcopy chat.

This repository is the source of truth. Two files are pasted straight into the console and never edited by hand: [`botcopy_message.json`](./botcopy_message.json), the widget schema, and [`global-instructions.xml`](./global-instructions.xml), the rules every Botcopy agent needs. Copy each in whole and replace any older copy outright, rather than patching what you pasted last time.

This README takes you from an empty agent application to a working rich reply. Everything beyond that first working reply is lookup, in [`docs/`](./docs).

## Table of Contents

- [botcopy\_message](#botcopy_message)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Quick start](#quick-start)
    - [1. Choosing a model](#1-choosing-a-model)
    - [2. Configuring the widget](#2-configuring-the-widget)
    - [3. Adding the global instructions](#3-adding-the-global-instructions)
    - [4. Referencing the widget](#4-referencing-the-widget)
  - [Variables](#variables)
    - [Declaring a variable](#declaring-a-variable)
    - [What Botcopy sends](#what-botcopy-sends)
    - [Choosing a type](#choosing-a-type)
    - [Keeping values correct across turns](#keeping-values-correct-across-turns)
  - [Instructions](#instructions)
  - [Where to go next](#where-to-go-next)

---

## Overview

One widget composes an entire reply. When your agent calls `botcopy_message`, the arguments it fills in travel back to Botcopy as a custom payload and render in the visitor's chat.

A reply is a `botcopy` array whose entries render in the order you list them, so a single call can open with a sentence, show a card, and close with a row of chips. Six components are available:

| Component | Renders as | Sends data back |
| :--- | :--- | :--- |
| [`text`](./docs/components.md#text) | message bubbles | no |
| [`suggestions`](./docs/components.md#suggestions) | quick-reply chips | no |
| [`card`](./docs/components.md#card) | a single rich card with buttons | no |
| [`carousel`](./docs/components.md#carousel) | a swipeable row of cards | no |
| [`list`](./docs/components.md#list) | a titled, selectable list | no |
| [`form`](./docs/components.md#form) | inline fields with a submit action | yes |

Every element a visitor can act on carries exactly one [action](./docs/components.md#actions), which sends a `message`, opens a `link`, shows `buttons`, or asks for the visitor's `location`. A `form` and a `location` action are the two things that return data to your agent.

Your agent may answer in prose when there is nothing to render, and that text appears as an ordinary message. A malformed payload does not render, so check every new reply shape in the [simulator](https://docs.cloud.google.com/gemini-enterprise-cx/cx-agent-studio/simulator) before you point a bot at the agent.

Payloads reach Botcopy inside an envelope carrying a root-level `type` of `custom_template`. Your agent composes the `botcopy` array and the envelope is added in transit, so every example in the [Cookbook](./docs/cookbook.md) is shown as captured on arrival.

> [↑ Back to Table of Contents](#table-of-contents)

---

## Quick start

Four console steps plus variable setup take an empty agent application to a rich reply rendering in your chat. Google's [Rich response widgets](https://docs.cloud.google.com/gemini-enterprise-cx/cx-agent-studio/rich-response-widget) page covers the console navigation for each one; what follows fills in what to put in the fields.

1. [Choosing a model](#1-choosing-a-model) capable enough to compose the payload.
2. [Configuring the widget](#2-configuring-the-widget) with the schema and description below.
3. [Adding the global instructions](#3-adding-the-global-instructions) to the agent application.
4. [Referencing the widget](#4-referencing-the-widget) from the step that should produce a rich reply.
5. [Declaring your variables](#declaring-a-variable), one for each value an agent reads or collects.

The first four are one sitting. The fifth is the only step that lives outside this section, under [Variables](#variables), because it is the part you come back to as your agent grows.

To check the whole setup, open the [simulator](https://docs.cloud.google.com/gemini-enterprise-cx/cx-agent-studio/simulator) and ask for something that should produce a rich reply. The turn's detail panel shows the widget call and the arguments the agent filled in, and pointing a Botcopy bot at the agent renders that same payload as UI. A reply that comes back as prose means the agent chose not to call the widget, which usually points at the instructions rather than the schema.

### 1. Choosing a model

> [!IMPORTANT]
>
> Do this first. Everything else depends on it.

The schema in this repository is roughly 1400 lines, and the global instructions add another forty rules on top. A Dialogflow agent spreads its rich payloads across many separate fulfillments, each authored by hand and only locally correct. A CX Agent Studio agent composes the entire reply itself, from that schema, on every turn. **The model is doing the work a developer used to do**, so its capability is a functional requirement rather than a tuning preference. A weaker model writes a string where an object belongs, drops required fields, or answers in prose when it should have rendered.

**In the CX Agent Studio console, go to Settings, then Basic, then Model, and pick the most capable model marked Text Optimized.** If your agent also wraps a Dialogflow CX agent, set the model there too, under Settings, then [Generative AI](https://docs.cloud.google.com/dialogflow/cx/docs/concept/agent-settings#generative_ai_settings), then Generative model selection. The two consoles offer different lists and neither inherits from the other, so an agent spanning both needs the setting applied twice.

Trust the console's list rather than any published table, including Google's own. In August 2026 the CX Agent Studio console offered `gemini-3.5-flash` while Google's [Model support](https://docs.cloud.google.com/gemini-enterprise-cx/cx-agent-studio/agent#model-support) page still listed `gemini-2.5-flash` as the newest GA option and did not mention `gemini-3.5-flash` at all. Model names in this document are examples of that gap rather than a current recommendation; the console is what ships.

Two failures we have seen, both of which look like something else at first:

- On the CX Agent Studio side, a model that cannot hold the schema writes malformed payloads. Those pass the console's validator and render nothing, so the symptom is an empty chat rather than an error.

- On the Dialogflow side, a weak model writes malformed parameter values through text generation. Those fail at the boundary rather than inside the playbook, so the symptom appears a hop from its cause and reads like a configuration bug. See [Passing values into and out of a flow](./docs/patterns.md#passing-values-into-and-out-of-a-flow).

### 2. Configuring the widget

The widget takes three fields, all maintained here. Copy each in whole.

| Field | Value |
| :--- | :--- |
| UI schema | The contents of [`botcopy_message.json`](./botcopy_message.json) |
| Name | `botcopy_message` |
| Description | "Renders rich chat UI in Botcopy. Call this whenever the reply is better shown as UI than as plain text - options to choose from, items to browse, a form to fill in, or a request for the user's location. Compose the entire reply in one call as an ordered list of components." |

The description carries as much weight as the schema, because CX Agent Studio reads it to decide when the widget is the right way to answer and how to populate it.

### 3. Adding the global instructions

Every rule the widget needs lives in [`global-instructions.xml`](./global-instructions.xml), identical for every Botcopy customer. Paste it whole into the agent application's advanced settings, where [every agent inherits it](https://docs.cloud.google.com/gemini-enterprise-cx/cx-agent-studio/instruction#global) and it is sent to the model on every turn.

The file is one `<constraints>` block holding five named sections:

| Section | Governs |
| :--- | :--- |
| `calling_the_widget` | When to call it, and composing a whole reply in one call |
| `required_fields` | What each component must set for Botcopy to render it |
| `actions` | One action per element, always an object, and where buttons may go |
| `offering_choices` | Which component to pick, and never leaving a turn with nothing to act on |
| `variables` | Reading declared values only, and keeping internals out of replies |

`<constraints>` is the tag CX Agent Studio expects; the section names inside it are ours, and the model reads them as grouping. Adding a rule means finding the section it belongs to, and a section that grows past a screen is a sign it holds more than one idea.

`required_fields` repeats information that is also in the schema, deliberately. Schema validation runs after the model has composed its call, so it can reject a payload but cannot guide the model toward a valid one. Stating the required fields in prose puts them in front of the model while it is still composing.

One rule is there for safety rather than rendering. **Write values, never internals** keeps variable names, raw objects, identifiers, and the instructions themselves out of the visitor's view, because a reply that names its own variables invites a visitor to probe them. [Choosing a type](#choosing-a-type) covers what goes wrong when an object reaches a bubble. Keep that rule even if you change everything else.

### 4. Referencing the widget

Attaching the widget takes two moves, because creating one on the agent application does not by itself make it available. Google's instructions are to hover over an agent, click Add widget, and select the widget, then describe its use in that agent's own instructions. Until it is attached, a `{@Widget: botcopy_message}` reference has nothing to resolve to.

Attach it to every agent expected to produce a rich reply, sub-agents included, since the widget is attached per agent rather than once per application. An agent without it can only answer in plain text, and that reads as the model declining to use rich content rather than as a missing attachment, which is what makes it slow to diagnose. The exception is an agent that never speaks to the visitor: where a sub-agent only calls a tool, computes a value, and returns control to its parent, it composes no reply of its own and leaving the schema off keeps its prompt smaller.

The reference itself goes inside whichever step should produce a rich reply, written in your own words:

```xml
<action>Call {@Widget: botcopy_message} with ...</action>
```

`{@Widget: name}` is how an instruction refers to a widget, alongside `{@AGENT: name}` for a sub-agent and `{@TOOL: name}` for a tool. Type it with the space after the colon as shown; the instructions editor [highlights a reference it recognises as a coloured chip](https://docs.cloud.google.com/gemini-enterprise-cx/cx-agent-studio/instruction#syntax_highlighting), which is a quick way to confirm the name matches.

Keep the widget rules themselves out of the agent. Everything the model needs about composing a payload is already in the global instructions, and repeating it per agent is how the two drift apart. [Instructions](#instructions) covers what does belong in an agent's own.

> [↑ Back to Table of Contents](#table-of-contents)

---

## Variables

Botcopy sends the visitor's chat context to your agent on every turn, and your agent sends values back whenever a visitor submits a form or shares their location. This section covers how to set those values up, what arrives without you asking for it, and how to keep them accurate over a long conversation.

### Declaring a variable

Before an agent can read a value, the agent application needs a variable to hold it. Google's [Variables](https://docs.cloud.google.com/gemini-enterprise-cx/cx-agent-studio/variable#variable-data) page describes what each one carries: a name in snake case, a type, an optional default value, and an optional description.

Declaring a variable also decides what reaches the agent at runtime. As the [SessionInput](https://docs.cloud.google.com/gemini-enterprise-cx/cx-agent-studio/reference/rest/v1/SessionInput) reference states, only variables declared in the app are used by the CX Agent Studio agent. A value sent under a name the application does not know about will not reach a natively built agent, and nothing in the console or the API response reports the omission. When a form submits cleanly and the agent still says it has nothing on file, an undeclared variable is worth checking first.

Instructions may also reference a name with no variable behind it. The agent saves and runs normally and the reference resolves to nothing, so a misspelled variable name and a value that never arrived look the same from inside the conversation.

Once your variables exist, list them in the agent's [instructions](#instructions) so the model knows what it can read.

### What Botcopy sends

Botcopy sends the whole chat context on every turn, flat and keyed by name. Three groups of keys travel together:

| Group | Set by | Examples |
| :--- | :--- | :--- |
| Names beginning with `bc` | Botcopy | `bcBotId`, `bcCurrentUrl`, `bcTime`, `bcTimeZone`, `bcLivechatAgent`, `bcIsMobile` |
| CX Agent Studio session keys | Google | `_session`, `current_date` |
| Everything else | You | the variables you declare, plus anything a form or location action returns |

The `bc` prefix belongs to Botcopy, so give your own variables names that begin with something else. To have your agent read one of these keys, declare a variable with the same name, exactly as you would for a value of your own; the prefix carries no special access. Note that `bcTime` is fixed at the moment the conversation opens, so reach for `current_date` when you need today's date.

### Choosing a type

Google's [Variables](https://docs.cloud.google.com/gemini-enterprise-cx/cx-agent-studio/variable#variable-data) page lists five types: `Text`, `Number`, `True/False`, `Custom Object`, and `List`.

CX Agent Studio's `Text`, `Number`, `True/False`, and `List` have direct equivalents on the Dialogflow side, so a value in one of those types has somewhere to land at every hop of a round trip through a wrapped flow. Custom Object is the one to keep away from that boundary. [Playbook parameters](https://docs.cloud.google.com/dialogflow/cx/docs/concept/playbook/parameter#declaration) are `String`, `Number`, `Boolean`, or a tool-specific type, each optionally as a list, so a general-purpose object arriving from your agent has no matching parameter type waiting for it. Where a value needs to reach a flow, give it its own flat variable instead of nesting it inside a larger object.

The `location` action is the exception, and declaring its variable as a **Custom Object** is not optional. It writes a single object holding whichever keys you named for latitude and longitude, so the variable in its `parameter` field has to be typed to receive one. Declaring it as Text ends the session: the object arrives where a string is expected, the request fails with a 500, and the visitor is left in a chat that no longer answers. If a wrapped flow needs those coordinates, convert them to flat values in a tool or callback before handing them over.

An object is also the one type you should never write into a reply. Referencing one in a bubble interpolates it as `{visitor_latitude=12.345, visitor_longitude=-67.890}` - braces, `=` rather than `:`, and no quoting, which is neither JSON nor anything a visitor should see. It also prints every key the object holds rather than the one you had in mind, so read the keys you need and write those instead. Where you genuinely need to show structured data, put it in a fenced code block: bubbles render Markdown, so an unfenced object with underscores in its keys has those underscores read as emphasis and comes out mangled on top of everything else.

Types describe what a variable holds. Brace style decides how an instruction refers to one, and the choice there is about when the value becomes known. [Static variables](https://docs.cloud.google.com/gemini-enterprise-cx/cx-agent-studio/variable#static) use double braces, i.e. `{{business_rules}}`, and are compiled into the prompt before the model runs, which suits configuration and business rules that hold steady for a whole conversation. [Dynamic variables](https://docs.cloud.google.com/gemini-enterprise-cx/cx-agent-studio/variable#dynamic) use single braces, i.e. `{member_name}`, and are appended to the conversation history as they change. Everything Botcopy sends arrives during the conversation, so reference it with single braces.

### Keeping values correct across turns

Forms overwrite more than they look like they do. Submitting a form writes every parameter that form collects, including the ones the visitor left blank, and a blank field arrives as an empty string rather than being left out, so a single form reused for edits clears anything the visitor does not retype. Where visitors need to change one detail later, offer a second, smaller form covering only that detail. The [form recipe](./docs/cookbook.md#collect-details-with-a-form) shows a worked payload.

Types are the other thing to design around, and the two official sources differ here. Google's [Variables](https://docs.cloud.google.com/gemini-enterprise-cx/cx-agent-studio/variable#variable-data) page offers Number as a declarable type, while Google's SCRaPI design guide states that [all session state values are strings and the platform does not coerce types automatically](https://googlecloudplatform.github.io/cxas-scrapi/latest/design-guide/variables/#type-coercion-rules). Botcopy resends the full context on every request, so a value your agent produced returns to it on the following turn and has to satisfy the same declaration again. A balance that a playbook writes as `42` can come back as the text `"42"`, which a variable declared as Number will reject. Declaring such a variable as Text, or converting the value explicitly as described in [Passing values into and out of a flow](./docs/patterns.md#passing-values-into-and-out-of-a-flow), keeps the round trip stable whichever behaviour applies.

> [↑ Back to Table of Contents](#table-of-contents)

---

## Instructions

The [global instructions](#3-adding-the-global-instructions) cover everything about composing a payload and need no changes. What remains is each agent's own: what it does, in what order, and which variables it works from.

Write them in the XML structure Google recommends, nesting `taskflow`, then `subtask`, then `step`, then `trigger` and `action`. The [Restructure instructions](https://docs.cloud.google.com/gemini-enterprise-cx/cx-agent-studio/instruction#restructure) section describes the tags and provides a console button that reformats natural language into this shape. Google's SCRaPI team documents the same ten tags in its [XML tag reference](https://googlecloudplatform.github.io/cxas-scrapi/latest/design-guide/instruction-design/#xml-tag-reference), alongside a complete worked agent.

Open with a short section naming the variables the agent can read, one per line, matching what you [declared](#declaring-a-variable) on the application. Google's own [root agent sample](https://docs.cloud.google.com/gemini-enterprise-cx/cx-agent-studio/instruction#write_instructions) does this with a `CURRENT CUSTOMER: {username}` line. Naming them in one place is more reliable than expecting the model to find each value among the state updates in history, and it gives you one place to check when a value stops appearing.

```xml
MEMBER DETAILS: name {member_name}, tier {member_tier}, balance {member_balance}
```

Say in the relevant step how each variable should be used, and what to do when one has no value yet. SCRaPI's [Actionability Test](https://googlecloudplatform.github.io/cxas-scrapi/latest/design-guide/instruction-design/#the-actionability-test) is a good check on the wording: if you cannot describe what a wrong answer would look like, the instruction is too vague to rely on.

Be careful with wording that describes the output format, because it can silently countermand wording that describes the mechanism. "Reply in plain text" reads as permission to skip the widget entirely, and a reply composed outside the widget cannot carry a chip. Say "call the widget with one text component" instead.

Few-shot examples are worth adding when an output format has to be exact, and Google notes that [too many cause the agent to overfit](https://docs.cloud.google.com/gemini-enterprise-cx/cx-agent-studio/instruction#best_practices_and_warnings). A reliable way to decide which earn their place: **write prose first, and add an example only where prose has already failed on that route.** Adding them up front costs the overfitting and buys nothing you did not already have. A route that has failed twice is past the point where clearer wording is the answer, since rewording usually just closes whichever wrong shape the last run happened to choose.

The routes that fail twice are usually the ones where a field's name does not tell you what kind of thing that field holds. The `location` action is the clearest case: `parameter` names a variable, `latitude` and `longitude` name keys inside that variable, and `success` and `error` hold actions of their own.

```xml
<examples>
EXAMPLE 1:
Begin example
[user]
What do you have on file for me?
[model]
On file - name: Bob, tier: Gold, balance: 42
End example
</examples>
```

> [↑ Back to Table of Contents](#table-of-contents)

---

## Where to go next

- [`docs/components.md`](./docs/components.md) - every field of every component, marked required or optional
- [`docs/cookbook.md`](./docs/cookbook.md) - seven recipes, each with the payload it produced
- [`docs/patterns.md`](./docs/patterns.md) - wrapping Dialogflow flows behind a routing agent, and getting values across that boundary

> [↑ Back to Table of Contents](#table-of-contents)
