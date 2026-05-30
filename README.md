# CS2 Server Picker

A Windows desktop application that blocks Valve servers in unwanted regions
(for example Russia and Turkey) using Windows Firewall. The goal is to make CS2
matchmaking prefer servers in Western Europe (Frankfurt), so you get better
latency and play mostly with players from EU West.

## What we want to do

1. Show a list of regions (RU, TR, EU West, etc.).
2. Check the regions we want to block.
3. Press "Apply" and the app adds the rules to Windows Firewall.
4. Launch CS2, and matchmaking avoids the blocked servers.
5. Press "Reset" and all rules added by the app are removed.

## How it works in short

```
You (RO) -> CS2 searches for a server
         -> RU/TR servers are blocked in the firewall
         -> CS2 only finds Frankfurt / EU West servers
         -> players are mostly from EU West
```

Important: we block servers, not players. A player from RU can still connect to
a Frankfurt server.

## Implementation of each file

This is the implementation plan. The actual code is written step by step.

### main.py
The application entry point. Checks whether the program runs as Administrator
(required for the firewall), then starts the main window from `gui/app.py`.

### core/firewall.py
The engine of the app. Talks to Windows Firewall through `subprocess` and
`netsh`. Planned functions:
- add an outbound block rule for an IP range
- delete a rule by name
- list the existing rules created by the app
- full reset (delete all rules with the `CS2SP_` prefix)

All created rules use the `CS2SP_` prefix in their name, so they are easy to
identify and remove without affecting other firewall rules.

### core/regions.py
Loads `data/regions.json` and provides the logic for regions and IP ranges.
Planned functions:
- read and validate the JSON file
- return the list of regions to the GUI
- map a region to its IP ranges, for `firewall.py`

### core/profiles.py
Manages saved profiles (combinations of checked regions).
Planned functions:
- save a profile to disk
- load an existing profile
- list and delete profiles

### gui/app.py
The main window built with customtkinter. Brings together the list of region
cards, the profile panel and the "Apply" and "Reset" buttons.

### gui/region_card.py
A visual card for a single region: checkbox, flag and the region name. Keeps the
state (checked / unchecked) for each region.

### gui/profile_panel.py
The side panel for profiles: save a new profile, pick an existing profile,
delete a profile.

### data/regions.json
The data with the IP ranges of Valve servers, grouped by region. It can be
updated occasionally, because Valve IPs may change over time.

### assets/flags/
Flag icons (png) used by the region cards.

## Known limitations

| Aspect | Reality |
|---|---|
| We block servers, not players | An RU player can still appear on a Frankfurt server |
| Valve IPs can change | `regions.json` needs an occasional update |
| Requires Administrator | To modify the firewall |
| Longer matchmaking time | Fewer servers available |
| VPN does not help ping | It adds overhead, it does not lower ms |
