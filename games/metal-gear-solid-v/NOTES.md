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

### 2026-11-08 Phase 6: two copies on one PC run (owner launched them, no bot yet)
- Both test copies have Dynamite installed (separate `DYNAMITE_*` saves) and the hook from
  `test/second-instance` (SHA-256 `38d4f486fc4a1fcf87ed186385d966236333db3fb8efcf94020ed52fecd5f6b9`); copy 2 has
  `dynamite/instance.txt` = `2`; one launcher (master server) from copy 1.
- Copy 2 log: `second instance: id 2, CreateMutexW hook MH_OK`, then
  `game mutex renamed to instance 2, lastError=0`; game version 1.0.15.4, all hooks created.
- Both copies reached mission 1 (title) and then 40020 (ACC) at the same time, same Steam account,
  no errors in either log. PowerShell: copy 2 `alive=True` after 60 s. Copy 1 log shows no effect
  from copy 2 starting.
- So: two MGSV copies run side by side on one PC with one Steam account (RTX 5060 Ti 16 GB, 16 GB RAM).
- Note: both copies loaded the same `DYNAMITE_GAME_DATA` (ACC). No damage seen yet; installer zip
  backups and our own backup exist. Per-copy save names are still a TODO.

#### Next
- Co-op traffic between them: both run as SteamID `76561198131349084`, so Steam P2P can't connect
  them. Build the `steam_api64.dll` proxy from the plan above (localhost UDP, fake peer SteamIDs).

### 2026-11-08 Phase 6: first co-op session on ONE PC, one Steam account (owner played both windows)

#### Observed
- Setup: copies 1 (host) and 2 (client) of 1.0.15.4 with Dynamite; hook from owner fork branch
  `test/steamproxy` (per-instance mutex + loads `steamshim.dll` when `steam_proxy.ini` exists);
  `tools/steamproxy` in hook mode; client has `fake_steam_id=76561202255233008`; one launcher
  (master server) started BEFORE both games.
- A replaced `steam_api64.dll` (proxy with all 747 exports forwarded, same names and ordinals) makes
  the game show "failed to load SteamAPI" without ever calling its `SteamAPI_Init` (logged). The
  same code as in-memory detours (MinHook on the loaded original DLL) works. Playbook gap: the
  mock-DLL route in networking-steam-spacewar.md does not work for this game.
- Self-test on real Steam (`steamproxy_selftest.exe --shim`): `ISteamUser::GetSteamID` returns via a
  hidden pointer after `this`; `GetAuthSessionTicket` is vtable[13] with the SteamID at +12.
- 23:42:01 client `CreateClientSession` -> host shim delivers `P2PSessionRequest_t(76561202255233008)`,
  Dynamite: `Connection accepted` / client: `Co-op connection established`, pings both ways, both in
  mission 10020. ~60 packets/s each way, after 12 min: client sent 35922 / received 35978,
  `dropped=0 sendFailed=0 badHeader=0`.
- Client `CreateClientSession` fails at once (`dynamite_connection_failed`, no P2P call) when the host
  is not yet in the mission (23:41:05, host entered 10020 at 23:41:42). Earlier failures (23:13-23:16)
  were with the launcher started after both games; not isolated which condition mattered.
- The ticket patch did not run in the game (no `GetAuthSessionTicket` call logged, no
  `CMD_AUTH_STEAMTICKET` on the master server), so it was not what fixed the connection.
- Host log: repeated `SightManagerImplSetMarkerHook ... false, objectID 1096` errors during play.

#### Owner observations (manual play, not a test)
- Enemies and players look in sync. Desync: D-Horse and vehicles called in from Mother Base.

#### Unknown
- Why the very first connect attempt (23:38:42) closed after 10 s on both sides.
- QA level stays `none`: no bot, no assertions yet. Next: in-game bot on this two-copy setup.

### 2026-11-09 Phase 6: QA harness (built and tested without the game; first real run pending)

Owner fork `JustSomeSir/dynamiteme`, branch `test/qa-bridge` (on top of `test/steamproxy`), `qa/README.md`.

#### Built
- coop-anything `tools/bridge` server embedded in the Dynamite hook (`hook/src/QABridge*.{h,cpp}`),
  started only when `coopqa.ini` sits next to `mgsvtpp.exe`; endpoint in `dynamite/qa-endpoint.json`.
