# Setup

## Requirements

- an n8n instance compatible with the exported nodes;
- a Telegram Bot API credential;
- an OpenAI credential;
- OpenAI text and image models available in your environment.

## Import

Import:

```text
workflow/telegram-ai-assistant.json
```

The public export is environment-neutral and intentionally excludes the original workflow/version identifiers, webhook UUIDs and credential references.

## Credentials

Attach credentials to:

- `Telegram Trigger`;
- `Send Chat Action`;
- `Send Start Reply`;
- `Send Generated Image`;
- `Send Fallback`;
- `Generate Start Reply`;
- `Generate Image`.

All Telegram nodes should normally use the same test bot credential. Both OpenAI nodes can use the same OpenAI account unless your environment requires separate credentials.

## Telegram webhook

Telegram Trigger requires a reachable HTTPS webhook endpoint. When the workflow is activated or placed into the appropriate test/listening mode, n8n registers the webhook for the bot.

Use a dedicated test bot while validating the workflow. Do not expose bot tokens in screenshots, exports or documentation.

## Models

The reusable export is configured with:

```text
Text:  gpt-4.1
Image: gpt-image-1-mini
```

Model availability can change. If either model is unavailable in your n8n/OpenAI environment, select an equivalent supported text or image model and repeat the regression tests.

## First test sequence

After credentials are connected:

1. activate or start listening with the workflow;
2. send `/help` and confirm the fallback message;
3. send `/start` and confirm a plain-text response;
4. send `/image simple blue robot icon` and confirm a photo is returned;
5. inspect the n8n execution history for all three paths.

See [Testing](testing.md) for the expected results.
