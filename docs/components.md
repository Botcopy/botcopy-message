# Components

Field-level reference for each component, plus the shared [actions](#actions) every interactive element can use. Every field is marked **Required** or **Optional**; the [global instructions](../global-instructions.xml) enforce the required ones, because the schema alone cannot.

For a working payload of each, see the [Cookbook](./cookbook.md). To start from scratch, see the [README](../README.md).

## Table of Contents

- [Components](#components)
  - [Table of Contents](#table-of-contents)
  - [text](#text)
  - [suggestions](#suggestions)
  - [card](#card)
  - [carousel](#carousel)
  - [list](#list)
  - [form](#form)
  - [Actions](#actions)

---

## text

One or more message bubbles, given as an array so a single entry can produce several bubbles in sequence. Markdown renders inside a bubble, and `\n` starts a new line within one.

| Field | Type | Required | Notes |
| :--- | :--- | :--- | :--- |
| `displayText` | string | **Required** | The text shown in the bubble |
| `ssml` | string | Optional | SSML markup for spoken output |
| `textToSpeech` | string | Optional | Phrase spoken aloud, taking priority over `ssml` |

> [↑ Back to Table of Contents](#table-of-contents)

---

## suggestions

Quick-reply chips shown under the reply. An array of chips, each with:

| Field | Type | Required | Notes |
| :--- | :--- | :--- | :--- |
| `title` | string | **Required** | Label shown on the chip |
| `action` | object | **Required** | One [action](#actions): `message`, `link`, or `location` |

Chips from earlier turns leave the screen once a new reply arrives, so write each reply as though its chips are the only ones visible.

> [↑ Back to Table of Contents](#table-of-contents)

---

## card

A single rich card.

| Field | Type | Required | Notes |
| :--- | :--- | :--- | :--- |
| `title` | string | Optional | Heading shown on the card |
| `subtitle` | string | Optional | Secondary line under the title |
| `body` | string | Optional | Body text |
| `image` | object | Optional | `url` **required** inside it, `alt` optional but always worth setting |
| `action` | object | Optional | `buttons` only. A card action cannot be a `message` or `link` |

On a card, the only action type is `buttons`, so anything a card offers the visitor is offered through a button rather than on the card itself.

> [↑ Back to Table of Contents](#table-of-contents)

---

## carousel

A swipeable row of cards, given as an array. Each item takes the same fields as [`card`](#card), with one difference: an item's `action` can be `buttons`, `message`, `link`, or `location`.

> [↑ Back to Table of Contents](#table-of-contents)

---

## list

A titled, selectable list.

| Field | Type | Required | Notes |
| :--- | :--- | :--- | :--- |
| `title` | string | Optional | Heading shown above the list |
| `items` | array | **Required** | The rows, in display order |

Each row takes `title`, `body`, `image`, and `action`, all optional, though a row with no `action` renders but cannot be selected, which reads to a visitor as a broken list. Row actions are `message`, `link`, or `location`. **Rows cannot carry buttons**, so reach for [`carousel`](#carousel) where each option has an image or needs its own buttons, and keep `list` for plain selectable rows.

> [↑ Back to Table of Contents](#table-of-contents)

---

## form

An inline form. Submitting it sends the `message` action.

| Field | Type | Required | Notes |
| :--- | :--- | :--- | :--- |
| `fields` | array | **Required** | The inputs, in display order |
| `action` | object | **Required** | A `message` action only |
| `title` | string | Optional | Heading at the top of the form |
| `subtitle` | string | Optional | Secondary line under the title |
| `style` | string | Optional | `message` or `wall` |
| `force` | boolean | Optional | Removes the cancel button so the form must be completed |

Each field is described by `fieldType`, which selects the kind of control:

| Field | Type | Required | Notes |
| :--- | :--- | :--- | :--- |
| `fieldType` | string | **Required** | `text`, `select`, `radio`, or `checkbox` |
| `label` | string | **Required** | Shown beside or above the control |
| `parameter` | string | **Required** | The variable that receives the value |
| `type` | string | **Required for `text`** | Narrows what the browser accepts and offers |
| `select` | array | **Required for `select`** | The options offered |
| `radio` | array | **Required for `radio`** | The options offered |
| `checkboxes` | array | **Required for `checkbox`** | The options offered |
| `groupLabel` | string | Optional | Heading above a group of choices |
| `required` | boolean | Optional | Defaults to false, so state it explicitly either way |
| `multiline` | boolean | Optional | Turns a `text` field into a multi-line input |
| `rows` | number | Optional | Height of a multi-line input |
| `placeholder`, `error`, `helperText`, `maxLength`, `pattern`, `autocomplete` | string or number | Optional | Refine the input experience |

Two of those are easy to trip over. **`multiline` is a boolean on a `text` field, not a value of `fieldType`.** And the options array is named after the control rather than being shared: a `select` field carries `select`, a `radio` field carries `radio`, and a `checkbox` field carries `checkboxes`.

For a `text` field, `type` is one of `text`, `email`, `tel`, `number`, `password`, `url`, `date`, `datetime-local`, `month`, `week`, `time`, `color`, or `checkbox`.

Each entry in an options array carries `selectLabel`, the text the visitor sees, and `parameter`, the value written when the visitor picks that entry. So `parameter` means two different things depending on where it sits: on a field, `parameter` names the variable that receives the value, and inside an option it is the value that variable receives. The [form recipe](./cookbook.md#collect-details-with-a-form) shows both in one payload.

> [↑ Back to Table of Contents](#table-of-contents)

---

## Actions

Every interactive element carries exactly one action, and which actions are available depends on where the element sits.

| Action | Fields | Available on |
| :--- | :--- | :--- |
| `message` | `command` and `type` **required**, `type` is `training` or `event`; `parameters` optional | [`suggestions`](#suggestions), [`carousel`](#carousel) items, [`list`](#list) rows, [`form`](#form) submit, and buttons |
| `link` | `url` **required**; `target` and `parameters` optional | [`suggestions`](#suggestions), [`carousel`](#carousel) items, [`list`](#list) rows, and buttons |
| `buttons` | array of `title` plus one action each | [`card`](#card) and [`carousel`](#carousel) items |
| `location` | `parameter`, `success`, and `error` **required**; `latitude`, `longitude`, and `errorCode` optional | [`suggestions`](#suggestions), [`carousel`](#carousel) items, [`list`](#list) rows, and buttons |

A `message` action's `type` decides how Botcopy treats `command` when the visitor acts. Use `training` to send the text as though the visitor had typed it, so your agent matches it the same way it matches anything else. Use `event` to send a named event instead, which suits a signal your agent should act on without the words appearing in the conversation.

A `location` action asks the browser for the visitor's coordinates and writes them to the variable named in `parameter`. That variable receives **one object**, and `latitude` and `longitude` name keys inside it rather than variables of their own, so declare the receiving variable as a [Custom Object](../README.md#choosing-a-type). Its `success` and `error` fields are both required, and each holds a further action: what the chat does once the browser answers, either way. A location request the visitor declines is a normal outcome rather than an edge case, which is why the schema insists you handle it.

> [↑ Back to Table of Contents](#table-of-contents)
