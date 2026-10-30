# Snowy Studios UI · v2

A UI library for Roblox executor scripts. Built for real hubs: many tabs, hundreds
of controls, desktop and phones, without costing frames.

`SnowyUi` in this repository is the published build. Load it straight from GitHub:

```lua
local Snowy = loadstring(game:HttpGet("https://raw.githubusercontent.com/0x1s2s/Snowy-Ui-Library/refs/heads/main/SnowyUi"))()

print(Snowy.Version) --> 2.0.0
```

**Contents** — [Loading and versions](#loading-and-versions) ·
[Quick start](#quick-start) · [Window](#window) ·
[Tabs and sections](#tabs-and-sections) · [Icons](#icons) ·
[Controls](#controls) · [Values and flags](#values-and-flags) ·
[Notifications](#notifications) · [Theming](#theming) · [Recipes](#recipes) ·
[How it behaves](#how-it-behaves) · [Migrating from v1](#migrating-from-v1) ·
[Gotchas](#gotchas)

---

## Loading and versions

| URL | What it serves |
| --- | --- |
| `.../Snowy-Ui-Library/refs/heads/main/SnowyUi` | Whatever is on `main` now. Every push reaches every script using it (GitHub raw caches ~5 minutes). |
| `.../Snowy-Ui-Library/<commit-sha>/SnowyUi` | That exact commit, forever. Pushing to `main` never changes it. |

Use `main` when you want fixes to land everywhere at once; pin a commit when a
script must never change under you. A pinned URL cannot be "updated" from GitHub —
the script itself has to point somewhere else.

The jsDelivr mirror works the same way:
`https://cdn.jsdelivr.net/gh/0x1s2s/Snowy-Ui-Library@main/SnowyUi` (branch refs
there are cached for hours, so keep it as a fallback, not the primary).

---

## Quick start

```lua
local Snowy = loadstring(game:HttpGet("https://raw.githubusercontent.com/0x1s2s/Snowy-Ui-Library/refs/heads/main/SnowyUi"))()

local win = Snowy:Window({
    Subtitle = "Murder Mystery 2",
})

local tab = win:Tab({
    Name = "Farming",
    Icon = "zap",
    Description = "Automatic collection and rejoin",
})

local box = tab:Section({ Title = "Collection", Column = 1 })

box:Toggle({
    Text = "Auto collect",
    Info = "Picks up every dropped coin in range",
    Default = true,
    Flag = "auto_collect",
    Callback = function(on)
        print("auto collect:", on)
    end,
})

box:Slider({
    Text = "Range",
    Min = 10, Max = 500, Default = 120,
    Suffix = " studs",
    Flag = "collect_range",
})

win:Notify({ Type = "success", Title = "Loaded", Text = "Press RightShift to hide." })
```

---

## Window

```lua
local win = Snowy:Window({
    Subtitle     = "Murder Mystery 2",       -- shown beside the logo (up to two lines)
    Logo         = "rbxassetid://123802801726537", -- optional, see Branding
    LogoRectOffset = Vector2.new(40, 256),   -- optional crop, skips measuring
    LogoRectSize   = Vector2.new(945, 457),
    Brand        = true,                     -- false: no logo, show Title instead
    Title        = "Snowy Studios",          -- only used when Brand = false
    IconPack     = "lucide",                 -- "lucide" (default) | "tabler"
    Keybind      = Enum.KeyCode.RightShift,
    Width        = 780,                      -- 560 – 880
    Height       = 540,                      -- 320 – 640 (starting height)
    MinHeight    = 340,
    MaxHeight    = 600,                      -- capped at 720
    MobileButton = true,                     -- the ping / FPS pill
    NotifySound  = "rbxasset://sounds/electronicpingshort.wav",
    NotifyVolume = 0.45,
})
```

| Method | Purpose |
| --- | --- |
| `win:Tab(opts)` | Add a tab, returns the tab |
| `win:Select(index)` | Switch to a tab |
| `win:Notify(opts)` | Toast, returns a handle |
| `win:Toggle(force)` | Show/hide; omit `force` to flip |
| `win:Search(query)` | Filter every control by text |
| `win:SetKeybind(key)` | Change the show/hide key at runtime |
| `win:LayoutRail()` | Re-flow the tab list after hiding/showing tab buttons |
| `win:SetSingleColumn(bool)` | Force one column (the header button does this too) |
| `win:Destroy()` | Unload and disconnect everything |

Fields scripts can rely on: `win.gui` (the ScreenGui), `win.root`, `win.open`,
`win.tabs` (each with `.index` and `.button`).

### Layout

- **Sidebar** on the left with icon + name per tab, the logo at the top and a
  profile card (avatar and display name, `@username` on hover) at the bottom.
  When the window is narrower than 620 design px it becomes an icon-only rail.
- **Header** shows the tab's name and description, the search field, and the
  column / minimise / close buttons. On narrow windows search collapses to an icon.
- **Two columns** when the content area is at least 540 px wide, one otherwise.
- **Dragging**: grab the header or the logo block. The window follows the pointer
  exactly (no jump on grab) and always stays on screen. Minimise collapses it to
  the header and keeps the top edge where it was.
- **Height** fits the active tab's content between `MinHeight` and `MaxHeight`.
  Once you have moved the window, switching tabs grows/shrinks it from the top
  edge instead of the centre.

### Branding

Every window carries the Snowy Studios wordmark by default. Pass your own `Logo`
to replace it (with `LogoRectOffset` / `LogoRectSize` if you know the crop;
otherwise the artwork is measured once and trimmed automatically), or
`Brand = false` to show `Title` as text. Long titles widen the sidebar slightly
(up to 196 px) and step the font down before they would truncate.

### The pill

A small pill shows the logo, live **ping** and live **FPS** (colour-coded: ping
green ≤90 ms, amber ≤160 ms; FPS green ≥50, amber ≥30). Tap it to show or hide
the window; drag it to move it out of the way.

It sits top-centre on desktop and bottom-centre on phones, where it hides while
the window is open (the close button is reachable). `MobileButton = false` hides
it, except on devices with no keyboard — closing the menu must never strand a
phone user.

---

## Tabs and sections

```lua
local tab = win:Tab({
    Name = "Aim Assist",              -- `Title` is accepted too
    Icon = "crosshair",
    Description = "Targeting, smoothing and hit filters",
})

local left  = tab:Section({ Title = "Aim",     Column = 1 })
local right = tab:Section({ Title = "Filters", Column = 2 })
```

`Description` shows under the tab name in the header — worth filling in.
Section titles render as small uppercase labels above their card.

`Column = 1` is left, `Column = 2` (or `Side = "Right"`) is right. In one-column
layouts every section moves into a single stack in creation order; pass `Order`
to override the two-column order.

Hide a tab without destroying it:

```lua
tab.button.Visible = false
win:LayoutRail()
```

---

## Icons

Icons come from [Lucide](https://lucide.dev/icons) via Iconify, cached to disk and
loaded in the background (window creation never waits on HTTP; a glyph shows
until the image lands).

- Any Lucide name works: `Icon = "swords"`, `"map-pin"`, `"paw-print"`, …
- Common shorthand from older scripts is mapped: `home` → house, `teleport` →
  map-pin, `esp` → scan-eye, `autofarm` → bot, `walkspeed` → footprints,
  `sparkle` → sparkles, `coin` → coins, `gear` → settings, `fly` → plane, …
- Phosphor names (`gear-six-bold`, `magnifying-glass`, …) have `-bold`/`-fill`
  stripped and are mapped; anything Lucide lacks falls back to Phosphor, so an
  icon is never blank.
- A numeric id or `rbxassetid://…` is used as the image directly.

`IconPack = "phosphor"` is accepted for compatibility and now means Lucide with
Phosphor fallback. `IconPack = "tabler"` uses Tabler.

---

## Controls

Every control accepts `Text`, plus optional `Info` (ⓘ tooltip — hover on
desktop, tap on phones), `Flag` (writes into `Snowy.Flags`) and `Callback`.

Rows lay themselves out for the space they get: labels wrap instead of running
under the control, and on narrow columns sliders, dropdowns and inputs move onto
their own line under the label.

### Toggle

```lua
box:Toggle({
    Text = "Enable aimbot",
    Default = true,
    Disabled = false,      -- greys it out and blocks input
    Flag = "aim_enabled",
    Callback = function(on) end,
})
```

The whole row is clickable, not just the switch.

### Slider

```lua
box:Slider({
    Text = "Field of view",
    Min = 0, Max = 180, Default = 165,
    Decimals = 0,          -- 1 or 2 for fractional values
    Suffix = "°",
    Flag = "aim_fov",
    Callback = function(value) end,
})
```

A range bar: the bar is the control, the value sits inside it, and the fill's
bright edge marks the position. Click or drag anywhere on the bar.

### Dropdown

```lua
box:Dropdown({
    Text = "Sort mode",
    Options = { "Closest", "Lowest health", "Highest threat" },
    Default = "Closest",
    Flag = "sort_mode",
    Callback = function(choice) end,
})
```

`Multi = true` makes it multi-select: `Default` and the callback use lists, and
the menu stays open between picks. The button summarises as `Head, Neck +2`, or
`All`. `Placeholder` is shown when nothing is selected.

Lists with more than 10 options get a filter box, and long lists open scrolled
to the current pick. Menus always open fully on screen.

### Keybind

**Click** to rebind (Backspace clears), **right-click** for the mode picker.

```lua
box:Keybind({
    Text = "Aim key",
    Default = Enum.KeyCode.C,
    Mode = "Hold",         -- "Toggle" | "Hold" | "Always"
    Flag = "aim_key",
    Callback = function(state, mode, key) end,   -- the key was used
    OnChanged = function(key, mode) end,         -- the binding changed
})
```

| Mode | Behaviour |
| --- | --- |
| `Toggle` | Pressing the key flips state on/off |
| `Hold` | State is true only while the key is held |
| `Always` | State stays true, key ignored |

Rebinding the menu key:

```lua
box:Keybind({
    Text = "Menu keybind",
    Default = Enum.KeyCode.RightShift,
    OnChanged = function(key) win:SetKeybind(key) end,
})
```

### Button, Input, Label, Divider

```lua
box:Button({
    Text = "Rejoin server",
    ButtonText = "Run",            -- the button sizes itself to this text
    Callback = function() end,
})

box:Input({
    Text = "Config name",
    Placeholder = "default",
    Default = "default",
    Flag = "config_name",
    Callback = function(str) end,   -- fires on Enter only
})

box:Label("Local only. Never replicates to the server.")   -- or { Text = "..." }
box:Divider()
```

### Curve

Draggable cubic bezier for easing and smoothing settings.

```lua
box:Curve({
    Text = "Smoothing curve",
    Default = { {0.05, 0.86}, {0.42, 0.86}, {0.58, 0.14}, {0.95, 0.14} },
    Callback = function(points)
        -- points.P0, points.C1, points.C2, points.P3 are Vector2
    end,
})
```

---

## Values and flags

Every control returns a handle:

```lua
local toggle = box:Toggle({ Text = "Enabled", Default = true })

toggle:Get()        -- current value
toggle:Set(false)   -- update it
toggle.Instance     -- the row (for a Label: the TextLabel itself)
```

| Control | `Get()` returns | `Set()` fires `Callback`? |
| --- | --- | --- |
| Toggle | boolean | yes |
| Slider | number | yes |
| Dropdown | choice, or a list when `Multi` | yes |
| Keybind | `key, mode, state` | no |
| Input | string | no |
| Curve | the raw node table | — |
| Button / Label / Divider | — | — |

Or read everything flat through flags:

```lua
box:Slider({ Text = "Speed", Flag = "walk_speed", Min = 16, Max = 100, Default = 16 })

print(Snowy.Flags.walk_speed)      --> 16
```

| Control | Flag value |
| --- | --- |
| Toggle | `true` / `false` |
| Slider | number |
| Dropdown | the choice, or an array when `Multi` |
| Keybind | `{ Key = ..., Mode = "Hold", State = false }` |
| Input | string |

---

## Notifications

```lua
win:Notify({
    Type = "success",              -- info | success | warning | error
    Title = "Configuration saved",
    Text = "Stored default with all current values.",
    Duration = 4,
})
```

| Type | Icon | Colour | Sound pitch |
| --- | --- | --- | --- |
| `info` | info | accent | 1.00 |
| `success` | circle check | green | 1.16 |
| `warning` | triangle | amber | 0.92 |
| `error` | cross | red | 0.80 |

Optional: `Icon` (any icon name), `Color` (overrides the type colour),
`Sound = false` (mute this one).

Toasts stack bottom-right on desktop and top-centre on phones (clear of the
jump button). Hovering pauses the countdown; the × or the returned handle
dismisses early:

```lua
local toast = win:Notify({ Title = "Working...", Duration = 30 })
toast:Close()
```

The sound ships with the Roblox client, so it cannot be moderated or fail to
download. Override per window with `NotifySound` / `NotifyVolume`.

---

## Theming

```lua
Snowy.Theme.Accent = Color3.fromRGB(255, 90, 140)   -- before creating the window

Snowy.Theme   -- Accent AccentDim AccentGlow Good Warn Bad
              -- Text Sub Faint Line Shell Rail Surface Elevated Hover
Snowy.Fonts   -- Regular Medium SemiBold Bold (BuilderSans FontFaces)
```

Colours are read at build time — change `Snowy.Theme` **before** `Snowy:Window()`.

---

## Recipes

**Save and load a config.** Flags are a plain table:

```lua
local HttpService = game:GetService("HttpService")

local function save(name)
    writefile("snowy-" .. name .. ".json", HttpService:JSONEncode(Snowy.Flags))
end

local function load(name)
    local path = "snowy-" .. name .. ".json"
    if not isfile(path) then return end
    for key, value in pairs(HttpService:JSONDecode(readfile(path))) do
        Snowy.Flags[key] = value
    end
end
```

Flags hold raw values, not controls — call `:Set()` on the handles you kept if
the UI should visually reflect a loaded config.

**Drive a loop from a toggle.** Read the flag inside the loop:

```lua
task.spawn(function()
    while task.wait(0.1) do
        if Snowy.Flags.auto_collect then
            -- collect
        end
    end
end)
```

**Hold-to-aim.** `Hold` mode gives you press and release:

```lua
box:Keybind({
    Text = "Aim key",
    Default = Enum.KeyCode.C,
    Mode = "Hold",
    Callback = function(active)
        aiming = active
    end,
})
```

**Clean unload.** Wire a button to `win:Destroy()`; it disconnects every input
connection and removes the ScreenGui.

---

## How it behaves

**Phones and small screens.** The window is laid out for the screen it is on
rather than shrinking a desktop window: the canvas width follows the viewport,
the sidebar becomes a rail, columns merge, and rows stack. Scale only drops a
little on very small screens (never below 0.8 on phones, 0.75 on desktop), so
text stays readable. Rotating the device or resizing the window re-flows live.

**Roblox menu.** While the Esc menu is open the window and pill step aside, and
come back when it closes.

**Avatar.** The profile card loads your headshot with a placeholder until it
arrives, retrying through the thumbnail API if needed.

**Reload safety.** Re-running a script disposes the previous instance through
`getgenv().SnowyStudiosUnload`, so connections never stack across executions.

**Performance.** Sections are plain frames rather than CanvasGroups (each
CanvasGroup is its own render target); only pages and the window root use them.
Search and height-fitting are debounced, and every draggable shares one input
connection.

---

## Migrating from v1

v2 keeps v1's API — existing scripts run unchanged. What looks or acts different:

- Windows are more compact: `Width` is capped at 880 and `MaxHeight` at 720.
- The Snowy wordmark shows by default; `Title` only appears with `Brand = false`.
- Icons are Lucide (old names are mapped, Phosphor is the fallback).
- Sliders are range bars with the value inside.
- `win:Tab({ Title = ... })` now names the tab (v1 ignored it and showed "Tab 1").
- Keybind Backspace now clears the binding (v1 bound Backspace instead).

---

## Gotchas

**Flags are global to the library, not per window.** Two windows in the same
session share `Snowy.Flags`. Prefix flag names if that matters.

**`Input:Set()` and `Keybind:Set()` do not fire callbacks**, by design — they are
for restoring state. Toggle, Slider and Dropdown do fire.

**Theme changes after `Snowy:Window()` will not apply** to controls already built.

**`Callback` on a Keybind is not a rebind hook.** It fires when the key is used.
Use `OnChanged` to react to the binding itself changing.

**Pinned URLs never update.** A script loading `.../<commit-sha>/SnowyUi` keeps
that version forever; point it at `refs/heads/main` to follow new releases.
