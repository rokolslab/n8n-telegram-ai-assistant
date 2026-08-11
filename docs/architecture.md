# Architecture

## Overview

The workflow treats Telegram as an event source and delivery channel, while n8n performs deterministic preprocessing and routing before invoking OpenAI.

```text
Telegram Trigger
      ↓
Normalize Message
      ↓
Runtime Settings
   ↙       ↘
chat action  main data
   ↓          ↓
   └──────► Merge
              ↓
          Route Command
          /     |      \
      /start  /image  fallback
        ↓       ↓        ↓
     OpenAI   OpenAI   Telegram
      text     image     text
        ↓       ↓
    Telegram Telegram
      text     photo
```

## 1. Telegram event intake

`Telegram Trigger` receives message updates. The reusable export intentionally contains no original webhook UUID or credential reference; n8n recreates environment-specific webhook details after import and credential assignment.

## 2. Deterministic preprocessing

`Normalize Message` creates a stable internal structure:

- `message.text`
- `message.command`
- `message.prompt`
- `message.chat_id`
- `message.user_name`

Optional chaining and defaults are used so missing fields do not immediately break the workflow.

## 3. Runtime settings

`Runtime Settings` stores model parameters and the system instruction used for the `/start` branch. It also selects the Telegram chat action:

- `/image` → `upload_photo`
- other commands → `typing`

Keeping these values in one node makes the behavior easy to inspect and change without editing several downstream nodes.

## 4. Chat-action synchronization

The workflow sends a Telegram chat action and uses a `Merge` node in `Choose Branch` mode so routing continues with the original data after the status action has been sent.

This improves user feedback without replacing or corrupting the primary message payload.

## 5. Command routing

`Route Command` uses deterministic rules:

- `/start` → text-generation branch;
- `/image` → image-generation branch;
- anything else → fallback branch.

The fallback path does not call OpenAI, which avoids unnecessary model usage for unsupported commands.

## 6. Text generation

`Generate Start Reply` calls an OpenAI text model using a system instruction plus the incoming command context. The Telegram send node extracts the text from the model response rather than forwarding the raw response object.

A real test exposed a response-mapping issue in this area; the final expression was adjusted to return clean model text.

## 7. Image generation

`Generate Image` uses an OpenAI image model and writes the result to binary property `data`. `Send Generated Image` then passes that binary object directly to Telegram using `sendPhoto`.

The tested implementation generated a PNG at `1024x1024`.

## Reliability and production extensions

A production deployment could add:

- centralized Error Trigger workflow;
- explicit validation for empty `/image` prompts;
- rate limiting and per-user quotas;
- allow/deny rules for users or chats;
- abuse and moderation controls for image prompts;
- structured execution logging and correlation IDs;
- retry/error branches for Telegram and OpenAI nodes;
- observability for model latency, token usage and image-generation cost.
