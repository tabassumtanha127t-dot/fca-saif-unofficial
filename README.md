# fca-saif-unofficial

> Unofficial Facebook Chat API for Node.js — Stable, error-free, all features unlocked.  
> Built for **Goat-Bot-V2** and similar bot frameworks.

---

## Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [Use in Goat-Bot-V2](#use-in-goat-bot-v2)
- [Features](#features)
- [API Methods](#api-methods)
- [Events](#events)
- [Options](#options)
- [fca-config.json](#fca-configjson)
- [Important Notes](#important-notes)
- [Author](#author)

---

## Installation

    npm install fca-saif-unofficial

---

## Quick Start

### With AppState (recommended)

    const login = require("fca-saif-unofficial");
    const fs = require("fs");

    const appState = JSON.parse(fs.readFileSync("appstate.json", "utf8"));

    login({ appState }, { listenEvents: true }, (err, api) => {
      if (err) return console.error(err);

      api.listen((err, message) => {
        if (err) return console.error(err);
        if (message.type === "message") {
          api.sendMessage("Hello!", message.threadID);
        }
      });
    });

### With Email & Password

    const login = require("fca-saif-unofficial");

    login({ email: "your@email.com", password: "yourpassword" }, { listenEvents: true }, (err, api) => {
      if (err) return console.error(err);
      console.log("Logged in!");
    });

### With async/await

    const login = require("fca-saif-unofficial");

    (async () => {
      const api = await login({ appState: [...] }, { listenEvents: true });
      console.log("Ready!");
    })();

---

## Use in Goat-Bot-V2

In your bot `package.json`, change the `priyanshu-fca` line to:

    "priyanshu-fca": "npm:fca-saif-unofficial@latest"

Then create `fca-config.json` in your bot root (see section below) and run:

    npm install

No other code changes needed.

---

## Features

- No `statusCose` typo bug — HTTP errors handled properly via axios
- Modern axios-based HTTP — replaces legacy broken `request` library
- MQTT real-time messaging — fast, low-latency message delivery
- Auto-retry on 5xx errors — exponential backoff, up to 5 attempts
- Rate limit (429) detection — auto waits and retries
- Network timeout handling — no crashes on connection drops
- Checkpoint detection — detects checkpoint events (282 / 956)
- Auto-reconnect every 60 min — keeps session alive on Render/Railway
- Reaction events — `message_reaction` fires correctly
- Unsend events — `message_unsend` fires correctly
- Reply message events — `message_reply` with full metadata
- Attachment and stream upload — images, videos, audio, files
- Mention support — send and receive @mentions
- TypeScript types included — full index.d.ts
- Anti-crash global handlers — unhandledRejection and uncaughtException safely caught
- Sequelize DB caching — reduces repeated Facebook API calls

---

## API Methods

### Messaging

    // Send a text message
    api.sendMessage("Hello!", threadID, callback);

    // Send with attachment
    const fs = require("fs");
    api.sendMessage(
      { body: "Check this!", attachment: fs.createReadStream("image.jpg") },
      threadID, callback
    );

    // Send with @mention
    api.sendMessage(
      { body: "@John hello", mentions: [{ id: "100...", tag: "@John", fromIndex: 0 }] },
      threadID, callback
    );

    // Reply to a specific message
    api.sendMessage("Replied!", threadID, callback, replyMessageID);

    // React to a message
    api.setMessageReaction("😍", messageID, threadID, callback);

    // Remove reaction
    api.setMessageReaction("", messageID, threadID, callback);

    // Edit a sent message
    api.editMessage("Updated text", messageID, callback);

    // Unsend/delete a message
    api.unsendMessage(messageID, threadID, callback);

    // Mark thread as read
    api.markAsRead(threadID, callback);

    // Send typing indicator
    api.sendTypingIndicator(threadID, callback);

### Thread Info

    // Get thread info
    api.getThreadInfo(threadID, callback);

    // Get thread list
    api.getThreadList(limit, timestamp, tags, callback);

    // Get thread history
    api.getThreadHistory(threadID, amount, timestamp, callback);

    // Set thread title
    api.setTitle("New Group Name", threadID, callback);

    // Change thread emoji
    api.changeThreadEmoji("🔥", threadID, callback);

    // Change thread color
    api.changeThreadColor("#FF0000", threadID, callback);

    // Change thread image
    api.changeGroupImage(readableStream, threadID, callback);

### User Info

    // Get user info by ID
    api.getUserInfo(userID, callback);

    // Get user ID by name
    api.getUserID("John Doe", callback);

    // Get friends list
    api.getFriendsList(callback);

    // Change nickname
    api.changeNickname("Nickname", threadID, userID, callback);

### Group Management

    // Add user to group
    api.addUserToGroup(userID, threadID, callback);

    // Remove user from group
    api.removeUserFromGroup(userID, threadID, callback);

    // Make/remove admin
    api.changeAdminStatus(threadID, userID, true, callback);  // make admin
    api.changeAdminStatus(threadID, userID, false, callback); // remove admin

### Other

    // Get a specific message
    api.getMessage(messageID, threadID, callback);

    // Create a new group
    api.createNewGroup(["userID1", "userID2"], "Group Name", callback);

    // Create a poll
    api.createPoll("Question?", threadID, { "Option 1": true, "Option 2": false }, callback);

    // Block / unblock user
    api.changeBlockedStatus(userID, true, callback);  // block
    api.changeBlockedStatus(userID, false, callback); // unblock

    // Get appState (save session)
    api.getAppState();

    // Logout
    api.logout(callback);

---

## Events

**Requires `listenEvents: true` in options.**

    api.listen((err, event) => {
      if (err) return console.error(err);

      switch (event.type) {

        case "message":
          console.log(event.senderID, event.body, event.threadID);
          break;

        case "message_reply":
          console.log(event.messageReply.body); // original message
          console.log(event.body);              // reply body
          break;

        case "message_reaction":
          console.log(event.userID);    // who reacted
          console.log(event.reaction);  // emoji (empty = removed)
          console.log(event.messageID);
          break;

        case "message_unsend":
          console.log(event.senderID, event.messageID);
          break;

        case "event":
          // log:subscribe       — user joined group
          // log:unsubscribe     — user left group
          // log:thread-name     — group renamed
          // log:thread-color    — color changed
          // log:thread-icon     — emoji changed
          // log:thread-admins   — admin changed
          console.log(event.logMessageType);
          break;

        case "read_receipt":
          console.log(event.threadID, event.time);
          break;

        case "typ":
          console.log(event.isTyping, event.from, event.threadID);
          break;
      }
    });

---

## Options

    login(credentials, {
      listenEvents: true,      // REQUIRED for reactions/unsend/reply events
      selfListen: false,       // receive own messages (default: false)
      selfListenEvent: false,  // receive own events (default: false)
      listenTyping: false,     // receive typing indicators (default: false)
      updatePresence: false,   // receive online/offline status (default: false)
      autoMarkRead: false,     // auto mark messages as read (default: false)
      autoReconnect: true,     // auto reconnect on disconnect (default: true)
      forceLogin: false,       // skip login checks (default: false)
      online: true             // appear online (default: true)
    }, callback);

---

## fca-config.json

Create this file in your **bot root folder** (same level as `package.json`):

    {
      "autoUpdate": false,
      "mqtt": {
        "enabled": true,
        "reconnectInterval": 3600
      },
      "autoLogin": true,
      "apiServer": "https://minhdong.site",
      "apiKey": "",
      "credentials": {
        "email": "",
        "password": "",
        "twofactor": ""
      },
      "antiGetInfo": {
        "AntiGetThreadInfo": false,
        "AntiGetUserInfo": false
      },
      "remoteControl": {
        "enabled": false,
        "url": "",
        "token": "",
        "autoReconnect": true
      }
    }

If this file is missing, default values are used automatically — no crash.

---

## Important Notes

1. `listenEvents: true` is mandatory for reaction, unsend, and reply events to fire.
2. This library uses MQTT for real-time messaging — not HTTP polling.
3. Works on Node.js 12 and above (Node 22 recommended for Goat-Bot-V2).
4. Save your appState regularly — sessions expire over time.
5. Do not use this for spam or ToS violations — your account may get banned.

---

## Author

Made by **Saif** — https://github.com/tabassumtanha127t-dot

Based on the work of DongDev and the original facebook-chat-api project.
