# Changelog

All notable changes to `asignua/filament-chat-pro` are documented here.

## v1.0.1 - 2026-10-08

- Licence: the commercial licence terms (LICENSE.md), prices in the README, screenshots and a cover in `art/`.
- Efficiency/fix: deleting a message no longer loads every chat bell of the conversation: a SQL prefilter on the stored (JSON-escaped) body plus the conversation ulid and chat types, iterated with `lazyById()`, notifiables eager-loaded, locale list and title patterns memoized per `delete()`. A (host-overridden) title template without `:who` is skipped, so it can no longer delete another author's identical-body bell.
- Fix: deleting a message finds bells written in another locale (sender's request locale or the recipient's `preferredLocale()`): the free chat's lang directory was resolved one level too high (`src/resources/lang`), so only the app/fallback locale was tried. It now comes from the translator's `filament-chat` namespace hint (fallback: the package root), plus host overrides in `lang/vendor/filament-chat` and the recipient's preferred locale.
- Fix: `DB::afterRollBack()` is used only where the framework has it (early Laravel 12.x); a failed disk write (`storeAs()` returning `false`) now refuses the message with a notification instead of saving an attachment row for a missing file (new `error_store` string, all ten locales).
- Fix: the bell author check is an exact match of the free chat's title templates (all locales, `:title` = anything), not a substring: a group title containing the author's name or a longer name (`Ann`/`Anna`) no longer deletes another author's bell.
- Fix: deleting a message removes its own bell notifications by identity, with no time windows (they deleted the bells of other messages: a reply in the same second, a later mention inside the edit window). A bell goes only if it is a chat notification type, addressed to a member, linked to this conversation, its title names the message's author and its body equals the message's current preview exactly. Safe-side limits: the bell made by the ORIGINAL text of an edited message (not stored anywhere) and a bell of an author renamed since stay; an identical text by the same author in the same conversation is also removed. Owner follow-up: the clean fix is to add the message ulid to the free filament-chat notification data (`NewMessageNotification`/`MentionNotification`); then match on it and drop these limits. A moderator who left a group cannot delete messages written after leaving.
- Fix: an upload still running when the conversation changes is cancelled in the browser (every running batch, through the component's `$wire` resolved in `init()`: the lazy `$wire` proxy looks its component up on the first property read, too late in `destroy()`) and refused on the server (the server guard remembers only the last conversation left, so after A-B-A a file of an upload that survived is dropped rather than attached); Send stays blocked until every running batch has finished.
- Fix: the Pro window/model check runs again in `boot()`, so plugin order no longer lets a custom class slip past it.
- Hardening: the stored file's extension comes from the sniffed content type; a client extension such as `.php`/`.html`/`.svg` is never kept.
- Efficiency: window search loads only ulids (`MessageSearchRepository::searchUlids()`); `path` of the attachments table is indexed (edit the create migration; hosts that already migrated can add `$table->index('path')`).
- Docs: README paste behaviour matches the code.

## v1.0.0 - 2026-10-07

First release. Requires `asignua/filament-chat` `^1.3`.

- Attachments: the paperclip, drag & drop, several files, upload progress, preview chips with a remove button; messages may be only files. Private disk, limits (size, count, allowed extensions and sniffed content types). Thumbnails and a keyboard-driven lightbox for pictures, cards for other files. Delivery route with a conversation-membership check; only raster images inline, everything else a download with `nosniff` and a sandbox CSP.
- Clipboard paste of screenshots and files into the composer (`screenshot-YYYYMMDD-HHMMSS.png`); text paste is untouched.
- Deleting own messages (soft delete, optional time window, `canDeleteUsing()` for moderators, `Gate` ability `delete`); the files are removed and the members' feeds refresh.
- Typing indicator over a private whisper channel (real-time only).
- Message search across all conversations and inside the open one.
- Hardening: deleting erases the text, record reference, mentions and the bell notifications (`delete.keep_text` for audit hosts); downloads need a chat user allowed into the panel; content types are sniffed with libmagic from the first 8 KB of the temp stream; files are written and removed in step with the transaction (`filament-chat-pro:prune-orphans`); pending files are dropped when the conversation changes; Send is blocked during an upload; search runs only when the term changes.
- Ten languages, dark mode, a Boost skill and guideline, `filament-chat-pro:install`.
