<!-- SPDX-License-Identifier: LGPL-3.0-only -->
<!-- SPDX-FileCopyrightText: Copyright (C) 2026 mplx <mplx@jennifer-lang.dev> -->

# Design

## Scope

The deck posts messages to a Slack™ channel through an Incoming Webhook:
plain text with `send`, and rich messages built from Block Kit™ blocks
(`header`, `section`, `fieldsSection`, `contextBlock`, `actionsBlock` with
`button`, `divider`) starting from `message` and posted with `sendMessage`.
`render` returns the JSON payload without sending it, and `escape` makes
text from elsewhere safe for mrkdwn.

It is deliberately not a Slack client:

- **No Web API.** No bot or user tokens, no `chat.postMessage`, no reading
  channels, no files, no threads. An Incoming Webhook is the smallest thing
  that lets a program notify a channel, and it needs no OAuth app install
  beyond creating the webhook.
- **No interactivity.** Buttons are URL buttons only; handling block actions
  needs a request URL and a server, which is another program.
- **Only the common blocks.** Images, inputs, select menus and rich-text
  blocks are left out; the six blocks cover notifications.
- **No client-side enforcement of Slack's limits** (block count, text
  lengths). Slack answers an invalid payload with a `4xx`, which reaches the
  caller as the response.

## Decisions

- **A deck, not a bundled module.** A client for one vendor's service belongs
  in the deck ecosystem. The deck is one of the bundled modules that
  `jennifer` moves out (M30.1 in its milestones, formerly M28.32).
- **Same surface as the module.** The types and functions are those of the
  standard library's `slack` module, so migrating changes only the import.
  Two additions differ: `escape`, and the webhook URL check.
- **The response is returned, not judged.** `send` and `sendMessage` return
  the `http.Response`. Slack answers `200` with the body `ok`; a revoked or
  unknown webhook is a `4xx` with a body such as `no_service`, which the
  caller reads. A notifier usually logs a failure and carries on.
- **Immutable builder of pre-rendered blocks.** A `Message` holds the fallback
  `text` and a list of block JSON fragments; every step returns a new
  message. Rendering each block when it is added keeps blocks of different
  shapes in one homogeneous list. Every string goes through `json.encode`.
- **Fallback text is optional but advised.** Slack shows `text` in
  notifications and in clients that do not render blocks; `render` leaves out
  empty parts.
- **Escaping is explicit.** Slack's mrkdwn turns `<...>` into links and
  mentions (`<!channel>`, `<@U123>`) wherever the text came from. `escape`
  replaces `&`, `<` and `>` with entities; it is a separate call so that a
  message meant to link or mention keeps working as it did in the module.

## Errors

The deck throws `Error{kind: "slack"}` for one check of its own: a webhook
URL that is not `https://`, or `http://` to a loopback host, thrown before
anything is sent, with the URL left out of the message.

Slack's answers are responses, not errors, so the deck has no error struct
(as recorded for the `@jennifer` decks in M28.57). Transport failures come
from `http` unchanged: `"timeout"`, `"closed"`, `"limit"`, and on
interpreters with transport error kinds (M28.65) a `net.NetError` of kind
`"net"` for a refused connection, reset, failed lookup or TLS failure (older
ones throw those as `"runtime"`).

## Security and limits

- **The webhook URL is the credential.** Anyone holding it can post to the
  channel. The docs tell callers to read it from the environment or a config
  file.
- **No cleartext sends.** `http` sends a plain `http://` URL to any host in
  the clear, so the deck refuses anything but `https://`. `http://` is allowed
  only to `127.0.0.1`, `localhost` and `[::1]`, for a local test server. The
  host is taken from the authority after any `user@` part, so
  `http://127.0.0.1@evil.example/` is refused.
- **Untrusted text** goes through `escape` before it reaches a mrkdwn block;
  JSON encoding keeps it from changing the payload's structure. Button URLs
  are passed as given.
- Response size, deadlines and redirects are `http`'s defaults; the deck adds
  no caps of its own and no retries.

## Testing and validation

`src/slack_test.j` tests every block's rendered JSON (escaping included),
`escape`, and sends through an `httpd` server on a loopback port that
answers like a webhook: `200 ok` on the expected path, `404 no_service`
otherwise. It also checks that every malformed or plain-`http://` URL throws
before a request goes out.

The tests never reach Slack. `examples/post.j` posts when `SLACK_WEBHOOK_URL`
is set and only prints its payload otherwise, so CI runs it offline.

## Known limitations and open questions

- Rate limits (`429` with `Retry-After`) are left to the caller; the deck
  does not retry.
- Webhooks cannot reply in threads or update messages; that needs the Web
  API, which is out of scope.

## Dependencies on the interpreter

- The floor is `>=0.27.0`, for the UTF-8 codec of `encoding.decode` and
  `testing.assertThrowsArgs`. 0.26.0 deprecates the bundled module (M30.1)
  and its `http` refuses a URL without a valid scheme.
- `http` needs the `net` capability, so posting needs the default `jennifer`
  binary; building and rendering messages also work on `jennifer-tiny`.
- The tests use `httpd` and `task` for the loopback server.
