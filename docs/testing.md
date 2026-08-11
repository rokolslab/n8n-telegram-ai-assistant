# Testing

## Original verification

The original implementation was validated structurally and then exercised through real Telegram webhook executions.

| Path | Result |
|---|---|
| Unsupported command (`/help`) | Success; fallback message sent |
| Image generation (`/image футуристический офис с ИИ-ассистентом`) | Success; PNG `1024x1024` generated and sent |
| Start command (`/start`) | Success; generated plain-text reply sent |

The workflow configuration and key node settings were also validated before the live runs.

## Runtime issues found during testing

Two issues were caught and fixed during the real test cycle:

1. The configured OpenAI text verbosity value `low` was not supported for the selected text operation/model combination. It was changed to `medium`.
2. The Telegram text-send expression initially risked forwarding a structured OpenAI response instead of the extracted text. The expression was adjusted to select the actual output text.

These fixes are preserved in the sanitized portfolio export.

## Regression tests after import

### 1. Fallback path

Send:

```text
/help
```

Expected:

- `Route Command` selects `fallback`;
- no OpenAI node runs;
- Telegram receives the help message.

### 2. Start path

Send:

```text
/start
```

Expected:

- Telegram shows `typing`;
- `Generate Start Reply` runs;
- Telegram receives plain generated text;
- no image node runs.

### 3. Image path

Send:

```text
/image simple blue robot icon
```

Expected:

- Telegram shows `upload_photo`;
- prompt text excludes the `/image` command prefix;
- image generation runs;
- binary result is passed to `Send Generated Image`;
- Telegram receives a photo.

### 4. Empty image prompt

Send:

```text
/image
```

Expected in the current reusable workflow:

- image branch is selected;
- the configured default image prompt is used.

For a production implementation, consider rejecting an empty image prompt instead.

## Portfolio evidence checklist

Capture only non-sensitive information:

- full workflow canvas;
- Telegram conversation showing `/start`, fallback and image response;
- execution history showing successful branches;
- `Normalize Message` expressions;
- `Route Command` switch configuration;
- one successful image-generation execution.

Never show bot tokens, API keys, credential details, private chat identifiers or unrelated personal messages.
