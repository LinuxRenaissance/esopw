# ESOPW — ESO Process Watcher

Automatically closes the Bethesda launcher once **The Elder Scrolls Online**
has actually started. No more leaving the launcher sitting around eating
resources while you play.

The watcher runs **only** for the duration of an ESO launch (it is wired into
Steam's launch options), so nothing runs in the background on days you don't
play.

## How it works

1. You launch ESO from Steam as usual.
2. Steam runs the game through this script (`esopw %command%`).
3. The script starts the normal launcher chain and, in the background, waits
   for the game process `eso64.exe` to appear.
4. A few seconds after the game starts, it kills **only** the launcher's
   Chromium (`Cr*`) processes.
5. The game keeps running — the game and launcher share one wine prefix, so the
   script is careful to leave `wineserver` and `eso64.exe` untouched.

Steam still tracks the game normally (overlay, playtime), because the script
`exec`s the real `%command%` at the end.

## Setup (once)

1. Put esopw bash scrip into ~/.local/bin/ folder (create it if it's not there)
2. In Steam, right-click **The Elder Scrolls Online** → **Properties**.
3. Go to the **General** tab → **Launch Options** field.
4. Type the following command:

   ```
   esopw %command%
   ```

That's it. Your normal flow is unchanged:

- **Play** in Steam → launcher opens (and updates the game if needed).
- **Play** in the launcher → game loads, and the launcher closes itself a few
  seconds later.

Note: In some distributions ~/.local/bin/ is not in your $PATH so in those
cases you should put the full path to the script in Steam's launch options.

## Verifying / debugging

The script logs every session to:

```
/tmp/esopw.log
```

A successful run ends with a line like:

```
OK: launcher closed, eso64.exe still running.
```

## Notes

- **Script location**: if you move the `ESOPW` folder, update the path in the
  Steam launch options.
- **The watcher self-terminates** after it closes the launcher, if you close
  the launcher without playing, or if the game never starts (2h safety limit).
- **Game updates**: if the launcher is downloading an update, the watcher just
  keeps waiting — it only fires once `eso64.exe` actually starts.

## Tunable settings

Edit the variables near the top of `esopw` if needed:

| Variable         | Default | Meaning                                         |
| ---------------- | ------- | ----------------------------------------------- |
| `SETTLE`         | `5`     | Seconds to wait after the game starts before closing the launcher. |
| `WAIT_LAUNCHER`  | `120`   | Max seconds to wait for the launcher to appear. |
| `SAFETY`         | `7200`  | Hard cap (2h) on waiting for the game to start. |

## Technical reference

Key facts discovered on this system (ESO via Proton):

| Role            | Identifier                                                  |
| --------------- | ---------------------------------------------------------- |
| Steam App ID    | `306130`                                                   |
| Game process    | `eso64.exe` (`comm`)                                        |
| Launcher        | CEF/Chromium app: `CrBrowserMain` + `CrGpuMain` / `CrRendererMain` / `CrUtilityMain` children |
| Launcher binary | `Bethesda.net_Launcher.exe`                                 |
| Shared prefix   | game and launcher live under the same `wineserver` / proton group, so only the launcher's `Cr*` processes are killed |
