---
name: eliom-client-server
description: "Correctness rules for code crossing the Eliom client/server boundary: let%server/let%client/let%shared annotations, [%client] values and when they execute, Eliom.Client.onload, injections ~%, injecting services once at module load, [@@deriving json], 31-bit client integers, decoding strings to variants at the boundary, Lwt_js_events loops and RPC error handling. Use when writing or reviewing .eliom/.eliomi files, or debugging 'cannot wrap functional values', a client effect that never fires, or numbers corrupted only in the browser."
license: ISC
---

# Eliom client/server boundary

One `.eliom` file holds server code, client code and code compiled for both. Most bugs
specific to Eliom come from values crossing that boundary: when they are captured, how they
are serialised, and when the client code receiving them runs. Module names are those of the
development version; `references/module-names.md` maps them to released names.

## Sections and annotations

- Annotate every top-level definition: `let%server`, `let%client`, `let%shared`,
  `type%client`, `module%server`, `open%client`. Prefer these to `[%%client.start]` and
  `[%%server.start]` blocks, which make the tier of a definition depend on distant context.
- Annotate server code too (`let%server` rather than bare `let`): the intent stays visible
  and survives reordering.
- Keep `open%...` to a few well-known modules (`Js_of_ocaml`, `Js_of_ocaml_lwt`,
  `Eliom.Content`, `Eliom.Content.Html`), each in the section where it applies. Refer to
  other modules qualified.
- Do not write JavaScript (no `Js.Unsafe.eval_string`, no inline scripts). Client behaviour
  is OCaml in client sections; Js_of_ocaml bindings cover the browser APIs.

## Types across the wire

- Type annotations are mandatory on client values, `[%client (expr : t)]`, and on every
  parameter and the result of `let%rpc` functions. Inference does not cross tiers, and the
  errors produced without annotations point elsewhere.
- Custom types passed through injections or RPCs derive `json`: `type t = ... [@@deriving json]`.
- Client integers are 31-bit under js_of_ocaml and wasm_of_ocaml. Any value that may exceed
  2^30 (database ids, timestamps, amounts in cents, hashes) crosses into `%client` or
  `%shared` code as `Int64` or as a string. `int` arithmetic on the client corrupts such
  values silently while the 63-bit server does not, so the bug shows only in the browser.
  Ocsigen Start user ids are `int64` for this reason.
- A closed set of values that travels as a string (URL parameter, RPC field, cache key) is
  decoded once, at the boundary, into a `[@@deriving json]` variant and pattern-matched from
  there. Threading the raw string and re-comparing it with `| _ ->` fallbacks lets the two
  ends drift apart.

## Client values: what runs, and when

A `[%client expr]` block inside rendering code creates a client value: `expr` runs on the
client each time the enclosing rendering runs.

- Every client section reached during rendering runs, whether its value is bound, ignored
  or placed in the DOM. `ignore [%client (e : unit)]` is enough; wrapping it in a placeholder
  `span` changes nothing.
- When it runs depends on which tier rendered the page:
  - Server-rendered page (first request): client sections run after the browser has loaded
    the page. The DOM is in place.
  - Client-rendered page (navigation through `Eliom.Client.change_page`): client sections
    run as soon as the rendering function returns, before the new DOM replaces the current
    one. A `Lwt.async (fun () -> Eliom.Client.change_page ...)` at the top of such a section
    is clobbered by the transition still in progress, and the navigation appears not to
    happen.
- A handler usually serves both paths, so defer anything that touches the freshly rendered
  page (programmatic navigation, focus, measurements) with
  `Eliom.Client.onload (fun () -> ...)`. It runs once the page is in place under both
  rendering paths.
- To wait for one node to be attached to the document, use `Ot.Nodeready.nodeready node`
  (Ocsigen Toolkit).

## Injections

`~%v` inside a client section captures the server value `v` at rendering time and
serialises it to the client.

- Inject data only: strings, numbers, records with `[@@deriving json]`, `D` node handles,
  `Eliom.Client_value.t` produced by another `[%client]`. Functions, channels and other
  non-serialisable values cannot cross.
- An injected value is a snapshot. For values that must keep updating on the client, use
  `Eliom.Shared.React` signals.
- Inject services and RPC stubs exactly once, at module load:

  ```ocaml
  let%server foo_service = Eliom.Service.create ...
  let%client foo_service = ~%foo_service
  ```

  This runs before any `App.register ~service:foo_service ...`. Registration attaches a
  closure to the server-side service object, which then cannot be serialised. A later
  `~%foo_service` inside a `[%client]` block in a page handler raises
  `Failure "cannot wrap functional values"` on the server; the failure is logged, the
  response is still sent with the injection dropped, and the client-side effect never
  fires. Inside client sections, use the module-level client identifier
  (`Foo_services.foo_service`), never `~%` again. The same holds for `let%rpc` stubs and any
  top-level value that gains a closure after server-side wiring.

## Client-side events and errors

- Bind events with `Lwt_js_events` (`clicks`, `mousedowns`, `changes`, ...) or attributes
  such as `a_onclick`. Functions with an `s` suffix loop over repeated events; the singular
  form waits for one occurrence.
- Wrap every RPC call made from client code in `Lwt.catch`: show the user a message
  (`Os.Msg.msg` in Ocsigen Start), log the detail, never fail silently. Inside an event loop
  this matters most: an uncaught rejection ends the `Lwt_js_events.clicks` loop, and the
  button stays dead until the next render with nothing shown. Put the catch in one shared
  click-handler helper rather than at each call site.
- Use `Lwt.catch` or `try%lwt`, not `try ... with`, around code that returns promises (see
  the `lwt` skill).

## References

- `references/checklist.md`: the rules above as a review checklist for a diff touching
  `.eliom` files.
- `references/module-names.md`: development names (`Eliom.Service`) versus released names
  (`Eliom_service`), and how to tell which one a project uses.