- Main-thread pump without new game addresses: Dynamite's `script_loader.Update` (in TppMain's
  per-frame list) requires `lua/coopqa.lua` when `coopqa.ini` exists; its `Update` calls
  `Dynamite.QAPump(dispatch)`. Requests run as Lua via `dispatch(kind, arg)`; the C++ side uses only
  Lua C functions that have 1.0.15.4 addresses (`pushvalue`, `pushstring`, `pcall`, `checklstring`,
  `remove`) and leaves the stack balanced. `lua_settop`/`luaL_loadbuffer`/`lua_type` have no
  1.0.15.4 address in the hook, which is why Lua does the loading and JSON encoding.
- Mission start from the helicopter: `TppMission.ReserveMissionClear{missionClearType =
  FROM_HELISPACE, nextMissionId, nextClusterId}`, the call `TppMission.OnEndMissionPreparation`
  makes after the deploy menu (moddata/standalone/TppMission.lua).
- Runner `qa/run_qa.py` (Python 3.8+, stdlib, vendored `bridge_client.py`): master server
  (`dynamite.exe -installNow`, no web UI), host to mission, waits for `host session res: 1` in the
  host log, client `CreateClientSession` with 3 tries until `Co-op connection established`, client to
  the same mission. Tests: `connect`, `positions`, `stability` (soak). Twins: client never connects;
  swapped player indices.

#### Observed (host tests, Linux)
- `qa/tests/test_qabridge.py`: real bridge server + real Lua 5.1 + `coopqa.lua` with stubs, driven by
  `bridge_client.py`: 16/16 checks pass; twins "stack not restored" and "Lua error reported as success"
  both caught.
- `qa/tests/test_runner_sim.py`: the runner against two simulated games: good -> PASS (twins fail as
  they must); desync, no connect, client crash -> FAIL. Found and fixed: processes leaked when the
  bring-up failed; the log tail could read the previous run's lines; a crash after the asserts
  passed (hence the soak test).

#### Unknown
- Whether `script_loader.Update` runs on the title screen (if not, the bridge answers only from the
  helicopter on). Title -> Continue is still a person (the runner prompts); the title script is not in
  the mod's sources.
- QA level stays `none` until a real run prints `RESULT: PASS` with the twins failing.

### 2026-11-09: title screen hang in runner-driven runs

Observed (host log, run 01:02): mission 1 at 01:02:18, emblem created 01:02:41, `coopqa: pumping in mission 40010`
at 01:02:44, first served `state` at 01:02:49, then the title menu never opened (owner: messages could be confirmed,
spinner stayed). Runs at 00:45 and 00:53 hung the same way after the first served request at mission 1.
Manual sessions without `coopqa.ini` opened the menu fine.

Checked and ruled out: a reinstall by the runner-started `dynamite.exe -installNow` (dynamite.ini not rewritten during
the run; Version=0 on both sides, so "already installed").

Derived: the title backdrop already reports mission 40010 and heliSpace=true, so polling state() can't tell the title
from the helicopter, and bridge requests served while the title menu loads are the likely trigger. Unknown: which call.

Change: the runner sends no bridge requests until the person presses CONTINUE and then Enter in the runner console;
master server output goes to `runs/<run>/master-server.log`. Runner simulation still ALL OK. Not yet run on the game.

### 2026-11-09 01:14: freeze after the first served request, in the helicopter

Observed: with the Enter gate, the owner pressed Continue, had control in the ACC (OnMissionStart 40010 at 01:14:02),
pressed Enter; coopqa built the first state reply at 01:14:13 (Lua log), the runner never received it, and the game
froze. The shim's periodic stats line (from RunCallbacks on the main thread) stopped after 01:14:06. So the title was
not the cause: serving any request on the main thread hangs it, somewhere after the Lua dispatcher returns.

Checked: the 1.0.15.4 addresses of lua_pushvalue (0x141a119d0), lua_remove (0x141a11c20) and luaL_checklstring
(0x141a16f90) disassemble to the Lua 5.1 functions (Dynamite itself never calls the first two). The bridge JSON parser
returns (no loop) on malformed input such as a locale comma.

Guesses: a C++ exception thrown inside the game process does not unwind; a locale decimal comma makes the reply invalid
JSON, which throws. Next build logs each step of the first requests ("qa trace:") and an exception self-test.

### 2026-11-09 01:22: cause of the freeze found

