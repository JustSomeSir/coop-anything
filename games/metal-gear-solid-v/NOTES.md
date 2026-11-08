# Metal Gear Solid V: The Phantom Pain: co-op notes

Research-only pass (no code written, no game installed, no reverse engineering done by us).
All findings are from public pages, the GitHub API, and web search. Dates are when verified.

## Game identity

| | Value |
|---|---|
| Game | Metal Gear Solid V: The Phantom Pain |
| Release | 2015 |
| Engine | Fox Engine (Konami, closed/proprietary) |
| Scripting | Lua 5.1, embedded; most game logic is scripted |
| Store tested | none by us; mods target the Steam release (AppID 287700) |
| Exe hash | unknown (not computed; no install touched) |

Not yet in coop-anything's DIRECTORY.md before this pass. Related game not covered here:
MGSV: Ground Zeroes (separate game, same engine); no co-op project found for it, not searched
in depth.

## Goal

Play The Phantom Pain in co-op with friends. Only a 2-player route exists today (Dynamite).
Nothing supports 3+ players in a playable build yet.

## Research log

### 2026-10-28 Phase 0-1: triage + research (web only)

#### Found (with sources)

| Find | What it really is | License | Last activity | Players | Use for us |
|---|---|---|---|---|---|
| [unknown321/dynamite](https://github.com/unknown321/dynamite) | The real one. 2-player co-op mod, "early development stage". Real mission playthroughs on video (missions 5, 15). PC Gamer: "a functioning Metal Gear Solid 5 co-op mod". Ships a `masterserver/` and a `hook/`. Requires unmodified Steam copy (JP not supported) and a game-version **rollback** before install (issue #136). | GPL-3.0 **+ Commons Clause** (no selling) | pushed 2026-09-29; created 2025-07-20; v0.6.0 video 2025-10-31; 476 stars, 55 open issues | 2 | Path 1 target: revive/finish. Bug reports in its GitHub issues. |
| [Hound-Protocol/Joint-Operations](https://github.com/Hound-Protocol/Joint-Operations) | Aims for up to 16 players (1 host + 15 clients) in the open world, Windows + Linux/Proton. Heavy org/docs/research scaffolding; README states **"Nothing is playable yet."** First build 0.1 (2 players) is *planned*. Plans to use the game's own online functionality + Steamworks SDK. | GPL-3.0-only (docs CC BY-SA 4.0) | pushed 2026-10-25; created 2026-10-21; 0 stars, 0 issues | up to 16 (planned) | Watch. Public research/decision notes may be useful. Clean-room IP policy (won't copy Dynamite code). |
| [alex-ppg/mgs-v-multiplayer-mod](https://github.com/alex-ppg/mgs-v-multiplayer-mod) | Placeholder repo. One-line README; co-op stated as the first target. No code yet. | CC0-1.0 | created + pushed 2026-10-25; 0 stars, 0 issues | unknown | Watch; nothing usable yet. |
| MGSV Modding Wiki: [Infinite Heaven](https://www.nexusmods.com/metalgearsolidvtpp/mods/1226), [V_Framework](https://mgsvmoddingwiki.github.io/V_Framework/), [IHHook](https://mgsvmoddingwiki.github.io/IHHook), [Zeta](https://mgsvmoddingwiki.github.io/Zeta) | The modding stack, not co-op itself. Infinite Heaven = big Lua mission framework. V_Framework = Lua layer exposing UI/sound/soldiers/bosses/vehicles ("breaks the wall between Lua and the exe"), needs IH + game 1.0.15.3/1.0.15.4. IHHook = `dinput8.dll` proxy adding Lua C API. Zeta = in-game mod manager/framework. | varies | current | The reason co-op is feasible here at all: native hook (IHHook) + rich Lua control (IH/V_Framework). |
| [TinManTex/MockFox](https://github.com/TinManTex/MockFox) | Runs the game's Lua + IH scripts outside the game in a Lua interpreter; used to generate IH docs. | see repo | - | Offline Lua analysis without launching the game. |

#### Observed (facts from sources)

- Fox Engine is closed/proprietary (Konami); no decomp or source port exists.
- MGSV is heavily Lua 5.1 scripted; Lua lives in `data1.dat`/patch `.dat` and inside `fpk` files.
  This is what the IH/V_Framework stack hooks.
- FOB (Forward Operating Base) is asynchronous PvP infiltration; Metal Gear Online (MGO) is a
  separate, unrelated multiplayer mode. Neither is co-op campaign.
- Dynamite is the only build anyone reports actually playing co-op in.

#### Derived (not verified by us)

- This is a **Path 1 (revive/finish)** situation, not build-from-zero: a functional 2-player
  mod exists. The honest play-now answer for a player is Dynamite.
- Dynamite's `masterserver/` + "online connection" requirement implies hosted matchmaking and/or
  relay, with the game instances synced host-authoritatively. The exact transport is **not
  confirmed** from public docs read so far.
- A co-op campaign on the Fox Engine almost certainly rides on the Lua layer (IH/V_Framework) +
  a native hook (IHHook-style `dinput8.dll` proxy) for state/entity sync, since there is no
  usable built-in co-op netcode and no source.

#### Unknown (next to check)

- Dynamite's exact networking model (direct-IP? relay? P2P + master server for matchmaking?) and
  its supported game build number / the exact version to roll back to (issue #136).
- What Dynamite actually syncs beyond "two players in the same mission": world state, Mother Base,
  items, save/progression ownership. Needs reading its docs (`webui/docs/docs.md`, `docs/MAKING_OF.md`)
  and/or a test install.
- Whether Dynamite builds on Infinite Heaven or hooks the game independently.
- License practicality: GPL-3.0 + Commons Clause allows forking/derivatives (non-commercial);
  fine for reviving, not for selling.
- Ground Zeroes: any co-op attempt (not searched in depth).

### 2026-10-28 Phase 1b: Dynamite deep-dive (its docs + 55 open issues)

Read `docs/MAKING_OF.md` (a long build diary) and `webui/docs/docs.md` (user docs), and listed the
open issues via the GitHub API. No install, no code. This resolves the networking/version/what-syncs
unknowns above.

#### Observed (from the author's own docs)

- **Origin:** author (unknown321) built a MGSV master-server reimplementation (`fuse`, Go+SQLite) to
  connect two players on a FOB map, then swapped the map/mission to single-player ones -> dynamite.
  First two-player contact 2025-04-07.
- **Why it is possible:** a Fox Engine session is natively multiplayer (up to 16). Single-player
  missions run a 1-player session; FOB missions run the same session but let a second player join.
- **Connection:** Steam P2P session, host accepts incoming connections, client supplies the host's
  **SteamID** (via UI target or hardcoded). A Steam-network connection is required to broker it; after
  connecting, the client knows the host's IP (so there is a griefing surface -> black/whitelists in
  `dynamite.log.txt`). Client must press Master+UP (`Dynamite.CreateClientSession`) after the host
  loads, every time.
- **Two player instances:** single-player missions allocate one player; a `.fox2` definition edit adds
  the second instance so both render.
- **Host authority:** enemies are hardcoded to attack player 0 (host) and ignore the joiner
  ("defender"). Enemy AI is host-controlled; movement commands are sent to the client. Dynamite patches
  targeting to "nearest player entity."
- **Native layer:** started as pure Go exe byte-patching (no DLL) for portability; targeting logic
  forced a move to C++ via an **IHHook**-based `dinput8.dll` proxy (IHHook is part of Infinite Heaven,
  but dynamite is NOT IH-based and rejected building on IH for readability/compat reasons).
- **Markers:** first hacked over an `AddLocalDamage` packet (6 floats), later replaced by the game's
  in-game network messaging. Only ~11 of the game's entity sync instances are implemented.
- **Save safety:** save location is defined in Lua, so dynamite backs up + copies the save and runs on
  the copy (tool `squib`); single-player data is untouched, online funds/resources stubbed.
- **Packaging:** single cross-platform Go binary; own repack tool `datfpk` (rejected SnakeBite/Makebite
  and GzsTool as unworkable on Linux / for rapid iteration); config is an `.ini` edited through a local
  web UI; local `fuse` on port 6667 avoids sending co-op records to Konami (ban risk).
- **Platforms:** Windows 10+ and Linux/SteamOS (Proton needs `WINEDLLOVERRIDES="dinput8.dll=n,b"`).
- **Version:** latest release v0.6.0 (2025-10-31); master pushed to 2026-09-29. Issue #136: the
  2026-06-09 Steam update breaks launch, hence the mandatory rollback (exact pre-patch build id not
  stated).

#### What it really syncs (observed)

| Area | State |
|---|---|
| Players visible | Buggy. Partner avatar often renders only in FOB mode, not regular missions (#109, #59). |
| Enemies | Partial. Vitals synced; positions and FOV NOT synced (#112, #113); AI host-driven. |
| Story/objectives | Partial. Secondary objectives + objective markers not shared (#115, #117); some update host-only (#118). |
| World state | Partial. Weather host-only; gimmicks/physics desynced; outpost capture not announced to client (#98). |
| Items/loadout | Per-player and partial. Loadout drop invisible to partner; ammo drop can be the wrong player's; weapon pickup "not allowed" (#87, #45). |
| Vehicles | Broken. One instance only, position desync, driver frozen; tank-as-host can crash (#128, #35, #42, #47). |
| Buddies | Quiet/D-Dog local (unseen by partner); D-Horse teleport desync (#43, #90, #122). |
| Markers (user + enemy) | Working (custom sync). |

#### Bug clusters (from the 55 open issues)

- **Connection/installer:** #136 (2026-06-09 patch breaks launch), #129/#127 connect fails, #92/#76/#102 installer, #85 (new save can't connect until missions 1&2 unlocked).
- **Enemy sync:** #112 positions, #113 FOV, #36 revenge, #55 skulls.
- **Player sync:** #109/#59 invisible partner, #29/#31 animations, #87 weapon-not-allowed, #94 cargo fulton.
- **Objectives:** #115 markers, #117 secondary objectives, #118/#95/#83 per-mission.
- **Vehicles/Walker Gear:** #128/#35/#42/#47/#19, #130/#116/#21.
- **Per-mission crashes:** #3 (#13 Pitch Dark), #16 (#43), #103 (#30), #15 (#28), #124 (#10).
- **Enhancements/roadmap:** #58 free roam, #123 invite-based join, #4 FOB items; `>2 players` (author's stated priority, currently fails to start).

#### Resolved from the earlier "Unknown" list

- Networking model: **confirmed** (Steam P2P by SteamID + local fuse master server; host-authoritative).
- What it syncs: **mapped** (table above).
- Builds on Infinite Heaven? **No**, uses IHHook only.
- Supported version: rollback target is the build **before the 2026-06-09 Steam patch** (exact id still unknown).
- Still open: exact pre-patch build number; whether unreleased master fixes change any of the above.

## Sources

- https://github.com/unknown321/dynamite
- https://github.com/unknown321/dynamite/blob/master/docs/MAKING_OF.md
- https://github.com/unknown321/dynamite/blob/master/webui/docs/docs.md
- https://github.com/unknown321/dynamite/issues
- https://www.pcgamer.com/games/action/after-10-years-theres-a-functioning-metal-gear-solid-5-co-op-mod-so-you-and-a-friend-can-be-played-like-a-damn-fiddle-together/
- https://github.com/Hound-Protocol/Joint-Operations
- https://github.com/alex-ppg/mgs-v-multiplayer-mod
- https://mgsvmoddingwiki.github.io/V_Framework/
- https://mgsvmoddingwiki.github.io/IHHook
- https://www.nexusmods.com/metalgearsolidvtpp/mods/1226
- https://github.com/TinManTex/MockFox

### 2026-11-03 Phase 0/2: triage on a real install + build of bimmylarter/dynamiteMe

#### Observed
- Steam install, english, buildid 24176213 (depot 287701 manifest 1506669297791974057), `version_info.txt`
  `Tpp_steam_mst_en_day3900Mgo_patch_0707_1632`. `mgsvtpp.exe` SHA-256
  `085c2f82d1c963c40b3d2d55786661dfee2b18cbbf388a710c00fa76c5e9bb45`, MD5 `bb905b1fc5bc5af60a9808b3088a1b32`,
  PE TimeDateStamp `1783412888` = game 1.0.15.4 (June 2026 update).
- Upstream Dynamite (master e559c3d) supports 1.0.15.3 only (exe md5 `7cc5f282...`, addresses header 1_0_15_3).
- [bimmylarter/dynamiteMe](https://github.com/bimmylarter/dynamiteMe) (master 9b70c85, 2026-10-15) adds 1.0.15.4:
  version picked by PE timestamp, ported address set (`tools/addressport`), installer accepts 1.0.15.4 `00.dat`
  md5 `b28bedbc...`; plus body/carry sync work on branches. Its timestamp matches the install above.
- [publicimagexyz/dynamite-aiassist](https://github.com/publicimagexyz/dynamite-aiassist): code audit + roadmap
  only (game-thread violations, unchecked network handlers, one global enemy target).
- Hook (`dinput8.dll`) builds with llvm-mingw 20261020 (clang 23, ucrt) after 3 small portability fixes
  (detour cast to `LPVOID` in `CREATE_HOOK`, `std::from_chars` on pointers, `<csignal>`). Exports
  `DirectInput8Create`. Not run in-game yet.
- Go launcher could not be built in a sandbox whose network blocks `golang.org/x/*` and `modernc.org/*`.

#### Playbook contradiction
- Write-up says the rollback is mandatory. With the dynamiteMe fork it is not needed on 1.0.15.4. Update README.

#### Unknown
- Whether dynamiteMe runs on 1.0.15.4 in practice (no test run yet). QA level stays `none`.

### 2026-11-03 Phase 6: first single-PC smoke run (owner launched it, no bot)
- Setup: copy of the 1.0.15.4 install, `dynamite.exe` built from dynamiteMe 9b70c85 + llvm-mingw hook fix
  (JustSomeSir/dynamiteme `build/llvm-mingw`), launched `mgsvtpp.exe` directly from the copy.
- `dynamite.log.txt`: `game version 1.0.15.4`, `CreateHooks() ... done` (all hooks installed, no create-hook
  failure), Lua hook up (`luaL_openlibsHook`), `TppMission.OnMissionStart` for mission code 1 and 40020 (ACC),
  emblem hooks fire. No error lines. Clean exit.
- Not tested: any 2-player session. QA level stays `none` (single-player load is not a co-op test).

### 2026-11-08 Phase 6 prep: two copies on one PC (no second player available)

#### Observed (static, `mgsvtpp.exe` 1.0.15.4)
- Imports `CreateMutexW`/`ReleaseMutex` and `CreateWindowExW`/`RegisterClassExW`, no `FindWindow*`.
  Code sections are protected (`.xtext`, `.xdata`, `.ecode`), so the mutex name can't be read
  statically; only one useful wide string (`Tpp_main.exe`). Whether a 2nd copy starts: `unknown`,
  needs a live run.
- `steam_api64.dll` shipped with the game: SHA-256 `08bd0152356e5619ee56a6714ac951b6800dbb21821693a4efaf85278442c60a`,
  FileVersion 02.77.37.82, interfaces incl. `SteamNetworking005`, `SteamUser018`, `SteamClient017`.
- Exe imports from it only flat accessors: `SteamAPI_Init`, `SteamAPI_RunCallbacks`,
  `SteamAPI_(Un)RegisterCallback`, `SteamAPI_(Un)RegisterCallResult`, `SteamUser`, `SteamFriends`,
  `SteamUtils`, `SteamUserStats`, `SteamRemoteStorage`, `SteamApps`, `SteamNetworking`.
- Dynamite rides the game's own `SteamUdpSocketImpl` (FOB netcode) and hooks its
  `OnP2PSessionRequest`, so all co-op packets go through `ISteamNetworking005`.

#### Derived: plan for a test transport
- A proxy `steam_api64.dll` that forwards every export to the real DLL (renamed), except:
  `SteamNetworking()` returns a wrapper whose vtable overrides `SendP2PPacket`,
  `IsP2PPacketAvailable`, `ReadP2PPacket`, `AcceptP2PSessionWithUser`, `CloseP2PSessionWithUser`,
  `GetP2PSessionState` and sends over localhost UDP to the other copy; `SteamAPI_RegisterCallback` is
  watched for callback id 1202 (`P2PSessionRequest_t`) so the proxy can raise it inside
  `SteamAPI_RunCallbacks` for the first packet from the peer; peers get fixed fake SteamIDs from an
  env var / ini so host and client can tell each other apart while both run on one Steam account.
- Installed only in the two test copies, never in the owner's install. Same idea as the playbook's
  mock DLL (networking-steam-spacewar.md), but a proxy, because the game needs real Steam for
  everything else (ownership, saves, master-server login).

#### Unknown
- Mutex: does a 2nd `mgsvtpp.exe` start while one runs? Does Steam allow it for the same app id?
- Does the local master server (fuse) or the FOB login reject two logins from one SteamID?
- Owner PC: RTX 5060 Ti 16 GB, 16 GB RAM. Enough for two copies at low settings: `unknown` until tried.

### 2026-11-08 Phase 5: packet validation in the hook (no second player needed)

#### Observed
- `DynamiteSyncImpl::RecvRaw` already runs `flatbuffers::Verifier` (so the aiassist audit's
  "unchecked network handlers" is only partly true), but the Verifier passes missing union bodies,
  null strings/structs/vectors, short vectors and unknown union types. Handlers dereference those
  (`m->category()->c_str()`, `m->pos()->x()`) and `memcpy` 176-byte BossQuiet structs out of the
  vectors (`HandleSendBossQuiet*`), i.e. a short packet reads out of bounds.
- Fix on owner fork branch `fix/validate-packets`: `hook/src/DynamiteSyncImpl/MessageValidation.h`
  (game-free checks per message type), called in `RecvRaw` before `packet_num` is advanced; also
  rejects packets shorter than the 8-byte header and verifies only the received bytes.
- `hook/tests/run_tests.sh` (host g++, ASan/UBSan): 34 cases pass; the Verifier alone passes 19 of
  19 malformed cases; 5 of 5 validator mutants are killed. Hook builds with llvm-mingw.

#### Unknown
- `SyncVar` writes `array_start + i` into a game variable array whose length the hook doesn't know;
  only uint32 wrap is rejected. Needs the var's array size from `ScriptDeclVarsImpl` (not mapped).
- Not run in game yet; behaviour with real packets needs the two-copy setup.

### 2026-11-08 Phase 6: why a second copy exits, and the fix

#### Observed (tools/instancediag on owner fork, branch `tools/instancediag`; 1.0.15.4)
- Second `mgsvtpp.exe` started from PowerShell while one copy sat at the title screen:
  `exit=0`, lived 1.9 s, no window, no Steam message. Its log:
  `CreateMutexW name="METAL GEAR SOLID V: THE PHANTOM PAIN" lastError=183 (ALREADY_EXISTS) caller=mgsvtpp.exe`
  at 1060 ms, then `ExitProcess code=0` 19 ms later. No `CreateProcess*` (no Steam/DRM relaunch).
- Steam overlay mutexes/events (`SteamXInput_mutex-IPCWrapper`, `GameOverlayRender_PIDStream_*`)
  also report ALREADY_EXISTS; they are shared by design and don't stop the game.
- Saves: a Dynamite-installed copy uses `DYNAMITE_GAME_DATA` / `DYNAMITE_CONFIG_DATA` /
  `DYNAMITE_PERSONAL_DATA` (moddata/standalone/TppDefine.lua); a vanilla copy uses the owner's
  `TPP_*` files. So every test copy must have Dynamite installed, never a vanilla copy past the title.

#### Fix (branch `test/second-instance`)
- `hook/src/SecondInstance.cpp` + `InstanceName.h`: if `DYNAMITE_INSTANCE` or
  `<game dir>/dynamite/instance.txt` gives an id, `CreateMutexW` for exactly that name becomes
  `"... #<id>"`. No id = no hook. Host test `hook/tests/instance_name_test.cpp` passes. Not run in game yet.

#### Unknown
- What fails next for the second copy (Steam init, D3D, save access, fuse login with one SteamID).
- Both copies share one set of DYNAMITE_* saves; concurrent writes are a risk (backups exist).
