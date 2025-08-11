
# Zoe Plugin

A lightweight and easy-to-use command handler framework for your bot, designed to simplify the creation of bot commands with clear patterns, permissions, and descriptions.

---

## Features

- Simple command pattern matching
- Permission control with `fromMe` flag (restrict commands to bot owner)
- Command metadata (description, type/category)
- Async handlers with message reply and edit support
- Minimal setup with easy integration

---

## Installation

Make sure you have Node.js installed.

1. Clone or download the repository containing Zoe.
2. Import the `Zoe` function from the `././lib` module in your project.

```js
const { Zoe } = require('././lib');
```

---

## How to Create Commands

Commands in Zoe are created by calling the `Zoe` function with two parameters:

### 1. Command Configuration Object

| Property | Type    | Description                                         |
| -------- | ------- | ------------------------------------------------- |
| `pattern`| String  | The trigger word or regex pattern for the command |
| `fromMe` | Boolean | Whether the command is restricted to bot owner    |
| `desc`   | String  | A brief description of the command                 |
| `type`   | String  | The category/type of the command                    |

### 2. Command Handler (Async Function)

The handler is an async function that receives a message object `m`. This object supports `.reply()` and `.edit()` methods for sending and updating messages.

---

## Example Command: Ping

```js
Zoe(
  {
    pattern: 'ping',
    fromMe: mode, // Boolean: true if only the bot owner can use this
    desc: 'Bot response in milliseconds.',
    type: 'info'
  },
  async (m) => {
    const start = Date.now();
    const msg = await m.reply('*Ping*');
    await msg.edit(`*Zoe!*\n*Latency!* ${Date.now() - start}ms`);
  }
);
```

**What it does:**

- When a user sends the command `ping`, the bot replies with "*Ping*".
- Then it edits that reply to show the latency (response time) in milliseconds.

---

## Usage

1. Import the Zoe function from the library:

   ```js
   const { Zoe } = require('././lib');
   ```

2. Define your command(s) using `Zoe()`:

   ```js
   Zoe(
     {
       pattern: 'hello',
       fromMe: false,
       desc: 'Say hello to the bot.',
       type: 'fun'
     },
     async (m) => {
       await m.reply('Hello! How can I help you today?');
     }
   );
   ```

3. Run your bot as usual — commands matching the pattern will automatically trigger the associated handler.

---

## Command Configuration Explained

- **pattern**: A string or regex to detect the command keyword in chat.
- **fromMe**: A boolean to restrict command usage to the bot owner (`true`) or allow everyone (`false`).
- **desc**: A short description to document what the command does.
- **type**: Categorize your command (e.g., `'info'`, `'admin'`, `'fun'`) for better organization.

---

## Message Object `m`

The message object passed to the handler has useful methods:

- `.reply(text)`: Sends a message replying to the user.
- `.edit(text)`: Edits a previously sent message.

This allows for dynamic and interactive command responses.

---

## Tips

- Keep command handlers small and focused.
- Use async/await for cleaner asynchronous code.
- Leverage the `desc` and `type` for auto-generating help menus or documentation.

---

## Contributing

Feel free to submit issues or pull requests to improve Zoe. Contributions are welcome!

---

## License

Specify your license here (e.g., MIT).

---
