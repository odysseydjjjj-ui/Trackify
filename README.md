# Trackify

A native desktop music app built with C# / .NET 8 and Avalonia UI (no WebView). The backend is Supabase: Auth (Google), Postgres, Storage. Windows today; Android and iOS/web clients can use the same server.

## Features

- **TrackSound library**: the app's own catalog inside Supabase. Streaming, queue, shuffle/repeat, likes, timed lyrics, playlists.
- **Studio** (Pro and up): upload tracks with cover and `.lrc` lyrics, allow paid downloads.
- **Tokens (§)**: earned by listening, spent on plans, gifts, downloads and usernames. All balance changes are atomic server-side functions with idempotency keys.
- **Plans** Free / Pro / Max / Ultra (3-day paid plans), enforced on the server.
- **Catalog**: gifts (silent MP4 loops via LibVLC), Unique gifts with a glow on the profile banner, resale market, usernames market.
- **Forum and chats**: link and insult filter with timed mutes, friends and direct messages.
- **Games**: Snake, 2048, Memory Match, Beat Tap, with a server leaderboard.
- **Profile**: avatar crop and upload, banner color and emoji, rings, several usernames (Ultra).
- **Console** (staff only, checked on the server): users, mutes, bans, check marks, roles, gifts, forum filter, settings, audit journal.
- **Updates**: Velopack + GitHub Releases.

## Requirements

- .NET 8 SDK
- A Supabase project with the Google provider enabled
- Windows 10/11 (the first supported platform)

## Setup

1. **Config.** Copy `config.example.json` to `config.json` (next to the exe or in `%AppData%\Trackify`) and fill in the Supabase URL and the **publishable (anon) key**. Never put a service-role key into the app or into git.
2. **Supabase redirect.** Authentication -> URL Configuration -> add `http://127.0.0.1:53682/callback`.
3. **Database.** In the Supabase SQL Editor run, in this order:
   `Server/001_trackify.sql`, `002_schema_rls.sql`, `002_verify.sql`, `003_rpc.sql`, `003_verify.sql`, `004_market.sql`, `005_hardening.sql`.
   Every `*_verify.sql` must end with all checks `ok`.
4. **Staff.** Put the Google emails of the staff accounts into `public.staff_allowlist` (the table is seeded with placeholders). Rights are granted on sign-in by confirmed email, never by username.
5. **Run.** `dotnet run`

## Build and release

```powershell
dotnet tool install -g vpk --version 0.0.1297
dotnet publish -c Release -r win-x64 --self-contained -o publish
vpk pack --packId Trackify --packVersion 0.1.0 --packDir publish --mainExe Trackify.exe
```

Upload everything from `Releases/` (Setup.exe, `*-full.nupkg`, `*-delta.nupkg`, `releases.win.json`) to a GitHub **Release** (not a pre-release) with the tag `v0.1.0`. The repository must be public. Installed apps check for updates on start and from Settings. To ship a new version raise `<Version>` in `Trackify.csproj` and repeat. Set `UpdateRepo` in the config to `owner/repo`.

## Security model

- The client only holds the publishable key. Row Level Security is on for every table; money, prices, limits, mutes and bans are decided by `SECURITY DEFINER` functions (`Server/003_rpc.sql`).
- Sessions are stored with Windows DPAPI. Tokens are never logged. Redirects are disabled for API calls.
- Known limits: paid downloads are an app-level rule, not DRM (streaming and downloading read the same file); listening rewards are measured by the server but idle listening is possible; the forum filter is a word list; game anti-cheat checks plausibility only.

## Project layout

```
Controls/   player, cards, gift video, built-in games
Models/     data models and settings
Services/   Supabase client, auth, audio, updates
Views/      pages and windows
Server/     SQL for Supabase (run in order, see Setup)
```
