
# Wanted Optimizer Pro

**A free, all-in-one Windows optimizer for gamers and power users: more than 100 tweaks, a live performance
dashboard, RAM and PC cleaning, Gaming Mode, and one-click undo, in a fast dark interface available in English and Arabic.**

Wanted Optimizer Pro is a Windows 10/11 (64-bit) desktop app. A C# / .NET 10 engine does the real work on your PC, and a
React + TypeScript interface (shown through Microsoft WebView2) puts it in one clean window. It is free to use; a
licence key, checked online each time the app starts, activates it.

> **Status:** version 1.0.0, first public release. The build and all 1,556 automated C# tests pass, and 113 of the 121
> tweaks were switched on and restored on a real Windows 10 PC. The installer and some screens are still being
> verified; see `docs/VERIFICATION_STATUS.md` and `docs/RELEASE_READINESS_REPORT.md` for exactly what has and has not been tested.

## Why it is different
* **Honest by design.** Every tweak card reads the real Windows state, applies the change, reads it back to confirm
  Windows kept it, and can put the original back. Nothing is a fake switch. If your hardware does not have a feature
  (for example AMD-only settings on an NVIDIA PC) the card says "Not found on this PC" and is disabled.
* **Reversible.** Changes are recorded before they are made. "Restore Defaults" and "Undo my tweaks" return settings to
  what they were, and "Undo my tweaks" works even without a licence.
* **Careful with risk.** Risky tweaks carry a Warning badge that explains what can be affected. The registry cleaner only
  touches startup entries whose program is definitely missing, backs them up first, and can restore them.
* **Private.** No account, no telemetry dashboard. Your settings, history and logs stay in `%LocalAppData%\WantedOptimizerPro`.

## What is inside
| Area | What you get |
|---|---|
| **Dashboard** | Live CPU, memory, disk and network performance with a chart, smart recommendations, **RAM Cleaner** (manual, or automatic when memory reaches a threshold you choose), **PC Cleaner**, and **Restore Points**. |
| **System Tuning** | Core, Windows interface, startup and boot, service, background task and storage tweaks. |
| **Performance** | CPU and timer, memory, power and latency tweaks. |
| **Hardware** | AMD and NVIDIA driver and service controls, shown only when that hardware is detected. |
| **Network** | General networking, DNS and adapter optimization. |
| **Privacy & Security** | Privacy, telemetry and Windows protection settings, each with its own warning where it matters. |
| **Gaming Mode** | One custom gaming profile you build from the tweaks. Switch it on from the title bar to apply it; switch it off to restore exactly what it changed. Export and import your selection as a file. |
| **Update Center** | Checks GitHub Releases, downloads the new version, verifies its SHA-256, replaces the install, and rolls back if the new version does not start. |
| **Settings** | Language (English / Arabic with full right-to-left layout), start minimized, launch at Windows start, licence status. |

The app runs with administrator rights (one Windows approval box at launch) so every tweak can apply; if you decline, it
still opens and asks again only when a tweak needs it.

## Highlights
* Dark interface with a blue accent, smooth animations, tray icon (closing hides to the tray), fixed-size window that
  scales down on small screens and 125% / 150% display scaling.
* Full **English and Arabic** translation, including right-to-left layout and an Arabic-optimised font.
* Live hardware sensors, with optional advanced CPU monitoring (installs the PawnIO driver only if you turn it on).
* Per-user installer: no administrator prompt to install, and in-app updates never need one either.

## Not included
Game Boost, HPET controls and Smart App Control were deliberately removed. The build is not code-signed yet, so Windows
SmartScreen warns on first run ("More info" > "Run anyway").

## Requirements

| Tool | Version |
|---|---|
| Windows | 10 or 11, x64 |
| .NET SDK | 10.0 (includes the Windows Desktop workload for WPF) |
| Node.js | 20.19 or newer (22 LTS recommended) |
| WebView2 Runtime | Preinstalled on Windows 11 and current Windows 10; otherwise <https://developer.microsoft.com/microsoft-edge/webview2/> |

## Run it

From the repository root in PowerShell:

