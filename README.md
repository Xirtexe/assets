# Zoe-XD Plugin Development

Complete documentation for creating, developing, and distributing plugins for **Zoe-XD**.

Zoe-XD uses a lightweight command registration system that allows developers to create commands and event-based handlers without modifying the bot's core source code.

Plugins can be installed locally or distributed as external JavaScript files through a URL.

---

## Table of Contents

- [Overview](#overview)
- [Requirements](#requirements)
- [Plugin Structure](#plugin-structure)
- [Creating Your First Plugin](#creating-your-first-plugin)
- [Command Configuration](#command-configuration)
- [Command Properties](#command-properties)
- [Pattern Matching](#pattern-matching)
- [Aliases](#aliases)
- [Permissions](#permissions)
- [Command Handler](#command-handler)
- [The `match` Argument](#the-match-argument)
- [The `client` Argument](#the-client-argument)
- [Message Object](#message-object)
- [Sending Replies](#sending-replies)
- [Editing Messages](#editing-messages)
- [Quoted Messages](#quoted-messages)
- [Downloading Quoted Media](#downloading-quoted-media)
- [Groups and Private Chats](#groups-and-private-chats)
- [User Information](#user-information)
- [Event-Based Plugins](#event-based-plugins)
- [Event Types](#event-types)
- [Complete Command Examples](#complete-command-examples)
- [External Plugins](#external-plugins)
- [Publishing an External Plugin](#publishing-an-external-plugin)
- [Plugin Naming Rules](#plugin-naming-rules)
- [Dependencies](#dependencies)
- [Error Handling](#error-handling)
- [Best Practices](#best-practices)
- [Security](#security)
- [Troubleshooting](#troubleshooting)

---

# Overview

A Zoe plugin is a JavaScript file that registers one or more commands with Zoe.

The basic structure is:

```js
Zoe(
  {
    pattern: "ping",
    fromMe: false,
    desc: "Check bot latency.",
    type: "info"
  },
  async (m, match, client) => {
    await m.reply("Pong!");
  }
);
```

Once the plugin is loaded, users can execute the command using the configured bot prefix:

```text
<prefix>ping
```

For example, if the prefix is `/`:

```text
/ping
```

Zoe automatically detects the command and calls its handler.

---

# Requirements

A Zoe plugin is simply JavaScript.

You should have:

- Basic JavaScript knowledge
- Node.js knowledge
- Understanding of asynchronous JavaScript
- Understanding of `async/await`
- Basic knowledge of WhatsApp/Baileys if your plugin performs advanced WhatsApp operations

Plugins run inside the same Node.js process as Zoe.

---

# Plugin Structure

A plugin normally contains:

```text
plugin-name.js
```

Example:

```text
plugins/
â””â”€â”€ weather.js
```

The plugin registers its commands when the file is loaded.

Example:

```js
Zoe(
  {
    pattern: "weather",
    fromMe: false,
    desc: "Get weather information.",
    type: "utility"
  },
  async (m, match, client) => {
    await m.reply("Weather command executed.");
  }
);
```

A plugin can register multiple commands:

```js
Zoe(
  {
    pattern: "hello",
    fromMe: false,
    desc: "Say hello.",
    type: "fun"
  },
  async (m) => {
    await m.reply("Hello!");
  }
);

Zoe(
  {
    pattern: "bye",
    fromMe: false,
    desc: "Say goodbye.",
    type: "fun"
  },
  async (m) => {
    await m.reply("Goodbye!");
  }
);
```

---

# Creating Your First Plugin

Create a file:

```text
hello.js
```

Add:

```js
Zoe(
  {
    pattern: "hello",
    fromMe: false,
    desc: "Say hello to the bot.",
    type: "fun"
  },
  async (m) => {
    await m.reply("Hello! How can I help you?");
  }
);
```

If the bot prefix is `/`, users can run:

```text
/hello
```

The command handler receives the incoming message through `m`.

---

# Command Configuration

The first argument passed to `Zoe()` is the command configuration object.

General structure:

```js
{
  pattern: "command",
  fromMe: false,
  alias: [],
  desc: "Command description",
  type: "category",
  on: "text"
}
```

Not every property is required for every command.

---

# Command Properties

## `pattern`

Defines the primary command name.

Type:

```js
String
```

Example:

```js
pattern: "ping"
```

The user can execute:

```text
/ping
```

The command matcher compares the incoming text against the configured prefix and command name.

Command matching is case-insensitive.

For example:

```text
/ping
/PING
/Ping
```

all match:

```js
pattern: "ping"
```

### Important

`pattern` is currently expected to be a string.

Do **not** use:

```js
pattern: /ping/i
```

Use:

```js
pattern: "ping"
```

---

# `fromMe`

Controls who can execute the command.

Type:

```js
Boolean | String
```

### Public command

```js
fromMe: false
```

The command can be used by normal users.

Example:

```js
Zoe(
  {
    pattern: "ping",
    fromMe: false,
    desc: "Check bot latency.",
    type: "info"
  },
  async (m) => {
    await m.reply("Pong!");
  }
);
```

### Restricted command

```js
fromMe: true
```

The command is restricted to authorized Zoe sudo users.

Example:

```js
Zoe(
  {
    pattern: "restart",
    fromMe: true,
    desc: "Restart the bot.",
    type: "owner"
  },
  async (m) => {
    await m.reply("Restarting...");
  }
);
```

Zoe checks the message's `isSudo` state before allowing restricted commands.

### Public string

Zoe also recognizes:

```js
fromMe: "public"
```

as a public command.

For normal public commands, prefer:

```js
fromMe: false
```

---

# `alias`

Defines alternative names for a command.

Type:

```js
Array<String>
```

Example:

```js
Zoe(
  {
    pattern: "ping",
    alias: ["p", "pong"],
    fromMe: false,
    desc: "Check bot latency.",
    type: "info"
  },
  async (m) => {
    await m.reply("Pong!");
  }
);
```

The same handler can now respond to:

```text
/ping
/p
/pong
```

Aliases are checked together with the primary `pattern`.

Example:

```js
alias: ["yt", "youtube"]
```

---

# `desc`

A short description of what the command does.

Example:

```js
desc: "Download audio from YouTube."
```

This is useful for command menus, help systems, and documentation.

Keep descriptions short and clear.

Good:

```js
desc: "Download YouTube audio."
```

Bad:

```js
desc: "This command is used to download audio files from YouTube links that users send to the bot."
```

---

# `type`

Defines the command category.

Example:

```js
type: "download"
```

Common categories:

```text
info
fun
admin
owner
group
download
media
utility
tools
search
ai
```

The value is primarily metadata used for organizing and displaying commands.

Example:

```js
type: "media"
```

---

# `on`

`on` allows a plugin to respond to messages/events instead of requiring a command pattern.

Supported values:

```text
all
text
sticker
image
video
audio
delete
```

Zoe checks these values against the incoming message type.

Example:

```js
Zoe(
  {
    on: "image",
    fromMe: false,
    type: "media"
  },
  async (m) => {
    await m.reply("You sent an image.");
  }
);
```

Unlike normal command handlers, event handlers do not require a `pattern`.

---

# Pattern Matching

For normal commands, Zoe builds the command using the configured prefix:

```text
PREFIX + pattern
```

For example:

```js
pattern: "ping"
```

with:

```text
PREFIX = /
```

results in:

```text
/ping
```

Zoe accepts:

```text
/ping
/ping hello
```

For:

```text
/ping hello world
```

the command is:

```text
ping
```

and the remaining text is:

```text
hello world
```

The remaining text is provided to the handler as `match`.

---

# Command Handler

The normal handler signature is:

```js
async (m, match, client) => {
}
```

There are three arguments:

| Argument | Description |
|---|---|
| `m` | Zoe message object |
| `match` | Text after the command |
| `client` | Serialized WhatsApp/Baileys client |

Example:

```js
Zoe(
  {
    pattern: "echo",
    fromMe: false,
    desc: "Repeat supplied text.",
    type: "fun"
  },
  async (m, match, client) => {
    if (!match) {
      return m.reply("Give me some text.");
    }

    await m.reply(match);
  }
);
```

---

# The `match` Argument

`match` contains everything after the command name.

For:

```text
/echo hello world
```

you receive:

```js
match === "hello world"
```

For:

```text
/echo
```

you receive an empty string.

Example:

```js
Zoe(
  {
    pattern: "say",
    fromMe: false,
    desc: "Repeat text.",
    type: "fun"
  },
  async (m, match) => {
    if (!match) {
      return m.reply("Usage: /say <text>");
    }

    await m.reply(match);
  }
);
```

---

# The `client` Argument

The third argument is the active Zoe WhatsApp client.

Example:

```js
Zoe(
  {
    pattern: "myid",
    fromMe: false,
    desc: "Get the current chat ID.",
    type: "info"
  },
  async (m, match, client) => {
    await m.reply(m.jid);
  }
);
```

The client can be used for advanced operations supported by the underlying WhatsApp/Baileys layer.

Example:

```js
async (m, match, client) => {
  await client.sendMessage(
    m.jid,
    {
      text: "Hello from the client!"
    }
  );
}
```

For normal replies, prefer the Zoe message helpers such as:

```js
m.reply(...)
```

rather than directly using the client.

---

# Message Object

The `m` object is Zoe's serialized message object.

Common properties include:

```js
m.chat
m.jid
m.sender
m.pushName
m.text
m.message
m.data
m.type
m.fromMe
m.isGroup
m.isPm
m.isBot
m.sudo
m.isSudo
m.reply_message
m.mention
```

A message instance may look conceptually like:

```js
{
  chat: "...",
  jid: "...",
  sender: "...",
  pushName: "...",
  text: "...",
  message: "...",
  data: {...},
  type: "...",
  fromMe: false,
  isGroup: true,
  isPm: false,
  isBot: false,
  sudo: [],
  isSudo: false
}
```

The exact values depend on the incoming WhatsApp message.

---

# Important Message Properties

## `m.jid`

The JID of the current chat.

Example:

```js
const jid = m.jid;
```

For a group, this will normally be the group JID.

---

## `m.chat`

The current chat identifier.

In normal command usage:

```js
m.chat
```

can be used as the current chat context.

---

## `m.sender`

The sender of the message.

Example:

```js
await m.reply(`Sender: ${m.sender}`);
```

WhatsApp may use LID addressing in some contexts, particularly with newer group addressing behavior. Do not assume every sender identifier is always an `@s.whatsapp.net` JID.

---

## `m.text`

The text content of the incoming message.

Example:

```js
console.log(m.text);
```

For:

```text
/ping hello
```

the message text contains:

```text
/ping hello
```

while the command's `match` contains:

```text
hello
```

---

## `m.data`

The underlying serialized WhatsApp message data.

Use this when a lower-level Baileys operation requires the original message structure.

For example, when quoting a message through the underlying client:

```js
await client.sendMessage(
  m.jid,
  {
    text: "Reply"
  },
  {
    quoted: m.data
  }
);
```

Use `m.data` for the actual quoted-message object.

---

## `m.isGroup`

Boolean indicating whether the message came from a group.

Example:

```js
if (!m.isGroup) {
  return m.reply("This command can only be used in groups.");
}
```

---

## `m.isPm`

Boolean indicating whether the message came from a private chat.

Example:

```js
if (!m.isPm) {
  return m.reply("Use this command in private chat.");
}
```

---

## `m.isBot`

Boolean indicating whether the sender/message is identified as a bot.

---

## `m.fromMe`

Boolean indicating whether the incoming message was sent by the bot itself.

---

## `m.isSudo`

Boolean indicating whether the sender is an authorized Zoe sudo user.

Example:

```js
if (!m.isSudo) {
  return m.reply("You are not authorized.");
}
```

For most owner/sudo commands, use:

```js
fromMe: true
```

instead of manually checking `m.isSudo`.

---

## `m.sudo`

Contains the sudo information associated with the message.

---

# Sending Replies

The simplest way to respond is:

```js
await m.reply("Hello!");
```

Example:

```js
Zoe(
  {
    pattern: "hello",
    fromMe: false,
    desc: "Say hello.",
    type: "fun"
  },
  async (m) => {
    await m.reply("Hello!");
  }
);
```

You can send dynamic content:

```js
await m.reply(`Hello ${m.pushName}!`);
```

---

# Editing Messages

The return value of `m.reply()` can be used to edit the sent message.

Example:

```js
const msg = await m.reply("Processing...");

await msg.edit("Done!");
```

A common pattern is:

```js
const start = Date.now();

const msg = await m.reply("Checking...");

await msg.edit(
  `Pong!\nLatency: ${Date.now() - start}ms`
);
```

---

# Quoted Messages

Zoe exposes the quoted/replied-to message through:

```js
m.reply_message
```

Do not assume `m.quoted` is the active quoted-message interface.

Check first:

```js
if (!m.reply_message) {
  return m.reply("Reply to a message.");
}
```

Example:

```js
Zoe(
  {
    pattern: "quoted",
    fromMe: false,
    desc: "Check quoted message.",
    type: "utility"
  },
  async (m) => {
    if (!m.reply_message) {
      return m.reply("Reply to a message.");
    }

    await m.reply("A message was quoted.");
  }
);
```

---

# Downloading Quoted Media

When the quoted message contains downloadable media, use:

```js
m.reply_message.download()
```

Example:

```js
Zoe(
  {
    pattern: "download",
    fromMe: false,
    desc: "Download replied media.",
    type: "media"
  },
  async (m) => {
    if (!m.reply_message) {
      return m.reply("Reply to a media message.");
    }

    const buffer = await m.reply_message.download();

    // Process buffer here.
  }
);
```

The returned value can then be passed to libraries such as:

```js
Jimp
ffmpeg
sharp
file-type
```

depending on what the plugin is doing.

---

# Groups and Private Chats

Use `m.isGroup` and `m.isPm` to restrict where commands work.

## Group-only command

```js
Zoe(
  {
    pattern: "groupinfo",
    fromMe: false,
    desc: "Show group information.",
    type: "group"
  },
  async (m) => {
    if (!m.isGroup) {
      return m.reply("This command can only be used in groups.");
    }

    await m.reply("This is a group.");
  }
);
```

## Private-only command

```js
Zoe(
  {
    pattern: "private",
    fromMe: false,
    desc: "Private chat command.",
    type: "utility"
  },
  async (m) => {
    if (!m.isPm) {
      return m.reply("This command can only be used in private chat.");
    }

    await m.reply("Private chat detected.");
  }
);
```

---

# User Information

Example:

```js
Zoe(
  {
    pattern: "whoami",
    fromMe: false,
    desc: "Show sender information.",
    type: "info"
  },
  async (m) => {
    await m.reply(
      `Name: ${m.pushName}\n` +
      `Sender: ${m.sender}\n` +
      `Chat: ${m.jid}`
    );
  }
);
```

---

# Event-Based Plugins

Zoe can execute handlers based on message/event types using the `on` property.

Example:

```js
Zoe(
  {
    on: "text",
    fromMe: false,
    type: "event"
  },
  async (m) => {
    console.log("Text message:", m.text);
  }
);
```

This is different from a normal command:

```js
{
  pattern: "hello"
}
```

A command waits for a matching command.

An `on` handler listens for a matching message type.

---

# Event Types

## `on: "text"`

Runs for text messages.

```js
Zoe(
  {
    on: "text",
    fromMe: false,
    type: "event"
  },
  async (m) => {
    console.log(m.text);
  }
);
```

---

## `on: "image"`

Runs when an image message is received.

```js
Zoe(
  {
    on: "image",
    fromMe: false,
    type: "media"
  },
  async (m) => {
    await m.reply("Image received.");
  }
);
```

---

## `on: "video"`

Runs when a video message is received.

```js
Zoe(
  {
    on: "video",
    fromMe: false,
    type: "media"
  },
  async (m) => {
    await m.reply("Video received.");
  }
);
```

---

## `on: "audio"`

Runs when an audio message is received.

```js
Zoe(
  {
    on: "audio",
    fromMe: false,
    type: "media"
  },
  async (m) => {
    await m.reply("Audio received.");
  }
);
```

---

## `on: "sticker"`

Runs when a sticker message is received.

```js
Zoe(
  {
    on: "sticker",
    fromMe: false,
    type: "media"
  },
  async (m) => {
    await m.reply("Sticker received.");
  }
);
```

---

## `on: "delete"`

Runs for supported WhatsApp delete/protocol-message events.

```js
Zoe(
  {
    on: "delete",
    fromMe: false,
    type: "moderation"
  },
  async (m) => {
    console.log("Delete event:", m.messageId);
  }
);
```

For delete events, Zoe sets:

```js
m.messageId
```

from the deleted message's protocol key.

---

## `on: "all"`

Runs for all supported message types handled by the event system.

```js
Zoe(
  {
    on: "all",
    fromMe: false,
    type: "event"
  },
  async (m) => {
    console.log("Message received:", m.type);
  }
);
```

Use `on: "all"` carefully because it can execute very frequently.

---

# Command vs Event Handler

### Command

```js
Zoe(
  {
    pattern: "ping",
    fromMe: false,
    desc: "Check latency.",
    type: "info"
  },
  async (m, match, client) => {
    await m.reply("Pong!");
  }
);
```

Triggered by:

```text
/ping
```

### Event

```js
Zoe(
  {
    on: "image",
    fromMe: false,
    type: "media"
  },
  async (m, match, client) => {
    await m.reply("Image received.");
  }
);
```

Triggered by an image message.

---

# Complete Command Examples

## Simple Command

```js
Zoe(
  {
    pattern: "hello",
    fromMe: false,
    desc: "Say hello.",
    type: "fun"
  },
  async (m) => {
    await m.reply("Hello!");
  }
);
```

---

## Command With Arguments

```js
Zoe(
  {
    pattern: "echo",
    fromMe: false,
    desc: "Repeat supplied text.",
    type: "fun"
  },
  async (m, match) => {
    if (!match) {
      return m.reply("Usage: /echo <text>");
    }

    await m.reply(match);
  }
);
```

Usage:

```text
/echo Hello Zoe
```

Output:

```text
Hello Zoe
```

---

## Command With Aliases

```js
Zoe(
  {
    pattern: "ping",
    alias: ["p", "pong"],
    fromMe: false,
    desc: "Check bot latency.",
    type: "info"
  },
  async (m) => {
    const start = Date.now();

    const msg = await m.reply("Ping...");

    await msg.edit(
      `Pong!\nLatency: ${Date.now() - start}ms`
    );
  }
);
```

Works with:

```text
/ping
/p
/pong
```

---

## Owner/Sudo Command

```js
Zoe(
  {
    pattern: "owner",
    fromMe: true,
    desc: "Owner-only command.",
    type: "owner"
  },
  async (m) => {
    await m.reply("Owner command executed.");
  }
);
```

---

## Group Command

```js
Zoe(
  {
    pattern: "group",
    fromMe: false,
    desc: "Group-only command.",
    type: "group"
  },
  async (m) => {
    if (!m.isGroup) {
      return m.reply("This command only works in groups.");
    }

    await m.reply(`Group JID: ${m.jid}`);
  }
);
```

---

## Quoted Media Command

```js
Zoe(
  {
    pattern: "buffer",
    fromMe: false,
    desc: "Download replied media as a buffer.",
    type: "media"
  },
  async (m) => {
    if (!m.reply_message) {
      return m.reply("Reply to a media message.");
    }

    try {
      const buffer = await m.reply_message.download();

      await m.reply(
        `Downloaded ${buffer.length} bytes.`
      );
    } catch (error) {
      console.error(error);
      await m.reply("Failed to download the media.");
    }
  }
);
```

---

## API Command

External APIs can be used normally.

```js
const axios = require("axios");

Zoe(
  {
    pattern: "ip",
    fromMe: false,
    desc: "Get public IP information.",
    type: "utility"
  },
  async (m, match) => {
    try {
      const response = await axios.get("https://api.ipify.org?format=json");

      await m.reply(
        `Your IP:\n${response.data.ip}`
      );
    } catch (error) {
      console.error(error);
      await m.reply("Failed to retrieve IP information.");
    }
  }
);
```

---

# External Plugins

Zoe supports externally hosted plugins.

An external plugin is simply a JavaScript plugin hosted at a URL.

For example:

```text
https://example.com/plugins/weather.js
```

The plugin can then be registered with Zoe's external plugin system.

The external plugin database stores:

```text
name
url
```

The internal plugin controller maps the database values into:

```js
{
  name: "...",
  url: "..."
}
```



---

# How External Plugins Are Loaded

When Zoe starts, it loads the registered external plugins.

For each external plugin:

1. Zoe checks whether `plugins/<name>.js` already exists.
2. If it does not exist, Zoe downloads the plugin from its configured URL.
3. The downloaded JavaScript is written to:
   ```text
   ./plugins/<name>.js
   ```
4. Zoe loads the plugin using `require()`.
5. The plugin registers its commands with Zoe.

This means the URL must return the actual JavaScript source code of the plugin.

---

# Publishing an External Plugin

The easiest way to distribute a plugin is to host a raw JavaScript file.

For example:

```text
my-plugin.js
```

Host the file somewhere that provides a direct/raw URL.

The URL should return:

```js
Zoe(
  {
    pattern: "example",
    fromMe: false,
    desc: "Example external plugin.",
    type: "utility"
  },
  async (m) => {
    await m.reply("External plugin works!");
  }
);
```

Do not provide an HTML webpage URL when Zoe expects JavaScript.

Bad:

```text
https://example.com/plugin-page
```

Good:

```text
https://example.com/plugin.js
```

or a raw-file URL from a source-code hosting service.

---

# External Plugin Example

File:

```text
weather.js
```

Contents:

```js
const axios = require("axios");

Zoe(
  {
    pattern: "weather",
    alias: ["w"],
    fromMe: false,
    desc: "Get weather information.",
    type: "utility"
  },
  async (m, match) => {
    if (!match) {
      return m.reply("Usage: /weather <city>");
    }

    try {
      const response = await axios.get(
        "https://example.com/weather",
        {
          params: {
            city: match
          }
        }
      );

      await m.reply(
        `Weather for ${match}:\n${response.data}`
      );
    } catch (error) {
      console.error(error);
      await m.reply("Failed to fetch weather.");
    }
  }
);
```

Host that file and provide its raw JavaScript URL to the Zoe external plugin installer.

---

# Important External Plugin Behavior

External plugins are installed during Zoe startup.

If the plugin file already exists locally:

```text
plugins/plugin-name.js
```

Zoe does not download it again during that startup.

Therefore, when updating an external plugin, the existing cached plugin file must be removed/reinstalled or otherwise handled by the bot's plugin-management system before Zoe can download the updated version.

If you publish a new version at the same URL, do not assume a running Zoe instance will automatically replace the old cached file.

---

# Plugin Naming Rules

The external plugin name becomes the local filename:

```text
plugins/<name>.js
```

For example:

```text
name = "weather"
```

results in:

```text
plugins/weather.js
```

Use simple names:

```text
weather
youtube
instagram
ai
translator
```

Avoid:

```text
weather plugin!!!
../something
my/plugin
```

Plugin names should be safe filesystem-friendly names.

---

# Dependencies

A plugin can use Node.js modules.

Example:

```js
const axios = require("axios");
```

However, remember that an external plugin runs inside the Zoe installation.

If the dependency is not installed in Zoe's environment, the plugin can fail to load.

For example:

```js
const somePackage = require("some-package");
```

requires that package to exist in the bot environment.

Prefer dependencies already available in Zoe when possible.

For simple API requests:

```js
const axios = require("axios");
```

is preferable to introducing unnecessary dependencies.

---

# Error Handling

Always use error handling for network requests, media processing, file operations, and other operations that can fail.

Recommended:

```js
Zoe(
  {
    pattern: "example",
    fromMe: false,
    desc: "Example command.",
    type: "utility"
  },
  async (m) => {
    try {
      // Your code
      await m.reply("Success.");
    } catch (error) {
      console.error("[example]", error);

      await m.reply(
        "An error occurred while processing the command."
      );
    }
  }
);
```

Do not silently ignore errors.

---

# API Requests

For API-based plugins:

```js
const axios = require("axios");

Zoe(
  {
    pattern: "api",
    fromMe: false,
    desc: "Call an API.",
    type: "utility"
  },
  async (m, match) => {
    try {
      const response = await axios.get(
        "https://example.com/api"
      );

      await m.reply(
        JSON.stringify(response.data, null, 2)
      );
    } catch (error) {
      console.error("[api]", error);

      await m.reply(
        `API request failed: ${error.message}`
      );
    }
  }
);
```

---

# Input Validation

Never assume that users provide valid arguments.

Bad:

```js
const url = match;

const response = await axios.get(url);
```

Better:

```js
if (!match) {
  return m.reply("Usage: /download <url>");
}

const url = match.trim();
```

For URLs:

```js
try {
  new URL(match);
} catch {
  return m.reply("Invalid URL.");
}
```

---

# Avoid Blocking the Bot

Avoid expensive synchronous operations.

Bad:

```js
fs.readFileSync(largeFile);
```

inside a frequently executed event handler.

Prefer asynchronous APIs:

```js
await fs.promises.readFile(file);
```

Long-running operations should also provide user feedback:

```js
const msg = await m.reply("Processing...");

const result = await processSomething();

await msg.edit("Completed.");
```

---

# Avoid Duplicate Commands

Do not create multiple plugins using the same command name.

Bad:

```js
pattern: "test"
```

in one plugin and:

```js
pattern: "test"
```

in another plugin.

Depending on registration order, one command may capture the message before another command gets a chance to handle it.

Use unique command names.

---

# Command Naming

Use short and predictable command names.

Good:

```text
ping
alive
weather
translate
sticker
yt
insta
```

Avoid unnecessarily long names:

```text
downloadyoutubevideoinhighquality
```

Use aliases when multiple names are useful:

```js
pattern: "youtube",
alias: ["yt", "ytdl"]
```

---

# Plugin Organization

A plugin should generally focus on one feature.

Good:

```text
weather.js
translate.js
sticker.js
youtube.js
instagram.js
```

Instead of putting everything into one huge file:

```text
everything.js
```

This makes plugins easier to maintain and distribute.

---

# Recommended Plugin Template

Use this as the standard starting point:

```js
const axios = require("axios");

Zoe(
  {
    pattern: "example",
    alias: ["ex"],
    fromMe: false,
    desc: "Example Zoe plugin.",
    type: "utility"
  },
  async (m, match, client) => {
    try {
      if (!match) {
        return m.reply("Usage: /example <input>");
      }

      // Your plugin logic here.

      await m.reply(`Input: ${match}`);
    } catch (error) {
      console.error("[example]", error);

      await m.reply(
        "Something went wrong while executing this command."
      );
    }
  }
);
```

---

# Minimal Plugin Template

For simple plugins:

```js
Zoe(
  {
    pattern: "hello",
    fromMe: false,
    desc: "Say hello.",
    type: "fun"
  },
  async (m) => {
    await m.reply("Hello!");
  }
);
```

---

# Advanced Plugin Template

```js
const axios = require("axios");

Zoe(
  {
    pattern: "search",
    alias: ["find"],
    fromMe: false,
    desc: "Search for information.",
    type: "search"
  },
  async (m, match, client) => {
    if (!match) {
      return m.reply("Usage: /search <query>");
    }

    const loading = await m.reply("Searching...");

    try {
      const response = await axios.get(
        "https://example.com/api/search",
        {
          params: {
            q: match
          }
        }
      );

      const result = response.data;

      await loading.edit(
        `Search result:\n\n${JSON.stringify(result, null, 2)}`
      );
    } catch (error) {
      console.error("[search]", error);

      await loading.edit(
        "Search failed."
      );
    }
  }
);
```

---

# Security

External plugins execute JavaScript inside the Zoe process.

This is important.

A plugin can potentially access:

- Node.js APIs
- Filesystem
- Environment variables
- Installed packages
- Network resources
- The Zoe client
- Bot credentials/session-related resources depending on what the environment exposes

Therefore:

> **Only install external plugins from developers or sources you trust.**

Never blindly install unknown JavaScript.

A malicious plugin could potentially execute arbitrary Node.js code.

For plugin authors, do not:

- Steal credentials
- Read authentication/session files
- Exfiltrate environment variables
- Upload private bot data
- Modify unrelated bot files
- Execute destructive shell commands
- Hide malicious behavior inside obfuscated code

Plugins should only perform the operations required for their documented functionality.

---

# Plugin Compatibility

A plugin should avoid depending on undocumented internal Zoe implementation details unless absolutely necessary.

Prefer the public plugin interface:

```js
Zoe(...)
```

and message properties/helpers such as:

```js
m.reply(...)
m.isGroup
m.isPm
m.isSudo
m.reply_message
m.jid
m.sender
m.text
```

If a plugin directly depends on internal Zoe/Baileys structures, it may break when Zoe or Baileys is updated.

---

# Debugging

When developing a plugin, use:

```js
console.log(...)
```

Example:

```js
Zoe(
  {
    pattern: "debug",
    fromMe: true,
    desc: "Debug message information.",
    type: "developer"
  },
  async (m, match, client) => {
    console.log("Message:", m);
    console.log("Match:", match);
    console.log("Client available:", !!client);

    await m.reply("Debug information printed to console.");
  }
);
```

For errors:

```js
try {
  // code
} catch (error) {
  console.error(error);
}
```

---

# Troubleshooting

## Command does not respond

Check:

1. The plugin file is actually loaded.
2. `pattern` is spelled correctly.
3. The command is being used with the correct prefix.
4. `fromMe` is not restricting the command.
5. Another plugin is not using the same command.
6. There are no JavaScript syntax errors.

---

## External plugin does not install

Check:

1. The URL is reachable.
2. The URL returns JavaScript source code.
3. The plugin name is valid.
4. The downloaded file is valid JavaScript.
5. Required dependencies exist in the Zoe environment.
6. Check the Zoe startup logs.

Zoe reports the external plugin installation stage during startup.

---

## Plugin downloads but command does not work

The file may have been downloaded successfully but failed while being loaded.

Check:

```js
require(...)
```

compatibility and inspect the startup error.

Common causes:

```text
SyntaxError
ReferenceError
Cannot find module
TypeError
```

---

## `match` is empty

For:

```text
/ping hello
```

the command must be:

```js
pattern: "ping"
```

and:

```js
match
```

will contain:

```text
hello
```

If the user sends only:

```text
/ping
```

then:

```js
match === ""
```

---

## Quoted message is unavailable

Check:

```js
if (!m.reply_message) {
  return m.reply("Reply to a message.");
}
```

Do not attempt to download media if there is no quoted/replied message.

---

# Plugin Checklist

Before publishing a plugin, verify:

- [ ] Plugin is valid JavaScript.
- [ ] Command name is unique.
- [ ] `pattern` is a string.
- [ ] `fromMe` is correctly configured.
- [ ] `desc` clearly describes the command.
- [ ] `type` is appropriate.
- [ ] `alias` contains valid alternative command names if needed.
- [ ] User input is validated.
- [ ] Errors are handled.
- [ ] API failures are handled.
- [ ] Quoted messages are checked before downloading.
- [ ] Required dependencies are available.
- [ ] No credentials or private data are exposed.
- [ ] External plugin URL returns raw JavaScript.
- [ ] Plugin has been tested on a real Zoe instance.
- [ ] Plugin does not conflict with existing commands.

---

# Quick Reference

## Basic Command

```js
Zoe(
  {
    pattern: "ping",
    fromMe: false,
    desc: "Check bot latency.",
    type: "info"
  },
  async (m, match, client) => {
    await m.reply("Pong!");
  }
);
```

## Alias

```js
alias: ["p", "pong"]
```

## Owner/Sudo

```js
fromMe: true
```

## Public

```js
fromMe: false
```

## Group Check

```js
if (!m.isGroup) {
  return m.reply("Group only.");
}
```

## Private Check

```js
if (!m.isPm) {
  return m.reply("Private only.");
}
```

## Arguments

```js
async (m, match) => {
  console.log(match);
}
```

## Client

```js
async (m, match, client) => {
  // advanced WhatsApp operations
}
```

## Reply

```js
await m.reply("Hello");
```

## Edit

```js
const msg = await m.reply("Loading...");
await msg.edit("Done!");
```

## Quoted Message

```js
m.reply_message
```

## Download Quoted Media

```js
const buffer = await m.reply_message.download();
```

## Event Handler

```js
Zoe(
  {
    on: "image",
    fromMe: false,
    type: "media"
  },
  async (m) => {
    await m.reply("Image received.");
  }
);
```

## Multiple Commands

```js
Zoe({...}, async (m) => {});
Zoe({...}, async (m) => {});
Zoe({...}, async (m) => {});
```

---

# Final Example

A complete practical plugin:

```js
const axios = require("axios");

Zoe(
  {
    pattern: "github",
    alias: ["gh"],
    fromMe: false,
    desc: "Get information about a GitHub user.",
    type: "search"
  },
  async (m, match) => {
    if (!match) {
      return m.reply("Usage: /github <username>");
    }

    const username = match.trim();

    const loading = await m.reply(
      "Fetching GitHub information..."
    );

    try {
      const { data } = await axios.get(
        `https://api.github.com/users/${encodeURIComponent(username)}`
      );

      const text =
        `*GitHub User*\n\n` +
        `Name: ${data.name || "N/A"}\n` +
        `Username: ${data.login}\n` +
        `Repositories: ${data.public_repos}\n` +
        `Followers: ${data.followers}\n` +
        `Following: ${data.following}\n` +
        `Profile: ${data.html_url}`;

      await loading.edit(text);
    } catch (error) {
      console.error("[github]", error);

      await loading.edit(
        "Failed to fetch GitHub information."
      );
    }
  }
);
```

Usage:

```text
/github octocat
```

or:

```text
/gh octocat
```

---

# Summary

The Zoe plugin system is built around:

```js
Zoe(config, handler);
```

Normal command:

```js
Zoe(
  {
    pattern: "command",
    alias: [],
    fromMe: false,
    desc: "Description",
    type: "category"
  },
  async (m, match, client) => {
    // command logic
  }
);
```

Event handler:

```js
Zoe(
  {
    on: "image",
    fromMe: false,
    type: "media"
  },
  async (m, match, client) => {
    // event logic
  }
);
```

The core interfaces to remember are:

```js
m.reply()
m.isGroup
m.isPm
m.isSudo
m.reply_message
m.jid
m.sender
m.text
```

and:

```js
match
client
```

for command arguments and advanced WhatsApp operations.

Build plugins as independent, focused JavaScript modules, validate user input, handle errors properly, and only distribute external plugins that are safe to execute.
