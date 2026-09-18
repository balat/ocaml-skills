# ocaml-skills

Claude Code skills for OCaml, written to be neutral in tone and to give the tools of the
OCaml ecosystem their full place. Two plugins so far:

| Plugin | Skills | For |
|---|---|---|
| `ocsigen` | `eliom-architecture`, `eliom-client-server`, `eliom-typed-markup`, `ocsigen-start`, `ocsigen-contributing` | Multi-tier web and mobile applications with Eliom, Ocsigen Start and Ocsigen Toolkit; contributions to the framework |
| `lwt` | `lwt` | Code using the Lwt cooperative concurrency library, on servers and in the browser |

Skills load on demand: only their name and description sit in context until a task matches.

## Installation

```
/plugin marketplace add balat/ocaml-skills
/plugin install ocsigen@ocaml-skills
/plugin install lwt@ocaml-skills
```

From a local checkout, for development:

```
claude plugin marketplace add /path/to/ocaml-skills
claude --plugin-dir /path/to/ocaml-skills/plugins/ocsigen
```

## Conventions

- Skills follow the Claude Code skill authoring guidance: a `description` that says what the
  skill does and when to use it, an imperative body under 500 lines, details in
  `references/`, one level deep.
- Module names in the Ocsigen skills are those of the development version of Eliom,
  Ocsigen Start and Ocsigen Toolkit (`Eliom.Service`, `Os.Session`, `Ot.Spinner`).
  `plugins/ocsigen/skills/eliom-client-server/references/module-names.md` maps them to the
  released names (`Eliom_service`, `Os_session`, `Ot_spinner`).
- English, no personal workflow rules, no machine-specific paths.

## Related

`avsm/ocaml-claude-marketplace` covers general OCaml development (project setup, testing,
odoc, Eio, cmdliner, and more). These plugins are meant to sit next to it.

## License

ISC.
