# Filament Chat Pro

> **Documentation only.** Filament Chat Pro is a commercial plugin: this public repository holds its
> documentation, changelog and licence terms. The package itself is installed from the private Composer
> repository you get with a licence — [buy one on Anystack](https://checkout.anystack.sh/filament-chat-pro).
> Questions: support@asign.in.ua.

<img class="filament-hidden" src="https://raw.githubusercontent.com/asignua/filament-chat-pro-docs/main/art/cover.jpg" alt="Filament Chat Pro">


The paid add-on for [Filament Chat](https://github.com/asignua/filament-chat), the team chat for Filament 5 panels. It turns the
chat into a place where work happens: **send files** (drag & drop, the paperclip, **paste a screenshot** straight from the
clipboard), look at pictures in a **lightbox**, **delete your own messages**, see **«Olga is typing…»** and **search every
message** you have ever been part of.

It is built on the free chat's extension seams — a window subclass, render hooks and a few protected methods — not on a fork, so
the free package keeps updating on its own. Dark mode, ten languages, and every feature is a switch.

## Screenshots

![A conversation with images, files and a deleted message](https://raw.githubusercontent.com/asignua/filament-chat-pro-docs/main/art/chat-page.jpg)

A screenshot pasted straight from the clipboard waits as a chip until you send it:

![Pasting a screenshot](https://raw.githubusercontent.com/asignua/filament-chat-pro-docs/main/art/paste.jpg)

Pictures open in a lightbox; files download with one click:

![The lightbox](https://raw.githubusercontent.com/asignua/filament-chat-pro-docs/main/art/lightbox.jpg)

Search every message you have been part of, across conversations:

![Searching messages](https://raw.githubusercontent.com/asignua/filament-chat-pro-docs/main/art/search.jpg)

Dark mode:

![Dark mode](https://raw.githubusercontent.com/asignua/filament-chat-pro-docs/main/art/chat-page-dark.jpg)

## Purchase

Filament Chat Pro is a commercial plugin. Both tiers include one year of updates; after that the
plugin keeps working on the last release you received, and you can renew for further updates.

| Tier | Projects | Activations | Price | Renewal |
| --- | --- | --- | --- | --- |
| Single Project | 1 | up to 3 (production, staging, local) | €69 | €35 / year |
| Unlimited | any number, SaaS included | unlimited | €169 | €85 / year |

Refunds are available within 14 days of purchase.

**[Buy a licence on Anystack](https://checkout.anystack.sh/filament-chat-pro)** — after the purchase you receive a licence key and
access to the private Composer repository. See [LICENSE.md](https://github.com/asignua/filament-chat-pro-docs/blob/main/LICENSE.md) for the licence terms.

## Requirements

- PHP 8.3+
- Laravel 12 or 13
- Filament 5
- [`asignua/filament-chat`](https://github.com/asignua/filament-chat) `^1.3` (installed with it as a dependency)
- Real-time chat over Reverb or Pusher for the typing indicator (everything else works with polling)

## Installation

1. **Add the private repository** (the same for every customer):

   ```bash
   composer config repositories.filament-chat-pro composer https://filament-chat-pro.composer.sh
   ```

2. **Add your credentials.** The username is the e-mail address you bought the licence with; the
   password is your licence key. A **Single Project** licence is bound to a fingerprint, so the password
   is the key followed by a colon and the fingerprint you activated — usually the production domain:

   ```bash
   # Single Project
   composer config --auth http-basic.filament-chat-pro.composer.sh you@example.com "LICENCE-KEY:shop.example.com"

   # Unlimited
   composer config --auth http-basic.filament-chat-pro.composer.sh you@example.com "LICENCE-KEY"
   ```

   This writes `auth.json` next to your `composer.json` — keep it out of git (add `auth.json` to `.gitignore`).
   On CI and servers pass the same JSON through the `COMPOSER_AUTH` environment variable instead.
   A Single Project licence allows three activations (for example production, staging and local); manage
   them in your Anystack account.

3. **Require the package:**

   ```bash
   composer require asignua/filament-chat-pro
   ```

4. **Run the installer** — it publishes the config and the two migrations (a new `chat_attachments` table and `deleted_at` on the chat messages):

   ```bash
   php artisan filament-chat-pro:install --migrate
   ```

5. **Register both plugins** on your panel:

   ```php
   use Asignua\FilamentChat\FilamentChatPlugin;
   use Asignua\FilamentChatPro\ChatProPlugin;

   $panel
       ->plugin(FilamentChatPlugin::make())
       ->plugin(ChatProPlugin::make());
   ```

   That is all: Chat Pro mounts its own chat window instead of the stock one and switches the chat's message model to its own
   (the same table, plus soft deletes and attachments). Without `FilamentChatPlugin` on the same panel the panel refuses to boot
   with a message saying so.

   **Register the chat on one panel only** (the same for Chat Pro): notification links, the private channels and the attachment route assume
   a single chat panel. The download route finds the signed-in person by trying the guard of every panel that carries the chat plugin.

   If your project **published** the free chat's `pages/chat.blade.php` or `chat-window` view: the published page keeps mounting the stock
   window (it ignores `ui.window_component`) and a published `chat-window` view has none of the hook places — re-publish or merge them, or
   Chat Pro's features will not appear.

   If the project commits published assets (`public/css`), run `php artisan filament:assets` after installing and after upgrades.

## Quick start

```php
$panel
    ->plugin(FilamentChatPlugin::make()->users(fn (Builder $query) => $query->where('is_active', true)))
    ->plugin(
        ChatProPlugin::make()
            ->attachmentDisk('s3', 'chat')            // a PRIVATE disk; default: "local"
            ->attachmentLimits(maxFileSizeKb: 20480, maxFiles: 10)
            ->deleting(window: 15)                     // authors may delete for 15 minutes
            ->canDeleteUsing(fn (User $user, Message $message): bool => $user->isModerator())
    );
```

Every setter writes the matching key of `config/filament-chat-pro.php` (publish it with the installer); what the plugin does not set
comes from the config. Closures (`canDeleteUsing`) live on the plugin only, so the config stays cacheable.

| Feature | Config | Plugin |
|---|---|---|
| attachments | `attachments.enabled` | `->attachments(false)` |
| pasting from the clipboard | `attachments.paste` | `->paste(false)` |
| disk / folder | `attachments.disk` / `.directory` | `->attachmentDisk('s3', 'chat')` |
| limits | `attachments.max_file_size` (KB) / `.max_files` | `->attachmentLimits(10240, 10)` |
| allowed files | `attachments.mimes` / `.extensions` | `->allowedFiles($mimes, $extensions)` |
| deleting, time window (minutes), keep the text | `delete.enabled` / `delete.window` / `delete.keep_text` | `->deleting(true, 15, keepText: false)` |
| who else may delete | — | `->canDeleteUsing(fn ($user, $message) => …)` |
| typing indicator | `typing.enabled` / `.throttle` / `.expire` | `->typing(false)` |
| search | `search.enabled` / `.min_length` / `.limit` | `->search(false)` |
| download route | `routes.prefix` / `routes.middleware` | — |
| attachments table | `tables.attachments` | — (set it before migrating) |

## Features

### Attachments

- A **paperclip** next to the send button, **drag & drop** of files onto the conversation (a dragged link to a record still
  attaches the record — the free chat's feature is untouched), several files at once, an **upload progress** bar, and **preview chips**
  above the input where a file can be removed before sending.
- A message may be **only files** — the text can stay empty.
- In the feed pictures are **thumbnails** (one large, several in a grid) that open a **lightbox** (Esc closes, the arrow keys browse,
  a download link); everything else is a **card** with an icon, the name and the size.
- Checks happen when the file is picked, so a refused file disappears at once with a notification — and again when the message is
  sent, so nothing slips in through a race: **size** (`max_file_size`, KB), **count** per message, and an **allow-list**. A file must
  pass both lists: `extensions` (the name) and `mimes` (the content type **sniffed from the file itself**, never the browser's claim;
  `image/*`-style wildcards work). The defaults allow pictures, PDF, Office and OpenDocument files, zip, txt and csv; `null` for a list
  means "anything". SVG and HTML are not on the defaults.
- Files are written to a **private disk** (`attachments.disk`, default `local`), under `directory/Y/m/<ulid>.<ext>` — the client's
  file name only lives in the database and is cleaned (no paths, no control characters, 150 characters at most).

The width and height of a picture are stored, so the feed reserves its space before it loads.

### Clipboard paste

Press Ctrl/Cmd+V in the composer with a screenshot (or files copied in the file manager) on the clipboard: the files join the
pending uploads. A pasted picture has no real name — browsers call it `image.png` — so it is named `screenshot-YYYYMMDD-HHMMSS.png`
(the server applies the same rule as a safety net). **Default is prevented only when files were taken**: pasting text works exactly as
before, and a paste that carries text plus only an unnamed picture of it (a cell copied from a spreadsheet, a web page) pastes the text; files with real names, or a screenshot with no text, are attached.

### Deleting your own messages

A trash button in the message menu opens a confirmation (Filament's modal). The message is **soft-deleted**: everyone sees «Message
deleted» in its place (a quote of it too, and the unread counters follow), and the message is **really erased**: in the same transaction its
text, record reference and mentions are blanked in the row, its files and rows are removed, and the bell notifications that quoted it are
deleted (they carry no message id, so they are matched by identity, never by time: chat notification type, member, this conversation's link, the author named in the title, and the exact quoted preview). The other members' feeds refresh
through the chat's own channel. Hosts that need an audit trail set `delete.keep_text` (`->deleting(keepText: true)`): the row then keeps its
content — readable by anyone with database access. In the conversation list a deleted latest message gives way to the previous one (for groups
you left too).

- Who: the author, while still a member of the conversation, inside `delete.window` minutes (`null` — any time).
- Moderators: `->canDeleteUsing(fn ($user, $message): bool => …)`. It is asked for every message the menu shows, so keep it cheap;
  a closure "yes" still needs the person to be able to read the conversation.
- `Gate::allows('delete', $message)` follows the same rule, for your own code.
- Editing stays the free chat's rule.

### Typing indicator

While someone types, the others see «Olga is typing…», «Olga and Ivan are typing…» or «3 people are typing…» above the input; it
disappears after `typing.expire` seconds (6) without a new signal or when a message arrives. The signal is a client **whisper** over
the private channel `filament-chat.conversation.{conversationUlid}`, at most one per `typing.throttle` seconds (3), authorised for the
conversation's **current** members only. The whisper carries just a member key and the names come from the server-rendered page, so a
whisper cannot inject text. It is **not authenticated beyond membership**: Pusher/Reverb client events do not carry a verified sender, so
any member can whisper on behalf of another member's key — fine for a «typing» hint, never a basis for any decision. It leaves the channel
when you switch conversation.

**Real-time only.** It needs the chat running over Reverb/Pusher (`FILAMENT_CHAT_REALTIME=true`). With polling the feature renders
nothing — there is no channel to whisper on.

### Message search

- A box above the conversation list searches **all your conversations**: hits show the conversation, the author, the date and a
  snippet with the match highlighted; a click opens the conversation and jumps to the message.
- A magnifier in the conversation header opens a bar at the top of the feed that searches **this** conversation only.
- The query runs only when the term changes; the hits are kept as ids and re-read (with the membership check) on every render, so polls
  and re-renders never search again. A plain case-insensitive `LOWER(body) LIKE` on the text (`%`, `_` in what you type are literal);
  SQLite's `LOWER` only folds ASCII, so on SQLite non-Latin text is matched case-sensitively (MySQL, MariaDB and PostgreSQL are fine), at least `search.min_length` characters (2),
  at most `search.limit` hits (30), newest first. Deleted messages are skipped; in a group you left you find only what was written
  before you left. Queries live in `MessageSearchRepository`. For millions of messages bring your own engine.

## Realtime notes

| `FILAMENT_CHAT_REALTIME` | What changes with Chat Pro |
|---|---|
| `true` (Reverb, Pusher, Ably) | everything, including the typing indicator |
| `false` / polling | everything except the typing indicator |

- **Whispers are client events.** Reverb supports them out of the box; on Pusher Channels enable *Client events* in the app settings
  (they only work on private/presence channels — ours is private). Without it the indicator silently never shows.
- The channel needs `/broadcasting/auth`; the free chat registers it when your app has none.
- Chat Pro uses the `window.Echo` the free chat creates (`realtime.echo`); nothing extra to install in the browser.

## Security

- **Private disk, no direct links.** Attachments are never reachable through a public URL. They are served by
  `GET /filament-chat-pro/attachments/{ulid}` after checking that the signed-in person (from the **chat panel's guard**) can see the
  conversation — admins included: a conversation is its members'. Someone who left a group keeps the history but gets no file sent
  after leaving. A guest is redirected to the panel login (403 when the panel has none); an unknown id or a missing file is 404.
- **Only raster pictures (JPEG, PNG, GIF, WebP, AVIF) are shown inline.** SVG, HTML and every document are always sent as
  `Content-Disposition: attachment`, with `X-Content-Type-Options: nosniff` and a locked-down `Content-Security-Policy: sandbox`, so an
  uploaded file can never run in your origin. Add `svg` to the allow-list if you must — it will still only ever download.
- The file name goes through `filename*` (RFC 5987) with an ASCII fallback; the stored name is the ulid, never the client's, and its extension comes from the sniffed content type (the client's only if it is plain and not `php`, `html`, `svg`, `js` and the like; otherwise `.bin`).
- Content types are sniffed server-side; both the extension and the type must be on the allow-list.
- Who may download: a member of the conversation who is still in the chat's `->users()` set and allowed into the panel
  (`canAccessPanel`). A deactivated or excluded person gets 403.
- Sniffing reads the first 8 KB from the temporary file's stream (also on a remote temp disk) with libmagic. Formats whose type is
  decided deeper into the file read as a generic type (an Office file as `application/zip`) — the default list includes those.
- Names, snippets and file names are escaped everywhere; a search hit is built piece by piece with only `<mark>` as markup.
- The delete button is only an offer: the action re-checks the rule, the conversation and the message on the server. The ulid is
  looked up inside the open conversation, so a stranger's message cannot be reached by guessing.
- Add throttling to the download route if you need it: `routes.middleware` = `['web', 'throttle:120,1']`.

## Gotchas

- **The `notifications.data` column must stay Laravel's stock `text`.** The bell cleanup looks the body up with `LIKE` on the stored JSON; a MySQL `json` or PostgreSQL `jsonb` column re-serialises the value (spaces, raw unicode), so nothing would match and the bells would keep the deleted text.
- **Deleting a message removes its bell notifications by text identity.** The free chat's notification data has no message id, so a bell
  is deleted when it is a chat notification of this conversation, its title is a free-chat title template filled with exactly the message's author (any locale: the request's, the fallback, the recipient's preferred one, every shipped or host-overridden one; the group title may be anything) and its body equals the message's
  preview exactly. Consequences: two identical texts by the same author in the same conversation lose both bells; the bell of an
  edited message's original text and bells whose author was renamed since stay. Follow-up for the owner: add the message ulid to the
  free `filament-chat` notification data, then match on it.

- **Both migrations are required** even if you switch a feature off: the message model gains `SoftDeletes` and an `attachments`
  relation, and every query then filters on `deleted_at`.
- **Livewire's upload limit applies first.** `livewire.temporary_file_upload.rules` defaults to 12 MB; raising `max_file_size` above
  it needs that setting raised too (and PHP's `upload_max_filesize` / `post_max_size`, and nginx `client_max_body_size`).
- **The message model and the window are swapped for Chat Pro's own** (`filament-chat.models.message`, `filament-chat.ui.window_component`)
  when the plugin registers — but only the free defaults are replaced. Your own subclass of `Asignua\FilamentChatPro\Models\Message` /
  `Livewire\ProChatWindow` is kept (and gets the Pro policy); any other class there makes registration throw a clear exception.
- Drag & drop of files stops at the conversation: the free chat's «drop a record link» overlay does not appear for files.
- A message that is only files cannot be saved with an empty text in the editor (the free chat refuses empty edits) — add a text or
  cancel the edit.
- A deleted message that was the latest of a conversation drops out of the list preview (the previous message shows) instead of
  «Message deleted» — for groups you left too; quotes and the feed do show the tombstone.
- **Temporary files.** Chat Pro deletes its temporary uploads once the message is stored. Whatever stays behind (a picked file never sent)
  is Livewire's `livewire-tmp` folder: Livewire cleans it after 24 hours on a local disk; on S3 add a lifecycle rule. Orphaned *stored* files
  (a crash between writing and committing) are removed by `php artisan filament-chat-pro:prune-orphans [--days=1] [--dry-run]` — schedule
  it daily.
- While a file is uploading, Send and Enter are blocked (with a hint); pending files are dropped when you switch conversation.
- The download route is registered when routes load: set `routes.prefix` / `routes.middleware` in the config, not in the plugin.
- The `uploads` property of the window is writable from the browser like any Livewire property; anything that is not a signed
  temporary upload is ignored.

## Upgrading notes

- Chat Pro 1.x needs `asignua/filament-chat` `^1.3` (the extension seams). Upgrade the free chat first.
- Moving from the free chat to Chat Pro on an existing installation: install, run the migrations, add the plugin. Existing messages
  are unaffected; the `deleted_at` column is nullable.
- `ChatService::send()` (free 1.3) has a new optional fourth parameter; a published `pages/chat.blade.php` keeps the stock window and a
  published `chat-window` view has no hook places (see Installation).
- If your app rebinds `MessageRepository`, copy the optional trailing `?Closure $scope` of its read methods (free 1.3).

## Usage from code

```php
use Asignua\FilamentChatPro\Models\Message;
use Asignua\FilamentChatPro\Services\MessageDeletionService;
use Asignua\FilamentChatPro\Repositories\MessageSearchRepository;

app(MessageDeletionService::class)->delete($message, $user);          // throws InvalidArgumentException when not allowed
app(MessageSearchRepository::class)->search($user, 'invoice');        // Collection<Message>, newest first
$message->attachments;                                                // Collection<Attachment>; $attachment->url(download: true)
Gate::allows('delete', $message);
```

Public extension points: `ChatProPlugin` (the setters above), `Livewire\ProChatWindow` (extend it and set it as
`filament-chat.ui.window_component`), `Models\Message`, `Models\Attachment`, `Services\AttachmentService`,
`Services\MessageDeletionService`, `Repositories\MessageSearchRepository`, `Support\TypingChannel`, and the route
`filament-chat-pro.attachment`.

## Translations

The interface ships in English, Ukrainian, German, Spanish, French, Italian, Dutch, Polish, Brazilian Portuguese and Turkish
under the `filament-chat-pro::filament-chat-pro` namespace. A test keeps every language in step with the English keys and
placeholders. Override a string by publishing the translations (`--tag=filament-chat-pro-translations`) and editing the copy in
`lang/vendor/filament-chat-pro`.

## Styles

The stylesheet (`resources/dist/filament-chat-pro.css`, Tailwind v4, dark mode) is linked after the panel's theme like the free chat's;
no custom theme changes are needed. Rebuild it with `npm install && npm run build` when you change the views.

## AI agents

The package ships [Laravel Boost](https://laravel.com/docs/boost) guidelines (`resources/boost/guidelines/core.blade.php`) and a skill
(`php artisan filament-chat-pro:install --skill`) so a coding agent wires it up correctly.

## Testing

```bash
composer install
vendor/bin/phpunit
vendor/bin/phpstan analyse --memory-limit=1G
vendor/bin/pint --test
```

Trying it by hand (the workbench panel with both plugins, three seeded users, SQLite):

```bash
vendor/bin/testbench workbench:build
vendor/bin/testbench db:seed --class='Workbench\Database\Seeders\DatabaseSeeder'
vendor/bin/testbench serve --host=0.0.0.0 --port=8000     # /admin — alice@example.com, bob@, carol@ / password
```

Chat runs on polling there (`FILAMENT_CHAT_REALTIME=false` in `testbench.yaml`); point it at a Reverb to see the typing indicator.

The browser parts (drag & drop, paste, the lightbox, the typing whispers) are JavaScript inside Blade and are not covered by the PHP
suite; the server side of each is.

## Changelog

See [CHANGELOG.md](CHANGELOG.md).

## License

Filament Chat Pro is commercial software. See [LICENSE.md](https://github.com/asignua/filament-chat-pro-docs/blob/main/LICENSE.md).
