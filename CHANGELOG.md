# Changelog

All notable changes to `asignua/filament-chat-pro` are documented here.

## v1.0.0 - unreleased

First release. Requires `asignua/filament-chat` `^1.3`.

- Attachments: the paperclip, drag & drop, several files, upload progress, preview chips with a remove button; messages may be only files. Private disk, limits (size, count, allowed extensions and sniffed content types). Thumbnails and a keyboard-driven lightbox for pictures, cards for other files. Delivery route with a conversation-membership check; only raster images inline, everything else a download with `nosniff` and a sandbox CSP.
- Clipboard paste of screenshots and files into the composer (`screenshot-YYYYMMDD-HHMMSS.png`); text paste is untouched.
- Deleting own messages (soft delete, optional time window, `canDeleteUsing()` for moderators, `Gate` ability `delete`); the files are removed and the members' feeds refresh.
- Typing indicator over a private whisper channel (real-time only).
- Message search across all conversations and inside the open one.
- Hardening: deleting erases the text, record reference, mentions and the bell notifications (`delete.keep_text` for audit hosts); downloads need a chat user allowed into the panel; content types are sniffed with libmagic from the first 8 KB of the temp stream; files are written and removed in step with the transaction (`filament-chat-pro:prune-orphans`); pending files are dropped when the conversation changes; Send is blocked during an upload; search runs only when the term changes.
- Ten languages, dark mode, a Boost skill and guideline, `filament-chat-pro:install`.
