# Eliom module names: development versus released

## Contents

- How to tell which naming a project uses
- Eliom
- Ocsigen Start and Ocsigen Toolkit
- Unchanged syntax

## How to tell which naming a project uses

- Released Eliom (opam, up to the 12.x series) exposes one top-level module per file:
  `Eliom_service`, `Eliom_content`, `Eliom_registration`, and so on.
- The development version exposes a single `Eliom` module with submodules (`Eliom.Service`),
  plus an `eliom-compat` library that re-exports the old names as aliases
  (`Eliom_service` is `include Eliom.Service`). Old code compiles unchanged when
  `eliom-compat.server` and `eliom-compat.client` are in the dune `libraries`.
- Check the project: `grep -rl 'Eliom\.Service\|Eliom\.Content' --include='*.eliom' .`
  versus `grep -rl 'Eliom_service' --include='*.eliom' .`; `opam show eliom` for the
  installed version; `eliom-compat` in the `dune` file.
- The skills in this plugin use the development names. In a project on released names,
  transcribe with the tables below: only the module path changes, the functions, types and
  labels underneath are the same.

## Eliom

| Development | Released |
|---|---|
| `Eliom.Service` | `Eliom_service` |
| `Eliom.Content` (`Html.F`, `Html.D`, `Svg`, `Manip`, `To_dom`) | `Eliom_content` |
| `Eliom.Registration` | `Eliom_registration` |
| `Eliom.Parameter` | `Eliom_parameter` |
| `Eliom.Client` | `Eliom_client` |
| `Eliom.Client_value` | `Eliom_client_value` |
| `Eliom.Reference` | `Eliom_reference` |
| `Eliom.Common` (scopes, `Eliom_404`, `Eliom_403`) | `Eliom_common` |
| `Eliom.Shared`, `Eliom.Shared_content` | `Eliom_shared`, `Eliom_shared_content` |
| `Eliom.React` | `Eliom_react` |
| `Eliom.Comet`, `Eliom.Bus` | `Eliom_comet`, `Eliom_bus` |
| `Eliom.Notif` | `Eliom_notif` |
| `Eliom.Lib` | `Eliom_lib` |
| `Eliom.Tools` | `Eliom_tools` |
| `Eliom.Config` | `Eliom_config` |
| `Eliom.State` | `Eliom_state` |
| `Eliom.Request_info` | `Eliom_request_info` |
| `Eliom.Cscache` | `Eliom_cscache` |
| `Eliom.Wrap` | `Eliom_wrap` |
| `Eliom.Route`, `Eliom.Mkreg`, `Eliom.Extension` | `Eliom_route`, `Eliom_mkreg`, `Eliom_extension` |

## Ocsigen Start and Ocsigen Toolkit

The same pattern applies: `Os.Session` was `Os_session`, `Os.Page` was `Os_page`,
`Os.Current_user` was `Os_current_user`, `Os.User`, `Os.Group`, `Os.Db`, `Os.Notif`,
`Os.Msg`, `Os.Email`, `Os.Types`, `Os.Handlers`, `Os.Services`, `Os.Tips`, `Os.Date`,
`Os.Lib`, `Os.Platform`, `Os.Request_cache`, `Os.Uploader`, `Os.User_view`,
`Os.User_proxy`, `Os.Fcm_notif`, `Os.Connect_phone`, `Os.Comet`, `Os.Icons` likewise.
Toolkit: `Ot.Spinner` was `Ot_spinner`, and so on for every widget module.

## Unchanged syntax

`let%server`, `let%client`, `let%shared`, `[%client ...]`, `~%`, `let%rpc`,
`[@@deriving json]`, `[%i18n ...]` and `[%pgsql ...]` are the same in both versions.
