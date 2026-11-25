# @deckyard/slack

[![test](https://github.com/deckyard/deck-slack/actions/workflows/test.yml/badge.svg)](https://github.com/deckyard/deck-slack/actions/workflows/test.yml)
[![Documentation](https://img.shields.io/badge/docs-github%20pages-blue)](https://deckyard.github.io/deck-slack/)
[![release](https://img.shields.io/github/v/release/deckyard/deck-slack)](https://github.com/deckyard/deck-slack/releases/latest)

Posts messages to a Slack™ channel through an Incoming Webhook from a
[Jennifer](https://jennifer-lang.dev/) program, as plain text or as Block Kit™
blocks. Replaces the `slack` module of Jennifer's standard library (shipped
since 0.18.0, deprecated from 0.26.0, removed in 0.28.0); the API is
unchanged, only the import differs.

Up to 0.1.0 this deck was `@jennifer/slack`. To switch, change the `[decks]`
entry to `"@deckyard/slack" = "^0.2.0"` and the imports to `"@deckyard/slack/"`.

**[Documentation](https://deckyard.github.io/deck-slack/)**

## Install

```sh
jvc add @deckyard/slack
```

This vendors the deck to `vendor/deckyard/slack/`. Without `jvc`, unpack the
`src/` subtree of a release archive (`deckyard-slack-<version>.tar.gz`) there.

## Usage

```jennifer
import "@deckyard/slack/" as slack;
import "http.j" as http;

def url as string init os.getEnv("SLACK_WEBHOOK_URL");
def r as http.Response init slack.send($url, "deploy finished");
io.printf("%d\n", $r.status);   # 200 with body "ok"; a bad webhook is 4xx, not an error
```

`examples/post.j` posts a plain and a Block Kit message.

## Requirements

Jennifer 0.27.0 or newer and network access to `hooks.slack.com`. Posts need
the default `jennifer` binary; the stock `jennifer-tiny` has no network driver.

## Development

```sh
tools/check-all.sh                   # the full gate, as CI runs it
jennifer test src/slack_test.j       # the test overlay
tools/check-docs.sh                  # docs against the code
tools/pack.sh dist                   # build the release archive
```

## Licence

LGPL-3.0-only, see [LICENSE](LICENSE). Copyright (C) 2026 mplx &lt;mplx@jennifer-lang.dev&gt;.

`tools/check-headers.sh` checks every source file's SPDX header against
`deck.toml`.

## Trademarks

Slack™ is a trademark of Slack Technologies, LLC. Block Kit™ and other product
and company names may be trademarks of their respective owners. They are used
only to identify the products this deck works with. This deck is an
independent project, not affiliated with, sponsored or endorsed by Slack
Technologies, LLC.
