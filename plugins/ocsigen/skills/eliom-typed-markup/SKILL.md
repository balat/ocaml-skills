---
name: eliom-typed-markup
description: "Generating HTML and SVG with TyXML in Eliom and Js_of_ocaml: Eliom.Content.Html and Svg, F versus D nodes, Manip and To_dom, typed links and forms checked against services, typed SVG attributes and transforms, replacing node contents without innerHTML. Use when building page content, widgets, dynamic or animated SVG, or whenever markup is about to be assembled as strings (Printf, innerHTML, insertAdjacentHTML, hand-written escaping)."
license: ISC
---

# Typed HTML and SVG in Eliom

Eliom pages are built with TyXML through `Eliom.Content.Html` and `Eliom.Content.Svg`. The
type checker rejects invalid nesting (a `p` inside a `p`, a `tr` outside a table) and the
library escapes text and attribute values. This skill covers how to use that machinery and
why string-built markup has no place in an Eliom application. Module names are those of the
development version (`Eliom_content` in released versions).

## F or D

`Eliom.Content.Html` provides two node constructors with identical signatures:

- `F` (functional): abstract values. `To_dom.of_element` builds a fresh DOM node each time.
  Default for content nothing refers to later.
- `D` (DOM): nodes carry a unique id. On the client, `To_dom.of_element` returns the actual
  node in the page. Use `D` for any element that client code binds events on, reads or
  mutates, and pass its handle to client code with `~%`.

Mixing is normal: a `D.div` container holding `F` children. Prefer `F`; a `D` node costs an
id attribute and a lookup.

## Manipulating D nodes

`Eliom.Content.Html.Manip` acts on `D` elements directly: `appendChild`, `appendChildren`,
`removeChildren`, `replaceChildren`, `removeSelf`, `Class.add`, `Class.remove`. Use it
instead of converting to a DOM node by hand. `Eliom.Content.Svg.Manip` does the same for
SVG. When a lower-level API is needed (measuring, focus, `setAttribute`),
`To_dom.of_element node` gives the `Dom_html.element Js.t`.

## Links and forms are typed

- `D.a ~service:Foo_services.user_page [txt "Profile"] userid`: the parameter is checked
  against the service definition. Changing a service's parameters is a compile error at
  every link.
- Forms: `Form.post_form ~service (fun (name, age) -> [Form.input ~input_type:`Text ~name Form.string; ...]) ()`.
  Field names come from the service's parameters.
- `~xhr:false` on a link forces a real HTTP request instead of an in-app transition; use it
  for server-only services (see `eliom-architecture`).

## Markup is never built as strings

Do not assemble HTML or SVG as text and inject it: no `Printf.sprintf "<div class=\"...\">"`
followed by `el##.innerHTML := Js.string s`, `insertAdjacentHTML` or `outerHTML`; no
hand-written `esc` helper for `<`, `>`, `&`, `"`; no `style="..."` strings (styling lives in
stylesheets, per-element geometry in typed attributes).

Why: string markup bypasses TyXML's validity checking, becomes an XSS vector as soon as any
value is attacker-influenced, breaks the XML namespace handling that `To_dom` performs for
SVG and MathML, and cannot be checked against the document model. When escaping by hand
seems necessary, the typed equivalent exists.

| Instead of | Use |
|---|---|
| `innerHTML := markup` to replace content | build the typed subtree, then `Manip.replaceChildren node [subtree]` |
| `innerHTML := ""` | `Manip.removeChildren node` |
| `setAttribute "data-act" v` | `a_user_data "act" v` (renders `data-act`) |
| an SVG string | `Eliom.Content.Svg.F` nodes (below) |
| `style="left: 10px"` | a class in the stylesheet, or a typed attribute for a value that varies per element |
| a hand-rolled `esc` function | nothing: `txt` and attribute constructors escape |

## SVG

`Eliom.Content.Svg` mirrors `Html`: `F`, `D`, `Manip`, `To_dom`. Build `Svg.F.g`, `path`,
`rect`, `circle`, `line`, `text`, `title`, ... with typed attributes, and embed the result in
HTML with `Html.F.svg ~a:[Svg.F.a_viewBox (0., 0., w, h); Svg.F.a_class ["chart"]] [...]`.
The HTML `svg` element takes SVG children and SVG attributes (`Svg.F.a_*`).

Numeric attributes are typed, not strings: lengths are `(float * unit option)` pairs,
transforms are a `Svg_types.transform list`, anchors are variants. `a_d` (path data) is a
string by the SVG specification itself. `references/svg.md` gives the attribute types with
examples and a complete drawing.

## Incremental updates and animation

- Re-rendering a typed subtree on each change or frame and swapping it in with
  `Manip.replaceChildren` is idiomatic, and fast enough for most interfaces.
- To adjust one attribute (a pan/zoom transform, a class toggle), keep the `D` handle and set
  that attribute on its DOM node, or use `Manip.Class.*`. This is legitimate DOM mutation,
  not string injection.
- For values that change continuously, `Eliom.Shared.React` (reactive nodes) can replace
  manual updates when it keeps the code simpler.