```powershell
# Option A: one command, bundled interface
dotnet run --project src/Backend/WantedOptimizer.Host
```
`dotnet build` runs `npm install` (when needed) and `npm run build` automatically whenever `wwwroot/index.html` is missing or any file under `src/Frontend` is newer than it, so the interface can no longer go stale behind the C# build.
You can also refresh it by hand with `npm run build` in `src/Frontend` (or `.\scripts\build.ps1`).

```powershell
# Option B: development mode with hot reload (Vite dev server + host)
.\scripts\dev.ps1
```
Dev mode enables F12 DevTools and the context menu in the window. Both are disabled in bundled mode.

Other scripts: `.\scripts\build.ps1` (frontend bundle + `dotnet build -c Release`),
`.\scripts\publish.ps1` (framework-dependent x64 output in `artifacts\publish`).

Visual Studio 2022 17.14+ / Rider: open `WantedOptimizerPro.slnx`, set `WantedOptimizer.Host` as startup project, run.

### What you should see
A fixed-size window (designed for 1280x800; it scales down on smaller screens or 125%/150% display scaling and
cannot be resized or maximised) with a custom title bar, the brand mark, and a blue-accented sidebar of pages:
Dashboard, System Tuning, Performance, Hardware, Network, Privacy & Security, Gaming, plus Settings and Updates.
Closing the window hides it to the system tray; Exit from the tray menu quits.

Two settings on the Settings page: "Start minimized" (SQLite) and "Launch at Windows start", which writes one value
under `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` and only moves once Windows confirms it. Neither is
enabled on your behalf.

Data lives in `%LOCALAPPDATA%\WantedOptimizerPro\` (`data\wanted-optimizer.db`, `logs\`, `webview2\`).

## Layout

```
WantedOptimizerPro/
├─ WantedOptimizerPro.slnx            solution
├─ global.json                        pins the .NET 10 SDK
├─ Directory.Build.props              shared C# settings
├─ Directory.Packages.props           central NuGet versions
├─ scripts/                           dev.ps1, build.ps1, publish.ps1
└─ src/
   ├─ Backend/
   │  ├─ WantedOptimizer.Core/            contracts only: bridge protocol, IModule, abstractions (no Windows APIs)
   │  ├─ WantedOptimizer.Infrastructure/  SQLite + migrations, settings store, file logger, app paths
   │  ├─ WantedOptimizer.Modules.Shell/   the one built-in module (app info, ping, settings, module list)
   │  └─ WantedOptimizer.Host/            WPF + WebView2 executable, composition root (namespace WantedOptimizer.Desktop)
   └─ Frontend/                           Vite + React 19 + TypeScript + Tailwind v4 + Framer Motion
      ├─ public/assets/wanted.png         official brand mark, used as supplied
      └─ src/
         ├─ bridge/                       transport (client.ts) and typed channel facade (api.ts)
         ├─ catalog/                      the tweak catalogue: types.ts, tweaks.ts, pages.ts
         ├─ components/layout/            sidebar, header, navigation registry
         ├─ components/ui/                icons, logo, cards, tabs, toggle, warning badge, controls
         ├─ components/tweaks/            tweak card, shared catalogue page, demo state
         ├─ components/dashboard/         memory panel, live performance, chart, recommendations
         ├─ pages/                        one file per sidebar destination
         └─ shared/hooks/                 persisted settings, host connection, sidebar state
```

The catalogue is data, not markup: `catalog/tweaks.ts` holds one definition per tweak identity and
`catalog/pages.ts` lists which categories show which ids, in the specification's order. A tweak listed in
several tabs is referenced, never copied, so its description, badge and impact text stay identical
everywhere (Master Specification v2 §11). Five of the seven pages are the same `TweakCatalog` component
with different data.

Dependency direction: `Host → Modules.* → Core ← Infrastructure`. Only `Host` references WPF/WebView2.
The frontend never touches the file system or Windows; it can only call registered bridge channels.

## Frontend ↔ backend bridge

Transport: WebView2 `postMessage` (page to host) and `PostWebMessageAsJson` (host to page). All messages are JSON.

```jsonc
// page → host (request)
{ "id": "m1x-1", "channel": "settings.set", "payload": { "key": "ui.sidebarCollapsed", "value": "true" } }

