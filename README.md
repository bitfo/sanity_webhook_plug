<!-- badges -->

[![Hex.pm Version](http://img.shields.io/hexpm/v/sanity_webhook_plug.svg)](https://hex.pm/packages/sanity_webhook_plug)
[![Hex docs](http://img.shields.io/badge/hex.pm-docs-blue.svg?style=flat)](https://hexdocs.pm/sanity_webhook_plug)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE.md)

# Sanity Webhook Plug

You're reading the main branch's readme. Please visit
[hexdocs](https://hexdocs.pm/sanity_webhook_plug) for the latest published documentation.

<!-- MDOC !-->

SanityWebhookPlug is a Plug that verifies Sanity webhooks for your Elixir Plug
application.

## Installation

```elixir
def deps do
  [
    {:sanity_webhook_plug, "~> 0.1.0"}
  ]
end
```

## Usage

Use this plug in your endpoint:

```elixir
# If using Plug or Phoenix, place before `plug Plug.Parsers`
# For Phoenix apps, in lib/my_app_web/endpoint.ex:
plug SanityWebhookPlug,
  paths: ["/webhooks/sanity/bust_cache"],
  halt_on_error: true,
  secret: {MyApp.Config, :get, [:sanity_webhook_secret]}
```

You may alternatively configure the secret in config, which will be read during
runtime:

```elixir
# in config/runtime.exs
config :sanity_webhook_plug,
  secret: System.get_env("SANITY_WEBHOOK_SECRET")
```

Verifying the signature requires reading the body, but its best to do this
before _interpreting_ the body into JSON or other parsed formats. Plug can
protect your system by limiting how much of body to read to prevent exhaustion.
Ideally, any of these settings you have for `Plug.Parsers` in your endpoint, you
should also have here for SanityWebhookPlug.

By default, errors will be handled by the plug by responding with a 500 error
and a error message.

**Handle Errors Yourself**

If you want to handle errors yourself, you may configure the plug to
`halt_on_error: false` and handle the error yourself. If an error occurs, you
can get it with `SanityWebhookPlug.get_error(conn)` and handle it yourself.

For example, here's a controller that checks the error:

```elixir
# assuming you have a route setup in your router to land in this controller.
def MyAppWeb.SanityController do
  require Logger

  def my_action(conn, params) do
    # ... do your normal thing
  end

  def action(conn, _) do
    if message = SanityWebhookPlug.get_error(conn) do
      Logger.error("Sanity Webhook error: " <> message)
      Plug.Conn.resp(conn, 500, "Error: " <> message)
    else
      apply(__MODULE__, action_name(conn), [conn, conn.params])
    end
  end
end
```
