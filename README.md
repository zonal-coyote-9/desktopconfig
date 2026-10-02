# desktopconfigs
Scripts and tools related to new system configuration.

## Overview

This repo holds first-run setup, configuration, security-hardening, and maintenance scripts for three desktop platforms — [Fedora](#fedora), [Mac](#mac), and [Windows](#windows) — plus cross-platform utilities in [Git/](#git) and [Functions/](#functions).

Every platform follows the same general shape:

1. **Setup** — one-time, run right after a fresh OS install. Installs apps and baseline settings.
2. **Configure** — one-time, applies OS preference tweaks.
3. **Secure** — optional, run any time after setup. Reversible hardening in independently-selectable categories, with harden / report / restore-defaults modes.
4. **Update** — recurring, run any time to pull in OS/package updates. Not a first-run step.

Each folder has its own README with full usage for every script: [Fedora](Fedora/README.md), [Mac](Mac/README.md), [Windows](Windows/README.md), [Git](Git/README.md), [Functions](Functions/README.md). The platform READMEs are also included in each platform's release download.

Every Fedora, Mac and Windows script shows its help when run with no options and only makes changes once you pass `--run` / `-Run` (or the script's own options).

All setup/secure/update scripts are written to be idempotent — safe to re-run if interrupted or to pick up where a previous run left off (e.g. after a required reboot).

## Fedora

Written for Fedora Workstation (GNOME). Requires `dnf`.

- **[01.Setup-Fedora.sh](Fedora/01.Setup-Fedora.sh)** — first-run setup. Run as your normal user (it calls `sudo` internally for the steps that need it):
  ```bash
  chmod +x 01.Setup-Fedora.sh
  ./01.Setup-Fedora.sh --run
  ```
  Sets the hostname, adds RPM Fusion repos, and upgrades the system, then presents the same per-category app menu as the Windows setup (Comms, Media, Office, Utilities, Dev, Creative, Others, AI) and installs your picks from Flathub/dnf. Claude and ChatGPT have no official Linux desktop apps, so the AI menu adds app-grid launchers for their web apps, plus Claude Code via Anthropic's official installer. After that it runs unattended: fonts, codecs, AMD drivers, and GNOME Tweaks/extensions. The menu comes after the upgrade's reboot check, so a fresh install that needs to reboot only asks once. Pass `--install-all` to skip the menus and install every app, and `--machine-name NAME` to set the hostname without being asked. If a kernel update needs a reboot, it reboots itself after a 10-second countdown — just re-run the script afterward to pick up where it left off.

- **[02.Configure-Fedora.sh](Fedora/02.Configure-Fedora.sh)** — first-run preference tweaks, run after setup as your normal user from a terminal inside your GNOME session: window buttons, 24-hour clock, Ptyxis terminal opacity, stops GNOME Software autostarting at login, and turns on AirPlay speaker discovery permanently (via Fedora's `pipewire-config-raop` package on Fedora 43+, or a per-user PipeWire config file on older releases).
- **[05.Secure-Fedora.sh](Fedora/05.Secure-Fedora.sh)** — optional security hardening, run any time after setup. Must be run as root:
  ```bash
  sudo ./05.Secure-Fedora.sh --all
  ```
  Categories include firewall, SELinux, LUKS (report-only — can't be enabled after install without reformatting), SSH (disabled unless `--keep-ssh`), network discovery, sudo, services, lockscreen, auditd, credential/password policy, kernel attack-surface sysctls, privacy, automatic security updates, Firefox policy hardening, and Sysmon for Linux, plus opt-in `--privacy-strict`/`--firefox-strict` categories that can break camera/mic or plain-HTTP sites, and an opt-in `--separate-admin` that creates a separate admin account and removes you from `wheel` (it double-checks the new account works before demoting you). Every category supports three modes: harden (default), `--report` (prints Expected vs Found, changes nothing), and `--restore-defaults` (reverts to Fedora's stock settings). Run `--help` for the full category list, or `--all --report` first to see what would change.

- **[04.Update-Fedora.sh](Fedora/04.Update-Fedora.sh)** — recurring maintenance, not a first-run step: `dnf update`, Flatpak app updates, firmware updates via `fwupdmgr`, cache cleanup, orphan removal, then reboots if needed.

## Mac

Requires Python 3. Run scripts with `python3 <script>.py` or `./​<script>.py` (all are executable). Scripts that need admin rights ask for your password once at the start and then run unattended. Each script is self-contained, so any one can be copied and run on its own.

- **[01.Setup-Mac.py](Mac/01.Setup-Mac.py)** — first-run setup: sets the computer name, installs Xcode Command Line Tools, installs Homebrew, then presents the same per-category app menu as the Windows setup (Comms, Media, Office, Utilities, Dev, Creative, Others, AI, plus a Mac-only CLI Tools menu for the shell aliases) and installs your picks via Homebrew or the Mac App Store (`mas`). All menus are answered up front, so the installs run unattended. Pass `--install-all` to skip the menus and install every app, and `--machine-name NAME` to set the computer name without being asked.
- **[02.Configure-Mac.py](Mac/02.Configure-Mac.py)** — first-run preference tweaks: Dock/Finder/trackpad/Terminal settings, enables the Application Firewall, turns on automatic software updates, installs the repo's Terminal profile and `.zshrc`/PowerShell profile dotfiles, and restarts the affected apps. Optionally moves Desktop/Documents/Downloads into cloud storage and leaves symlinks behind so they sync — pass `--cloudstorage 'OneDrive'` (or `'Proton Drive'`, `'iCloud Drive'`, `'None'`), or pick from a prompt where Enter skips it. The provider's app must already be installed and signed in; if its sync folder isn't found, redirection is skipped rather than moving files into an unsynced folder.
- **[05.Secure-Mac.py](Mac/05.Secure-Mac.py)** — optional security hardening, mirrors the Fedora/Windows scripts' category/mode design (harden / `--report` / `--restore-defaults`):
  ```bash
  ./05.Secure-Mac.py --all
  ```
  Categories: firewall, FileVault, Gatekeeper (+ download quarantine), SIP (report-only), SSH, sharing, sudo, lockscreen, auditd (report-only), Guest account + password/lockout policy, privacy (incl. Mac Analytics sharing), software-update policy, and Microsoft Edge policy hardening (same policy set as Windows; skipped automatically if Edge isn't installed), plus an opt-in `--edge-strict`, and an opt-in `--separate-admin` that creates a separate admin account and removes you from the admin group (it checks the new account's password and Secure Token before demoting you). Unlike Windows/Fedora, macOS locks several deep privacy/mDNS controls behind SIP, so those categories are intentionally left out — see the script's own header comment for the full list of what's skipped and why.
- **[04.Update-Mac.py](Mac/04.Update-Mac.py)** — recurring maintenance, not a first-run step: `brew update/upgrade/cleanup`, `mas` app updates, and `softwareupdate --all --install`.
- **[MacSystemInfo.py](Mac/MacSystemInfo.py)** — utility, run any time: prints hardware/software/storage/power/display info via `system_profiler`. 

## Windows

PowerShell scripts. If script execution is blocked on a fresh machine, run once as Administrator: `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser`.

You may also need to unblock files:

```powershell
unblock-file .\filename.ps1
```

- **[01.Setup-Windows.ps1](Windows/01.Setup-Windows.ps1)** — first-run setup, interactive: offers to rename the PC if it still has its default `DESKTOP-XXXXXXX` name (needs an elevated shell), presents a menu per app category (Comms, Media, Office, Utilities, Dev, Creative, Others, AI) and installs your selections via `winget`.
  ```powershell
  .\01.Setup-Windows.ps1 -Run         # pick apps per category
  .\01.Setup-Windows.ps1 -InstallAll  # skip the menus, install every app
  .\01.Setup-Windows.ps1 -MachineName WORKSTATION-01 -InstallAll  # no prompts at all (elevated shell)
  ```
  > **Environment-specific tail:** after installing apps it also runs a Terminal setup script and installs DoD/organization certificates and maps a network drive, from paths (`..\..\Apps\Terminal\Win\TermSetup.ps1`, `..\..\Tech\PKI\NativemodeIssuingCA.cer`) that live outside this repo, in the author's own home-network folder layout — these won't exist on a machine that only has this repo checked out. Pass `-SkipPostInstallSteps` to skip that tail and stop after the app installs.
- **[02.Configure-Windows.ps1](Windows/02.Configure-Windows.ps1)** — first-run preference tweaks, run after setup as your normal user (no elevation needed): 24-hour time and always showing file extensions in File Explorer. Sign out and back in to apply everywhere.
- **[05.Secure-Windows.ps1](Windows/05.Secure-Windows.ps1)** — optional security hardening, run any time after setup, from an elevated (Administrator) PowerShell:
  ```powershell
  .\05.Secure-Windows.ps1 -All
  ```
  Categories: Defender, firewall, BitLocker, SMB signing/SMBv1, network discovery (LLMNR/NetBIOS), UAC, RDP, SSH (OpenSSH Server, if installed), lockscreen, auditing, credential/password policy, attack-surface extras (Defender Network Protection, PowerShell logging, mDNS), privacy (telemetry, Recall, Copilot, activity history), and Edge policy hardening — plus opt-in `-PrivacyStrict`/`-EdgeStrict` categories that can break camera/mic or plain-HTTP/self-signed sites, and an opt-in `-SeparateAdmin` that creates a separate admin account and removes the signed-in user from Administrators (it verifies the new account's password before demoting you). Same three modes as the other platforms: harden (default), `-Report`, `-RestoreDefaults`. Run `Get-Help .\05.Secure-Windows.ps1 -Full` for the complete category reference.
  Three update categories (`-WindowsUpdate`, `-StoreApps`, `-WingetUpdate`) are bundled in but are one-way installs, not settings — [04.Update-Windows.ps1](Windows/04.Update-Windows.ps1) runs all three for you.
- **[04.Update-Windows.ps1](Windows/04.Update-Windows.ps1)** — recurring maintenance, not a first-run step (elevated): installs pending Windows Update, Microsoft Store, and winget updates. `-Report` lists what's pending instead.
- **[New-EdgeProfile.ps1](Windows/New-EdgeProfile.ps1)** — utility, run any time: creates a set of named Edge browser profiles (Personal, School, ADM-CCE, ADM-TPM, ADM-Nativemode).
- **[Terminal/](Windows/Terminal/)** — Windows Terminal settings/state and a PowerShell profile consumed by setup, not run standalone:
  - `TermSetup.ps1` replaces Windows Terminal's `LocalState` folder with a symlink into `%OneDriveConsumer%`, so settings sync via OneDrive. It's invoked automatically at the end of `01.Setup-Windows.ps1`, but deletes the existing `LocalState` folder first — only run it once OneDrive is installed and syncing.
  - `Microsoft.PowerShell_profile.ps1` and `MyFunctions.ps1` reference machine-specific tooling paths (`C:\Tools\JITShell`, `C:\CeiseData`) that only exist on the author's own work machines — review and strip those lines before reusing this profile elsewhere.

## Git/

Cross-platform (Windows/Mac/Linux) repo bootstrapping, run in this order on any OS:

1. **[setup.py](Git/setup.py)** — installs git and Git Credential Manager, configures your global `user.name`/`user.email`, and creates the workspace directories the `get_repo_*` scripts below clone into (`Developer/personal`, `Developer/cce`, `Developer/azcjtf`, `Obsidian`).
2. **`get_repo_*.py`** — each syncs a fixed list of repos into one workspace category: clones any that aren't present yet and runs `git pull --ff-only` on the ones that are, so they're safe to re-run. Run whichever apply to you:
   - [get_repo_personal.py](Git/get_repo_personal.py) → `Developer/personal/*`
   - [get_repo_cce.py](Git/get_repo_cce.py) → `Developer/cce/*`
   - [get_repo_cjtf.py](Git/get_repo_cjtf.py) → `Developer/azcjtf/*`
   - [get_repo_natmocode.py](Git/get_repo_natmocode.py) → `Developer/natmocode/*`
   - [get_repo_natmohomecode.py](Git/get_repo_natmohomecode.py) → `Developer/natmohomecode/*`
   - [get_repo_obsidian.py](Git/get_repo_obsidian.py) → `Obsidian/*`

## Functions/

Standalone utilities, not tied to first-run setup — use whenever:

- **[generate_password.py](Functions/generate_password.py)** — interactive or `--non-interactive` CLI password generator (random-character or memorable/word-based), with a strength/crack-time estimate.
- **[generate_username.py](Functions/generate_username.py)** — interactive or `--non-interactive` CLI username generator, including a "Military" operation-name mode.
- **[convert_to_jpeg.py](Functions/convert_to_jpeg.py)** — converts HEIC/HEIF (and other Pillow-supported) images to JPEG. Requires `pip install pillow pillow-heif`.
- **[group_downloads.py](Functions/group_downloads.py)** — sorts a Downloads folder into category subfolders (Images, Installers, Isos, Music, Videos, Docs, other) by extension.
- **[group_photos.py](Functions/group_photos.py)** / **[group_screenshots.py](Functions/group_screenshots.py)** — sort photos or screenshots into `Year/Month` folders by last-modified date.

## Releases

Each platform (Mac, Windows, Fedora) is versioned and released independently. Pushing a tag with the matching prefix triggers [.github/workflows/release.yml](.github/workflows/release.yml), which zips that platform's folder and publishes it as a GitHub Release.

| Platform | Tag prefix    | Example         |
|----------|---------------|-----------------|
| Mac      | `mac-v`       | `mac-v1.0.0`    |
| Windows  | `windows-v`   | `windows-v1.0.0`|
| Fedora   | `fedora-v`    | `fedora-v1.0.0` |

To cut a release:

```bash
git tag mac-v1.0.0
git push origin mac-v1.0.0
```

Repeat with the relevant prefix for Windows or Fedora. The workflow packages only the tagged platform's folder (excluding `__pycache__`/`.pyc`) and attaches it to a new GitHub Release named after the tag — no local `gh` CLI setup required.

### Changelog

Versioning stays manual (you choose the number when you tag), but the changelog is generated automatically:

- On each tag push, the workflow builds release notes from the commit messages touching that platform's folder since its previous tag (or the full history, for a first release).
- Those notes become the GitHub Release body and are also prepended to `<Platform>/CHANGELOG.md` (e.g. [Mac/CHANGELOG.md](Mac/CHANGELOG.md)), which the workflow commits back to `main`.

Because the changelog is derived from commit messages, write commits touching a platform folder as a one-line summary of the user-facing change — that text ends up in the changelog verbatim.