Observed: trace stopped at "call: remove". `lua_remove` has a 1.0.15.4 address in the address table and a declared
function pointer, but `SetFuncPtrs()` never assigns it, so the bridge called a null pointer (the game's crash handler
then hangs instead of closing). Fixed in dynamiteMe test/qa-bridge; the bridge now refuses to run if any Lua pointer it
uses is null. The exception self-test passed on both the setup and the main thread, so exceptions are fine.
Lesson for the playbook: an address in the table does not mean the function pointer is bound; grep the setter.

### 2026-11-09 01:26: first served request works; CoopQA was nil in eval

Observed: after the lua_remove fix the runner got `state` from the helicopter ("host in helicopter"), then
`eval CoopQA.startMission(...)` failed with "attempt to index global 'CoopQA' (a nil value)". Derived: MGSV runs
scripts in their own environment, so `_G.CoopQA` set inside coopqa is not the global table that loadstring chunks get.
Fix: chunks get setfenv to a table with CoopQA that falls back to coopqa's environment; the Lua test now loads coopqa
in a sandbox and the old version fails it (6 checks).

### 2026-11-09 01:36: both copies shared one save

Observed (owner): with startMission on the host, the client copy loaded straight into the mission instead of the
helicopter. Both copies use the same Steam account, so the same `DYNAMITE_*` saves
(`%USERPROFILE%`-independent: `<steam>\userdata\<id>\311340\remote` and `...\287700\local\*{0,1}`).
Fix (dynamiteMe 3bd68c0): with an instance id TppDefine.lua names the saves `DYN<id>_*`; `qa/instance_saves.py`
creates them once from the `DYNAMITE_*` ones. Saves are XOR-encoded keyed by md5(file name) (squib save.Decode), so a
renamed save must be re-encoded; the Python codec matched the Go one byte for byte; decoded bytes 0..15 are md5 of the
rest and are checked before writing. The runner now fails early if host and client report the same `saveFile`.
Not yet run on the game. Open question: Steam Cloud may upload the new files in 311340\remote (unknown).

### 2026-11-12 19:19: first passing automated run on the game (main session)

Observed (runner output pasted by the owner, hook b843d74-dirty, two copies on one PC, mission 10020):
save files host DYNAMITE_GAME_DATA / client DYN2_GAME_DATA; client connected on attempt 3 (two attempts 5 s apart
did not connect); connect PASS (2 members and sessionConnect on both sides); positions PASS (both players 0.01 m apart
between the two views, tolerance 2.0 m); positions oracle twin failed as it must (150.13 m); stability PASS
(59 samples over 60 s, both alive, 2 members throughout). The session twin (client does not connect) had started and
was waiting for the owner at the title; its result is not known yet. Title CONTINUE is still pressed by a person.

### 2026-11-12 19:28: session twin result

