---
name: eliom-architecture
description: "Structuring multi-tier Eliom/Ocsigen web and mobile applications: shared, server and client sections, server-side first render then client-side navigation, service design (GET vs POST, registration on both tiers, server-only services and ~xhr:false links), RPCs with ocsigen-ppx-rpc, Eliom references and scopes, feature file layout (foo_services, foo_handlers, foo_db). Use when creating or reorganising an Eliom app, adding a page, service or feature module, deciding which tier code runs on, or choosing where state lives."
license: ISC
---

# Eliom application architecture

Eliom compiles one OCaml program into a server (native or bytecode) and a client (JavaScript
or WebAssembly through js_of_ocaml or wasm_of_ocaml). One source describes both tiers; ppx
section annotations decide where each definition lives. Documentation:
<https://ocsigen.org/eliom> (manual and API) and the tutorial
<https://ocsigen.org/tuto/latest/manual/basics>.

Module names below are those of Eliom 13 (in development): `Eliom.Service`, `Eliom.Content`.
Eliom 12 and earlier, that is every opam release to date, use `Eliom_service`,
`Eliom_content`, and so on. `../eliom-client-server/references/module-names.md` gives the
mapping and how to tell which naming a project uses.

## Rendering model

- First request: the server renders the full HTML page. Search engines, and users without
  the client program, see an ordinary web site.
- Later navigation: the client program stays alive. Typed links and forms call
  `Eliom.Client.change_page`, which renders the next page in the browser without reloading.
  The application has the responsiveness of a single-page app while keeping URLs, links,
  forms and the back button.
- Data the client needs comes through RPCs. Render blocks that wait for data inside
  `Ot.Spinner` (Ocsigen Toolkit) so the rest of the page appears at once instead of waiting
  for the slowest call.
- Typical targets: a web app in browsers (server render first, client afterwards) and a
  mobile app in a Cordova webview (client rendering only, same server). Design responsive
  layouts from the start when both are targets.

## Which tier runs what

Write page content, widgets and handlers in shared sections so that pages can be generated
on either tier. Reserve single-tier sections for what genuinely belongs there:

| Tier | Belongs there | Annotation |
|---|---|---|
| Server only | database, file system, secrets, external services, authorization decisions | `let%server` |
| Client only | DOM manipulation, browser and device APIs, UI-only behaviour | `let%client` |
| Both | page rendering, widgets, page service registration | `let%shared` |

Register page services on both tiers, `let%shared () = App.register ~service ...`, where
`App` is the module the application obtains from `Eliom.Registration.App`; in-app navigation
then renders them on the client. Register on the server only when the service is not part
of the client program: REST or JSON endpoints, downloads, printable pages, services called by
third parties. Link to such a service with `~xhr:false` (`Eliom.Content.Html.D.a ~service
~xhr:false`) so the browser performs a real request instead of a client-side transition.

Client-server reactive programming (`Eliom.Shared.React`) is available; use it where it
makes the code simpler than rendering plus RPCs, not by default.

## File layout

Each feature `foo` is split by role. Give every `.eliom` used by other modules an `.eliomi`
exposing only what they need.

| File | Content |
|---|---|
| `foo_services.eliom(i)` | service definitions only |
| `foo_handlers.eliom(i)` | handlers: page generation, form processing |
| `foo.eliom(i)` | page content, widgets |
| `foo_container.eliom(i)` | layout shared by the feature's pages |
| `foo_db.ml` | server-only database access (plain `.ml`, never `.eliom`) |
| `foo_config.eliom` | configuration values |

Keep services and handlers in separate files: links and forms everywhere refer to services,
while handlers pull in the whole feature. Split a role file further when a feature grows.
This is the convention of the Ocsigen Start template; keep it consistent across the project.

## Services

- Bookmarkable pages are GET services:

  ```ocaml
  let%server user_page =
    Eliom.Service.create
      ~path:(Eliom.Service.Path ["users"])
      ~meth:(Eliom.Service.Get Eliom.Parameter.(int64 "id"))
      ()
  ```

- Actions with effects (create, update, delete) are POST services, typically registered with
  `Eliom.Registration.Action` or `Eliom.Registration.Redirection`.
- Parameters are typed (`Eliom.Parameter.(int "page" ** string "q")`, `int64`, `suffix`).
  Links and forms are checked against them at compile time (see `eliom-typed-markup`).
- Remote calls: `let%rpc f (x : t) : u Lwt.t = ...` with `ocsigen-ppx-rpc`. Typing and
  serialisation rules are in the `eliom-client-server` skill.
- Multi-step flows can register services with a `~scope` (session or client process) so
  that a stale URL or the back button cannot replay a step out of context.

## State

Choose the narrowest scope and the right store:

- `Eliom.Reference.eref ~scope init` holds server-side state per scope, volatile by default,
  persistent with `~persistent`. Scopes, broadest to narrowest: `Eliom.Common.global_scope`
  (whole server), `site_scope` (one application), `default_group_scope` (a group of sessions,
  typically one user), `default_session_scope` (one browser), `default_process_scope` (one
  tab, or one mobile app instance).
- Ocsipersist (Ocsigen's key-value store, with SQLite, PostgreSQL and DBM backends) for
  application data that must survive restarts but has no other reader (caches, counters).
- The database (PostgreSQL through PG'OCaml in Ocsigen Start) for durable data that other
  tools may query.
- Ocsigen Start adds scopes that survive login and logout (see the `ocsigen-start` skill).

## Trust boundary

The client program is under the user's control. Every RPC and every server handler
re-checks authorization on the server; client-side gating is a convenience for the user
experience. Never accept the current user's id from the client: the server knows it
(`Os.Current_user`). Details in the `ocsigen-start` skill.

## Errors and performance

- Handle HTTP errors with Eliom's error handler (`Eliom.Registration.set_exn_handler`,
  matching `Eliom.Common.Eliom_404` and `Eliom.Common.Eliom_Wrong_parameter`); do not let
  exceptions escape service handlers.
- Do not sequence independent remote calls (RPCs, queries): start them together and wait
  for both (`Lwt.both`, `Lwt_list.map_p`; see the `lwt` skill, plugin `lwt` of this
  marketplace).
