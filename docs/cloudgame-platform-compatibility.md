# Cloud Star Rail launch compatibility

Reproduced on MacBook Neo, macOS 27.2, `com.miHoYo.cloudgames.hkrpg` 4.7.0.
The custom “游戏启动失败 / 参数错误” dialog displays the backend response from
`POST /hkrpg_cn/cg/dispatcher/api/statusCheck`; CGSDK `startGame` is never reached.

LLDB captured these three variants on the same installation:

| Variant | `isiOSAppOnMac` | `x-rpc-client_type` | `x-rpc-sys_version` | Device model | Response |
| --- | --- | --- | --- | --- | --- |
| Original PlayTools | true | 12 | 27.2_27.2 | iPhone16,2 | -110002, 参数错误 |
| PlayTools removed temporarily | false | 1 | 27.2 | arm64 | 0, OK |
| PlayTools retained, only concrete ProcessInfo getter restored | false | 1 | 27.2 | iPhone16,2 | 0, OK |
| Compiled PlayTools compatibility fix, diagnostic plugin removed | false | 1 | 27.2 | iPhone16,2 | 0, OK |

The compatibility override runs before the application's main routine, only for this bundle
and only when `isMacCatalystApp` is true. It restores the native Catalyst value
on the actual ProcessInfo instance's runtime class. Keychain, input, device model,
and screenshots remain available. The successful check does not by itself verify
an entire cloud game session.

Validation: arm64 Release build succeeded. The framework and installed Neo game
passed `codesign --verify --deep --strict`. The compiled framework was loaded from
the game's own Frameworks directory, and LLDB detached after capturing the
successful startup check.

Related upstream report: https://github.com/PlayCover/PlayCover/issues/2066
