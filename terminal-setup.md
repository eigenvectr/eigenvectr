# Windows Terminal Setup (Git Bash + PowerShell + oh-my-posh)

Replication guide for my terminal config. Current state: Hack Nerd Font Mono + custom `Tiwahu` Windows Terminal color scheme + the bundled `tiwahu` oh-my-posh theme with a hardcoded `edward` identity label replacing the `username@hostname` session segment.

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

> **Gotcha on managed/corp installs:** winget on Lockheed machines places `oh-my-posh.exe` at `%LOCALAPPDATA%\Microsoft\WindowsApps\oh-my-posh.exe` (a WindowsApps alias shim), which does **not** ship with a sibling `themes\` folder. That makes `$env:POSH_THEMES_PATH` empty. Workaround: fetch theme JSONs directly from the upstream repo into `~\.poshthemes\` (see step 4).

## 3. Windows Terminal `settings.json`

Path: `%LOCALAPPDATA%\Packages\Microsoft.WindowsTerminal_8wekyb3d8bbwe\LocalState\settings.json`

Set defaults:

```json
"profiles": {
    "defaults": {
        "colorScheme": "Tiwahu",
        "font": {
            "face": "Hack Nerd Font Mono",
            "size": 11
        }
    }
}
```

### Color schemes to add to the `schemes` array

Keeping multiple so I can swap via `defaults.colorScheme` without re-editing JSON.

**Tiwahu (current default)** — custom scheme built from the `tiwahu` oh-my-posh theme's own palette (`#222222` base, cream foreground, plus tiwahu's accent hexes mapped into the ANSI slots). There's no upstream WT scheme for tiwahu — build it yourself:

```json
{
    "name": "Tiwahu",
    "background": "#222222",
    "foreground": "#cccccc",
    "cursorColor": "#f1f0e9",
    "selectionBackground": "#444444",
    "black": "#222222",
    "red": "#cf432B",
    "green": "#98c379",
    "yellow": "#ffcc88",
    "blue": "#007ACC",
    "purple": "#7014eb",
    "cyan": "#7FD5EA",
    "white": "#cccccc",
    "brightBlack": "#666666",
    "brightRed": "#ff8888",
    "brightGreen": "#b5cea8",
    "brightYellow": "#ffe0b2",
    "brightBlue": "#61afef",
    "brightPurple": "#906cff",
    "brightCyan": "#a8e1ec",
    "brightWhite": "#f1f0e9"
}
```

**Gruvbox Material** (alternates — from [sainnhe's gist](https://gist.github.com/sainnhe/587a1bba123cb25a3ed83ced613c20c0)): three variants share the same palette but differ by background darkness:
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

## 4. oh-my-posh theme — tiwahu with `edward` identity

Since `$POSH_THEMES_PATH` is empty on this install, fetch the canonical tiwahu JSON straight from upstream:

```bash
curl -sL https://raw.githubusercontent.com/JanDeDobbeleer/oh-my-posh/main/themes/tiwahu.omp.json \
  -o ~/.poshthemes/my_tiwahu.omp.json
```

Then edit `~/.poshthemes/my_tiwahu.omp.json` with two changes to the first `blocks[0].segments` array:

**A) Replace the `session` segment** (which shows `username@hostname`, rendering the corporate asset tag `e429606@US02TK41402CW`) with a `text` segment hardcoded to `edward`. Same dark-gray slot, just a literal identity label:

```json
{
    "background": "#222222",
    "foreground": "#666666",
    "style": "plain",
    "template": " edward ",
    "type": "text"
}
```

Place it right after the `os` segment, before the `path` segment.

**B) Leave the path segment's template alone** (` {{ .Path }} `). Earlier iteration had edward baked into the path template, but that's redundant once the text segment above is in place.

Final segment order (left to right in the prompt): `executiontime → status → root → os → [edward text] → path → git → dotnet → go → python → rust`, with the `❯` shell prompt on a second line.