Observed (runner output from the owner): twin "client does not call CreateClientSession": both sides report 1 member,
connect failed as it must. Final table: connect ok, positions ok, positions twin ok, stability ok, connect twin ok,
RESULT: PASS. Write-up QA section updated with the summary and its limits; directory qa level left at `none` until
the owner decides (the run used a fork branch and a loopback transport, not the listed project's release).

### 2026-11-12: automating the title CONTINUE (started)

Known from the mod's Lua (TppMission.lua, TppMain.lua): `gvars.ini_isTitleMode` is true while the title runs over the
loaded mission. The title script itself (`title_sequence.fpk`) is not in the mod's sources and was not found in
TinManTex/mgsv-deminified-lua (data1 only). `mvars.seq_sequenceTable[<current>]._messageExecTable` holds the current
sequence's message handlers (TppSequence.lua), and TppMain.OnMessage looks up `OnlineChallengeTask.OnMessage` on every
message, so a wrapper there can record messages. Plan: `qa/probe_title.py` records sequence names, title flags, the
handlers and the messages while the owner goes through the title once; then make the runner press CONTINUE itself.
Not run on the game yet. The bridge is pumped on the title again (the old hang was the unbound lua_remove).

### 2026-11-12 19:40: title probe results (1.0.15.4, host copy, owner pressed the keys)

Observed (runs/20261112-194022-probe/timeline.log):
- mission 1: Seq_Mission_Prepare -> Seq_Demo_SetSavedLanguage -> Seq_Demo_CreateOrLoadSaveData ->
  Seq_Demo_LogInKonamiServer (5 s) -> Seq_Demo_ShowDlcAnnouncePopup (1 s) -> Seq_Demo_Init; `gvars.ini_isTitleMode` false.
- mission 40010 (heliSpace true from here on): Seq_Mission_Prepare (10 s) -> Seq_Demo_StartHasTitleMission (8 s) ->
  Seq_Game_TitleMenu (5 s) -> Seq_Game_MainGame. `gvars.ini_isTitleMode` is true until MainGame, where it turns false
  together with `mvars.mis_missionStateIsNotInGame`.
- Seq_Game_TitleMenu has OnSelectContinue / OnSelectGameStart / OnSelectMGO / OnSelectRestartHeli; its message table
  maps UI message StrCode32("Continue") = 1439467306 to OnSelectContinue (same function address).
- The OnlineChallengeTask.OnMessage wrapper saw nothing: OnlineChallengeTask was not a table with OnMessage ("none").
Unknown: which steps needed a key press from the owner (popups in mission 1, press-start in 40010).

Change: runner default `title_wait: auto`: waits for Seq_Game_TitleMenu, calls CoopQA.pressContinue() (the menu's own
OnSelectContinue), ready = heliSpace and Seq_Game_MainGame and not title mode and not notInGame. Simulation covers a
title that ignores CONTINUE (must fail). Not yet run on the game.

### 2026-11-12 19:50: first run with the runner pressing CONTINUE

Observed (runner output from the owner): `CoopQA.pressContinue()` returned ok and one second later the host was in
Seq_Game_MainGame: calling the menu's OnSelectContinue works. Two steps waited for the owner's key press:
1/Seq_Demo_LogInKonamiServer (38 s, 15 s on the client) and 40010/Seq_Demo_StartHasTitleMission (48 s, 39 s).
Master server log: CMD_GET_URLLIST/SVRLIST/SVRTIME arrive at the start of the login step, CMD_SEND_BOOT only after the
key press, so the wait is on the game side. On the client CONTINUE went 40010/Seq_Game_ChunkInstalled -> mission 10020
(Seq_Demo_ArrivalInAfghanistan ...): its DYN2 save had been left inside the mission by the earlier test runs.
Also found: `string.format("%d")` in the game's Lua is 32-bit, so the bridge JSON gave wrong values for integers
>= 2^31 (the probe's names.json lost every StrCode32 hash above 2^31; 160 of 328 names survived).

Changes (not yet run on the game): pressContinue picks the menu's OnSelectRestartHeli when
`gvars.title_nextMissionCode` is not a helicopter mission; the two key-wait steps get Enter via SendInput into the game
window, only when it is the foreground window (qa/winkeys.py, untested: no Windows here); the runner fails if the title
ends in a non-helicopter mission; integers are encoded with %.0f. Guess: Enter is the key those steps accept.
Unknown: what the two screens show. Dynamite upstream reports v0.6.0 available (launcher log); not looked at yet.

### 2026-11-12 20:07: first unattended run

Observed (runner output from the owner, nobody touched the games): Enter via SendInput ended
Seq_Demo_LogInKonamiServer (pressed 8 s into the step, next step 1 s later; Seq_Demo_ShowDlcAnnouncePopup was not seen
at 1 s polling) and Seq_Demo_StartHasTitleMission (pressed at 12 s) on both copies; pressContinue chose
OnSelectRestartHeli on both (title_nextMissionCode 10020) and each copy was in Seq_Game_MainGame one second later;
save files differ; client connected on attempt 1; connect, positions (0.01 m), stability (59 samples / 60 s) PASS.
Whole run 20:07:45 - 20:11:33. So Enter is accepted by both key-wait steps, and SetForegroundWindow worked for both
windows on the owner's Windows.

### 2026-11-12 20:20: D-Horse desync, start

Observed by the owner (manual session, 2026-11-09): both players called D-Horse; each mostly saw only their own; at one
point the client's horse appeared under the host, "almost in sync". A vehicle called by the host was invisible to the
client (the host was seen getting into nothing). Wanted: each player has their own horse, visible to the other.
Sources: Dynamite docs/MAKING_OF.md (Vehicles, Buddies) says the same and has no fix; upstream master (2026-09-29)
has no commit about vehicles or the horse beyond logging. The hook has no horse-specific code (grep).
Plan: measure first. `qa/experiment.py horse_probe` brings up the usual session, lists the buddy/GameObject Lua API
on both sides, calls D-Horse on the host, then on the client (TppBuddy2BlockController.CallBuddy: a guess from other
mods), and samples TppHorse2 positions and both player positions on both sides. Not yet run on the game.
