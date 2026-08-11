# Evidence and provenance

This repository contains a sanitized, reusable version of a workflow that was originally tested in a separate n8n environment.

## Recorded live executions

The source implementation recorded these real Telegram webhook executions:

- `/help` — successful fallback execution; the Telegram help message was delivered;
- `/image футуристический офис с ИИ-ассистентом` — successful image-generation execution; a PNG `1024x1024` was delivered to Telegram;
- `/start` — successful text-generation execution; the generated OpenAI response was delivered as plain text.

## Validation performed before live testing

The original workflow was also checked for:

- overall workflow validity;
- key node configuration validity;
- connection integrity between routing branches and output nodes.

## What was learned from the real runs

The live execution cycle exposed two implementation issues that static inspection alone did not surface:

- an unsupported OpenAI text verbosity parameter;
- an output-mapping problem between the OpenAI response object and Telegram text input.

Both were corrected before the final successful `/start` run.

This is relevant portfolio evidence because it demonstrates not only workflow assembly but also runtime debugging against real external APIs.

## Screenshots

The original project includes screenshots of:

- the full n8n workflow;
- preprocessing Expressions;
- Switch routing;
- Telegram chat results;
- n8n execution history.

The reusable repository intentionally does not embed environment-specific screenshot content until it has been reviewed for sensitive identifiers. A fresh deployment should capture equivalent screenshots using only test/demo data.