### Tiwahu uses hardcoded hex colors on purpose

Tiwahu's segments use hex values (`#cf432B` git red, `#007ACC` exec-time blue, `#ffcc88` root yellow, `#7014eb` dotnet purple, etc.), **not** ANSI names. That means swapping the WT color scheme does **not** retint the prompt — tiwahu keeps its designed look regardless of background. This is intentional.

### Alternative pattern — auto-tinting (for non-tiwahu themes)

If you're customizing a different theme (e.g. `catppuccin_macchiato.omp.json`) and want the prompt to auto-match whatever WT scheme is active, replace hex values in the `palette` block with ANSI color **names** instead of hex. Example:

```json
"palette": {
    "os":       "brightWhite",
    "closer":   "brightWhite",
    "pink":     "brightYellow",
    "lavender": "brightGreen",
    "blue":     "blue"
}
```

Valid ANSI names: `black`, `red`, `green`, `yellow`, `blue`, `magenta`, `cyan`, `white`, and the `bright*` variants of each. oh-my-posh resolves these against the active terminal's color scheme, so the prompt auto-retints whenever you swap schemes.

**Do NOT apply this trick to tiwahu** — it breaks the theme's designed palette.

## 5. Wire prompt into both shells

**Git Bash** (`~/.bashrc`):

```bash
# Oh My Posh — tiwahu
OMP_EXE="/c/Users/e429606/AppData/Local/Microsoft/WindowsApps/oh-my-posh.exe"
if [ -x "$OMP_EXE" ]; then
    eval "$("$OMP_EXE" init bash --config "$HOME/.poshthemes/my_tiwahu.omp.json")"
fi
```

**PowerShell** (`$PROFILE` — on OneDrive-redirected systems this resolves to `$env:OneDrive\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1`):

```powershell
# Oh My Posh — tiwahu
$ompExe = "$env:LOCALAPPDATA\Microsoft\WindowsApps\oh-my-posh.exe"
if (Test-Path $ompExe) {
    & $ompExe init pwsh --config "$env:USERPROFILE\.poshthemes\my_tiwahu.omp.json" | Invoke-Expression
}
```

## 6. Verify

1. Close all WT windows, reopen.
2. New tab should use Hack Nerd Font Mono + `Tiwahu` near-black (`#222222`) background.
3. Prompt glyphs (folder icon, git branch) should render as glyphs, not boxes.
4. Prompt left-to-right: OS icon → dark-gray `edward` → mid-gray path → (red git segment when in a repo).
5. `❯` on a second line, in tiwahu's VS Code blue (`#007ACC`).

## Troubleshooting

- **Glyphs show as boxes** → font didn't load; restart WT fully (close all windows).
- **"Invalid colorScheme" warning** → scheme name in `defaults.colorScheme` doesn't match any `name` in `schemes` array. Case- and whitespace-sensitive.
- **After swapping themes, prompt still looks like the old one** → stale tab. WT captures color scheme at tab spawn, and oh-my-posh reads its config at shell start. `. $PROFILE` reloads posh but the WT scheme is locked to that tab's spawn-time value. **Open a brand-new tab (Ctrl+Shift+T) instead.**
- **Gruvbox-material-ish cream/olive/amber prompt colors when you expect tiwahu's distinct palette** → shell is still running the old auto-tinting catppuccin config. Check `~/.bashrc` / `$PROFILE` point at `my_tiwahu.omp.json`, not `my_catppuccin.omp.json`, then open a new tab.
- **PowerShell `$PROFILE` points to wrong path** → on OneDrive-redirected Documents, `$PROFILE` resolves to the OneDrive path, not `C:\Users\<you>\Documents`. Run `echo $PROFILE` to confirm before editing.
- **Per-profile `colorScheme` override wins over `defaults.colorScheme`** → WT Settings UI → Profiles → PowerShell → Appearance → Color scheme should say "Use defaults", not a hardcoded value.
