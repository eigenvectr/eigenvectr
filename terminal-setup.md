# Windows Terminal Setup (Git Bash + PowerShell + oh-my-posh)

Replication guide for my terminal config. Goal: Nerd Font + oh-my-posh prompt that auto-tints to match whatever Windows Terminal color scheme is active, with gruvbox-material as the current default.

## 1. Install Nerd Font

Install a Nerd Font **Mono** variant (the non-Mono variants render too tall in WT). Easiest path is via oh-my-posh's font installer:

```powershell
oh-my-posh font install hack          # current pick
# alternates worth trying:
oh-my-posh font install jetbrains-mono
oh-my-posh font install fira-code
oh-my-posh font install meslo
```

After install, a full **WT restart** (close all windows) is required — new tabs alone won't pick up newly installed fonts due to font cache.

Fonts tried and rejected on this machine:
- MesloLGM Nerd Font (non-Mono) size 10 — tall/low-res
- MesloLGS Nerd Font Mono size 11 — didn't like
- MesloLGM Nerd Font Mono size 11 — didn't like
- Iosevka NFM size 12 — too narrow/tall by design
- **Hack Nerd Font Mono size 11 — current**

## 2. Install oh-my-posh

```powershell
winget install JanDeDobbeleer.OhMyPosh -s winget
```

Restart shell, then verify: `oh-my-posh --version`.

## 3. Windows Terminal `settings.json`

Path: `%LOCALAPPDATA%\Packages\Microsoft.WindowsTerminal_8wekyb3d8bbwe\LocalState\settings.json`

Set defaults:

```json
"profiles": {
    "defaults": {
        "colorScheme": "GruvboxMaterialMediumDark",
        "font": {
            "face": "Hack Nerd Font Mono",
            "size": 11
        }
    }
}
```

### Color schemes to add to the `schemes` array

Keeping multiple so I can swap via `defaults.colorScheme` without re-editing JSON.

**Gruvbox Material** (current default — from [sainnhe's gist](https://gist.github.com/sainnhe/587a1bba123cb25a3ed83ced613c20c0)): three variants share the same palette but differ by background darkness:
- `GruvboxMaterialHardDark`  — bg `#1D2021` (most contrast)
- `GruvboxMaterialMediumDark` — bg `#282828` (canonical, matches classic gruvbox bg)
- `GruvboxMaterialSoftDark`  — bg `#32302F` (least contrast)

Shared palette for all three dark variants:

```json
{
    "name": "GruvboxMaterialMediumDark",
    "background": "#282828",
    "foreground": "#D4BE98",
    "cursorColor": "#D4BE98",
    "selectionBackground": "#665C54",
    "black": "#665C54",
    "red": "#EA6962",
    "green": "#A9B665",
    "yellow": "#D8A657",
    "blue": "#7DAEA3",
    "purple": "#D3869B",
    "cyan": "#89B482",
    "white": "#D4BE98",
    "brightBlack": "#928374",
    "brightRed": "#EA6962",
    "brightGreen": "#A9B665",
    "brightYellow": "#D8A657",
    "brightBlue": "#7DAEA3",
    "brightPurple": "#D3869B",
    "brightCyan": "#89B482",
    "brightWhite": "#D4BE98"
}
```

For Hard/Soft, change `"name"` and `"background"` only.

Other schemes I keep around: `Dracula`, `Gruvbox Dark` (classic, more saturated), `Tokyo Night`, `Catppuccin Macchiato`. Browse more at [windowsterminalthemes.dev](https://windowsterminalthemes.dev/).

> **Gotcha:** WT hot-reloads on every save. If you add the `colorScheme` reference before the scheme itself exists in the `schemes` array, WT will fire an "invalid colorScheme" warning. Add the scheme first, then flip `defaults.colorScheme` — or just ignore the transient warning once both edits are done.

## 4. oh-my-posh theme (auto-tinting trick)

Copy a built-in theme to a personal file so updates don't clobber it:

```powershell
Copy-Item "$env:POSH_THEMES_PATH\catppuccin_macchiato.omp.json" "$env:USERPROFILE\.poshthemes\my_catppuccin.omp.json"
```

**Key move:** replace hex values in the `palette` block with ANSI color **names**. oh-my-posh resolves ANSI names against the active terminal's color scheme, so the prompt auto-retints whenever you swap WT schemes — no theme editing required.

Current palette mapping (kept original keys for backward compat; values are ANSI names):

```json
"palette": {
    "os":      "brightWhite",
    "closer":  "brightWhite",
    "pink":    "brightYellow",
    "lavender": "brightGreen",
    "blue":    "blue"
}
```

Valid ANSI names: `black`, `red`, `green`, `yellow`, `blue`, `magenta`, `cyan`, `white`, and the `bright*` variants of each.

On GruvboxMaterialMediumDark this renders:
- path icon + `›` closer → cream `#D4BE98`
- path → amber `#D8A657`
- git → olive `#A9B665`

## 5. Wire prompt into both shells

**Git Bash** (`~/.bashrc`):

```bash
eval "$(oh-my-posh init bash --config "$HOME/.poshthemes/my_catppuccin.omp.json")"
```

**PowerShell** (`$PROFILE` — on OneDrive-redirected systems this resolves to `$env:OneDrive\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1`):

```powershell
oh-my-posh init pwsh --config "$env:USERPROFILE\.poshthemes\my_catppuccin.omp.json" | Invoke-Expression
```

## 6. Verify

1. Close all WT windows, reopen.
2. New tab should use Hack Nerd Font Mono + GruvboxMaterialMediumDark background.
3. Prompt glyphs (folder icon, git branch) should render as glyphs, not boxes.
4. `cd` into a git repo — git segment should show in olive green.
5. Swap `defaults.colorScheme` to a different scheme and reload a tab — prompt colors should re-tint automatically.

## Troubleshooting

- **Glyphs show as boxes** → font didn't load; restart WT fully (close all windows).
- **"Invalid colorScheme" warning** → scheme name in `defaults.colorScheme` doesn't match any `name` in `schemes` array. Case- and whitespace-sensitive.
- **Prompt colors don't change when swapping schemes** → palette still contains hex values, not ANSI names. Re-check step 4.
- **PowerShell `$PROFILE` points to wrong path** → on OneDrive-redirected Documents, `$PROFILE` resolves to the OneDrive path, not `C:\Users\<you>\Documents`. Run `echo $PROFILE` to confirm before editing.
