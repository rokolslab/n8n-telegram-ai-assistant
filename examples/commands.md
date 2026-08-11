# Demo commands

Use these commands after importing the workflow and connecting test credentials.

## Text branch

```text
/start
```

Expected: a generated welcome message describing `/start` and `/image`.

## Image branch

```text
/image футуристический офис с ИИ-ассистентом
```

Expected: an image is generated and sent to Telegram with the prompt in the caption.

A shorter neutral test prompt:

```text
/image simple blue robot icon
```

## Fallback branch

```text
/help
```

Expected:

```text
Извините, поддерживаются команды /start и /image <описание>.
```

Any unsupported command should use the same fallback branch.