// host → page (response)
{ "kind": "response", "id": "m1x-1", "ok": true,  "payload": { ... } }
{ "kind": "response", "id": "m1x-1", "ok": false, "error": { "code": "invalid_key", "message": "..." } }

// host → page (event, unsolicited)
{ "kind": "event", "channel": "settings.changed", "payload": { "key": "...", "value": "..." } }
```

Built-in channels: `app.getInfo`, `app.ping`, `settings.get`, `settings.getAll`, `settings.set`, `modules.list`; event `settings.changed`.

Safeguards already in place: the host ignores messages from any origin other than its own content,
blocks navigation away from it, runs handlers on worker threads, validates setting keys and sizes, never leaks
exception details to the page, and turns off DevTools and browser shortcuts outside dev mode.
In a normal browser tab `window.chrome.webview` does not exist, so the UI shows "Not connected to Windows" rather than invented data.

## Adding a module later

Backend (new project `WantedOptimizer.Modules.<Name>` referencing Core):
1. Implement `IModule` and register services plus handlers with `services.AddBridgeHandler<MyHandler>()`.
2. Each handler implements `IBridgeHandler` for one channel (`"<area>.<action>"`).
3. Add the module to `Host/ModuleCatalog.cs` and add the project reference and `.slnx` entry.
4. Need tables? Append a migration to `Infrastructure/Persistence/Migrations.cs`.

Frontend:
1. Add typed calls to `src/bridge/api.ts` (and payload types to `types.ts`).
2. Create `src/pages/<Name>Page.tsx`.
3. Add one entry to `src/components/layout/navigation.ts`; the sidebar, tooltips and page transitions
   pick it up.

Adding a tweak to an existing page needs no new component: add the definition to `src/catalog/tweaks.ts`
and its id to the right category in `src/catalog/pages.ts`. If it carries a warning badge, write its
"What can be affected" text on the definition, where every page that shows the tweak reads the same copy.

## Gaming Mode

A switch and a **SET** button in the window's title bar (left of the logo). SET opens the one Custom Gaming Profile; the switch applies it and, when turned off, restores exactly what it changed.

- **Where the list comes from.** `CustomizationService.Tools` (the Customization and Gaming catalogs) is the one registry of implemented optimizations. `GamingModeService.GetConfig` builds the list from it on every open, so a tool added to a catalog appears in Gaming Mode with no other change. A tool is offered only if it is an on/off setting, supported on this Windows, and readable. Tools that offer several choices (taskbar search, taskbar alignment) have no single "on" state and are not offered. The System Tuning, Performance, Hardware, Network and Privacy pages are presentation only today (no apply path exists), so they contribute nothing until they have real implementations on the same contract.
- **One state, one engine.** Gaming Mode never touches the registry itself. It calls `CustomizationService` (the same code the cards use), which reads state back from Windows after every write and publishes `customization.changed`. The Customization and Gaming pages subscribe to that event and re-read when the window regains focus, so a card always shows what Windows reports.
- **Records.** `gaming-profile.json` (ids only), `gaming-transactions.json` (one record per setting Gaming Mode changed: id, original state, applied state, status, backup kind, timestamp) and `gaming-registry-backup.json` (exact original registry values) in the data folder. Writes are atomic with a `.bak` copy, and an unreadable record blocks activation instead of being treated as empty. A record is written before the change and removed only after the restore reads back correctly.
- **Conflicts.** Turning it off restores a setting only if it still reads as Gaming Mode left it (and, where registry values are tracked, they still match). Otherwise it is reported and left alone until you choose "Restore the original" or "Keep my change".
- **Title bar.** The Windows caption is replaced by a strip drawn with `WindowChrome` (see `MainWindow.xaml`). The switch and SET only send `gamingMode.request` to the page, which runs the flow and shows the dialogs (`components/gamingMode/`).

## Update Center: configuration

Shipped configured for `wantedtn/Wanted-Optimizer-Pro` with `PackageAssetPattern` `WantedOptimizerPro-*-win-x64.zip`.
The `Updates` section of
`src/Backend/WantedOptimizer.Host/appsettings.json` (copied beside the exe) controls it:

| Key | Required | Meaning |
| --- | --- | --- |
| `Owner` | yes | GitHub user or organisation that publishes the releases. |
| `Repository` | yes | Repository name. |
| `PackageAssetPattern` | yes | Wildcard for the Windows package asset, e.g. `WantedOptimizerPro-*-win-x64.zip`. Exactly one `.zip` per release may match. |
| `ChecksumAssetPattern` | no | Wildcard for a checksum file (`<sha256>  <file name>` lines). Only used when GitHub gives no `digest` for the package. |
| `IncludePrereleases` | no | `true` also offers releases marked pre-release. Drafts are never offered. |

Each release needs:

- a tag that is a SemVer version greater than the app's `<Version>` in
  `Directory.Build.props` (`v1.2.0` and `1.2.0` both work);
- one `.zip` asset holding the published app (the files that sit beside
  `WantedOptimizerPro.exe`, either at the root of the zip or inside one top-level folder);
- a SHA-256: GitHub's own asset `digest`, or a checksum asset as above. A release with neither is not offered;
- the release notes in the body; they are what "What's New" shows.

If the installed exe is Authenticode-signed, the new exe must be signed by the same
publisher, or the update is refused. The updater never runs scripts from a release.
User data in `%LOCALAPPDATA%` (settings, SQLite database, logs) is never touched.

## Known gaps
* No Content-Security-Policy yet.
* Unsigned build (SmartScreen warning). Installer and updater are not yet verified on a clean Windows PC.
* Automated tests: C# xUnit projects under `tests/` and a browser harness under `tools/ui-harness`; the C# tests have
  not been run by the process that prepared this tree. Run `RUN CHECK.bat` / `dotnet test` on Windows.
* Elevation: by owner decision (2026-11-22) the app asks for administrator rights at every launch (one Windows approval box, `Integration/AlwaysElevated.cs`). The manifest stays `asInvoker`. If the box is declined, or the account is a standard one, the app runs unelevated and admin-only actions still use the UAC helper.

## Troubleshooting
- **"WebView2 Runtime not found"**: install the Evergreen runtime from the link above.
- **"The frontend bundle was not found"**: run `npm run build` in `src/Frontend`, or delete `wwwroot` and rebuild.
- **Blank window in dev mode**: the Vite server must be on port 5173 (it uses `strictPort`). Close whatever else uses it.
- **`npm` not found during `dotnet build`**: install Node.js, reopen the terminal, or build the frontend yourself and pass `-p:SkipFrontendBuild=true`.
- Logs: `%LOCALAPPDATA%\WantedOptimizerPro\logs\wop-<date>.log`.

### Gaming Mode selection, export and import

* There are no saved or named profiles. The popup edits one current selection of optimizations, stored by the host as soon as it changes in `%LocalAppData%\WantedOptimizerPro\gaming-profile.json` (atomic write, previous version kept as `.bak`). A file that exists but cannot be read blocks changes instead of resetting to empty. If an earlier build left `gaming-profiles.json` and there is no selection file yet, its active profile is carried over once by copy; that file is never written again.
* **Export Settings** opens a popup with a name (only a label for the file) and a preview of the current selection, then the Windows Save dialog (suggested `GamingMode_<Name>.json`). The file is `{ format, formatVersion: 1, exportedAt, profile: { name, optimizations: [{ id, name, category }] } }` and is verified after writing. It contains no profile key, backups, transaction records or machine data.
* **Import Settings** opens the Windows Open dialog, validates format, version and every id against the live catalog, previews the result, and after confirmation replaces the current selection with the available tweaks. It never applies anything to Windows.
* Tests: `tools/ui-harness/exportimport.mjs`, `sync.mjs`, `corrections.mjs` (browser harness with a stand-in host; the C# is not exercised there).

## Packaging
`powershell -ExecutionPolicy Bypass -File scripts\package.ps1` builds the installer and the update zip. See
`docs/INSTALLER_RELEASE.md`. Third-party components are listed in `THIRD_PARTY_NOTICES.md`.
