# Cookbook

Each recipe pairs the instruction you write with the payload an agent produced from that instruction. Every payload here was captured from a running agent rather than written from the schema, with the one exception noted under [Ask for the visitor's location](#ask-for-the-visitors-location).

All of them assume the [global instructions](../global-instructions.xml) are in place, so none repeat the payload rules. The agent these came from closed each reply with a chip back to its menu; those chips are omitted so each recipe shows only its own component.

Field-by-field detail lives in [Components](./components.md). To set the widget up from scratch, see the [README](../README.md).

## Table of Contents

- [Cookbook](#cookbook)
  - [Table of Contents](#table-of-contents)
  - [Offer a short set of choices](#offer-a-short-set-of-choices)
  - [Send text, with Markdown](#send-text-with-markdown)
  - [Show one item](#show-one-item)
  - [Show several items to browse](#show-several-items-to-browse)
  - [Let the visitor pick from a list](#let-the-visitor-pick-from-a-list)
  - [Collect details with a form](#collect-details-with-a-form)
  - [Ask for the visitor's location](#ask-for-the-visitors-location)

## Offer a short set of choices

Chips suit a small, closed set of options and save the visitor from typing. Put them last so the text that introduces them appears first.

```xml
<action>Call {@Widget: botcopy_message} with one text component saying each chip
  renders one component, followed by suggestion chips titled "Text", "Card", and
  "List". Give each chip an action containing a message object, with command set
  to the chip title and type set to "training".</action>
```

```json
{
  "botcopy": [
    { "text": [ { "displayText": "Each chip renders one component." } ] },
    {
      "suggestions": [
        { "title": "Text", "action": { "message": { "command": "Text", "type": "training" } } },
        { "title": "Card", "action": { "message": { "command": "Card", "type": "training" } } },
        { "title": "List", "action": { "message": { "command": "List", "type": "training" } } }
      ]
    }
  ],
  "type": "custom_template"
}
```

> [↑ Back to Table of Contents](#table-of-contents)

---

## Send text, with Markdown

One `text` entry holding several objects produces several bubbles in sequence. Markdown renders inside a bubble, and `\n` starts a new line within one.

```xml
<action>Call {@Widget: botcopy_message} with one text component holding two
  bubbles. The first says "This is one message bubble." The second demonstrates
  bold, italic, code, a link, and a three-item list.</action>
```

The bullet list in the second bubble is a Markdown formatting demonstration. Anything the visitor is meant to pick has to be a `suggestions`, `list`, `card`, or `carousel` component rather than lines of text in a bubble.

```json
{
  "botcopy": [
    {
      "text": [
        { "displayText": "This is one message bubble." },
        { "displayText": "Markdown in a bubble: **bold**, _italic_, `code`, a [link](https://www.botcopy.com), and a list:\n- first item\n- second item\n- third item" }
      ]
    }
  ],
  "type": "custom_template"
}
```

> [↑ Back to Table of Contents](#table-of-contents)

---

## Show one item

A [`card`](./components.md#card) presents a single item. Its only action is `buttons`, so whatever the card offers is offered through a button.

```xml
<action>Call {@Widget: botcopy_message} with a card titled "Trail Runner 2",
  subtitle "Road and trail", body "Lightweight, built for uneven ground.", an
  image with alt text, and a buttons action holding one button titled "View
  details" whose action is a link to the product page.</action>
```

```json
{
  "botcopy": [
    {
      "card": {
        "title": "Trail Runner 2",
        "subtitle": "Road and trail",
        "body": "Lightweight, built for uneven ground.",
        "image": { "url": "https://placehold.co/600x400/png", "alt": "Trail Runner 2 shoe" },
        "action": { "buttons": [ { "title": "View details", "action": { "link": { "url": "https://example.com/p/trail-runner-2" } } } ] }
      }
    }
  ],
  "type": "custom_template"
}
```

Set `alt` on every image so the reply stays usable with a screen reader.

> [↑ Back to Table of Contents](#table-of-contents)

---

## Show several items to browse

A [`carousel`](./components.md#carousel) is an array of the same shape. Reach for it when each option has an image worth showing or needs its own buttons; reach for [`list`](./components.md#list) when the options are plain text rows, since list rows cannot carry buttons.

```xml
<action>Call {@Widget: botcopy_message} with a carousel of the matching products.
  Give each item a title, a subtitle, a one-line body, an image with alt text,
  and a buttons action holding "View details" as a link to that product's
  page.</action>
```

```json
{
  "botcopy": [
    {
      "carousel": [
        {
          "title": "Trail Runner 2",
          "subtitle": "Road and trail",
          "body": "Lightweight, built for uneven ground.",
          "image": { "url": "https://placehold.co/600x400/png", "alt": "Trail Runner 2 shoe" },
          "action": { "buttons": [ { "title": "View details", "action": { "link": { "url": "https://example.com/p/trail-runner-2" } } } ] }
        },
        {
          "title": "City Walker",
          "subtitle": "Casual wear",
          "body": "Comfortable everyday walking shoe.",
          "image": { "url": "https://placehold.co/600x400/png", "alt": "City Walker shoe" },
          "action": { "buttons": [ { "title": "View details", "action": { "link": { "url": "https://example.com/p/city-walker" } } } ] }
        }
      ]
    }
  ],
  "type": "custom_template"
}
```

> [↑ Back to Table of Contents](#table-of-contents)

---

## Let the visitor pick from a list

A [`list`](./components.md#list) row carries one action and cannot hold buttons. A `training` command sends the row's text as though the visitor had typed it, so your agent matches it like any other input.

```xml
<action>Call {@Widget: botcopy_message} with a list titled "Choose a topic"
  holding three rows, each with a one-line body and an action containing a
  message object, with command set to the row title and type set to
  "training".</action>
```

```json
{
  "botcopy": [
    {
      "list": {
        "title": "Choose a topic",
        "items": [
          { "title": "Delivery times", "body": "View standard shipping schedules.", "action": { "message": { "command": "Delivery times", "type": "training" } } },
          { "title": "Returns policy", "body": "Check return windows and eligibility.", "action": { "message": { "command": "Returns policy", "type": "training" } } },
          { "title": "Size guide", "body": "Find your perfect fit.", "action": { "message": { "command": "Size guide", "type": "training" } } }
        ]
      }
    }
  ],
  "type": "custom_template"
}
```

Give every row an action. A row without one renders but cannot be selected.

> [↑ Back to Table of Contents](#table-of-contents)

---

## Collect details with a form

A [`form`](./components.md#form) gathers several values in one exchange. Ask only for what you need now, and name each `parameter` after a variable you have already [declared](../README.md#declaring-a-variable).

```xml
<action>Call {@Widget: botcopy_message} with a form titled "Start a return"
  holding three fields: a required text field "Order number" as order_number, a
  required email field "Email" as contact_email, and an optional select field
  labelled "Reason" offering "Wrong size", "Damaged", and "Changed my mind", each
  option writing to contact_reason. Use command "return started" and type "event"
  for the form action.</action>
```

```json
{
  "botcopy": [
    {
      "form": {
        "title": "Start a return",
        "fields": [
          { "fieldType": "text", "type": "text", "label": "Order number", "parameter": "order_number", "required": true },
          { "fieldType": "text", "type": "email", "label": "Email", "parameter": "contact_email", "required": true },
          {
            "fieldType": "select",
            "label": "Select a reason",
            "groupLabel": "Reason",
            "parameter": "contact_reason",
            "required": false,
            "select": [
              { "selectLabel": "Wrong size", "parameter": "Wrong size" },
              { "selectLabel": "Damaged", "parameter": "Damaged" },
              { "selectLabel": "Changed my mind", "parameter": "Changed my mind" }
            ]
          }
        ],
        "action": { "message": { "command": "return started", "type": "event" } }
      }
    }
  ],
  "type": "custom_template"
}
```

Two details in that payload are easy to get wrong. A `text` field sets both `fieldType` and `type`, while a `select` field sets `fieldType` alone. The second is that `parameter` means two different things: on a field, `parameter` names the variable that receives the value, while inside a `select` entry it is the value that variable receives when the visitor picks that option.

Submitted values arrive on the next turn under the variable each field names. Because a submission writes **every** field the form collects, including the ones left blank, offer a separate smaller form where visitors need to correct one detail later.

> [↑ Back to Table of Contents](#table-of-contents)

---

## Ask for the visitor's location

A `location` action asks the browser for the visitor's coordinates. It is an action rather than a component, so it rides on a chip, a carousel item, or a list row. Explain why you are asking in the text before it, since the browser will prompt for permission.

```xml
<action>Call {@Widget: botcopy_message} with one text component explaining that
  sharing a location finds the nearest branch, followed by a suggestion chip
  titled "Share my location" whose action is a location action storing the result
  in visitor_location. Give it a success action sending the event "location
  shared" and an error action sending the event "location declined"; both are
  required.</action>
```

```json
{
  "botcopy": [
    { "text": [ { "displayText": "I can find your closest branch if you share your location." } ] },
    {
      "suggestions": [
        {
          "title": "Share my location",
          "action": {
            "location": {
              "parameter": "visitor_location",
              "latitude": { "parameter": "visitor_latitude" },
              "longitude": { "parameter": "visitor_longitude" },
              "success": { "action": { "message": { "command": "location shared", "type": "event" } } },
              "error": { "action": { "message": { "command": "location declined", "type": "event" } } }
            }
          }
        }
      ]
    }
  ],
  "type": "custom_template"
}
```

> [!NOTE]
>
> This is the one payload here built from the schema rather than captured from a run, because the browser answers a location request outside the reply. The shape matches the schema, but confirm it in the [simulator](https://docs.cloud.google.com/gemini-enterprise-cx/cx-agent-studio/simulator) before relying on it.

Declare `visitor_location` as a [Custom Object](../README.md#choosing-a-type). It receives one object whose keys are the names you gave under `latitude` and `longitude`, so a shared location arrives as `{ "visitor_latitude": 12.345, "visitor_longitude": -67.890 }`.

The `success` and `error` actions are both required. A visitor declining the browser prompt is a normal outcome rather than an edge case, so give the `error` branch somewhere useful to go: a `location declined` event lets your agent follow up by asking for a postcode instead.

> [↑ Back to Table of Contents](#table-of-contents)
