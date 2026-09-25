# ocaml-skills

Claude Code skills for OCaml libraries and frameworks. Plugins:

| Plugin | Skills | For |
|---|---|---|
| `ocsigen` | `eliom-architecture`, `eliom-client-server`, `eliom-typed-markup`, `ocsigen-start`, `ocsigen-contributing` | Multi-tier web and mobile applications with Eliom, Ocsigen Start and Ocsigen Toolkit; contributions to the framework |
| `lwt` | `lwt` | Code using the Lwt cooperative concurrency library, on servers and in the browser |

Skills load on demand: only their name and description sit in context until a task matches.
The `ocsigen` skills refer to the `lwt` skill for concurrency details; install both plugins.

## Installation

```
/plugin marketplace add balat/ocaml-skills
/plugin install ocsigen@ocaml-skills
/plugin install lwt@ocaml-skills
```

From a local checkout, either register it as a marketplace and install from it:

```
claude plugin marketplace add /path/to/ocaml-skills
claude plugin install ocsigen@ocaml-skills
```

or load one plugin for a single session, without installing:

```
claude --plugin-dir /path/to/ocaml-skills/plugins/ocsigen
```

## Module names

The Ocsigen skills use the module names of Eliom 13: `Eliom.Service`,
`Os.Session`, `Ot.Spinner`. Eliom 12 and earlier use
`Eliom_service`, `Os_session`, `Ot_spinner`. The mapping, and how to tell which naming a
project uses, is in
`plugins/ocsigen/skills/eliom-client-server/references/module-names.md`.

## Documentation

Upstream documentation for the libraries these skills cover: [Eliom](https://ocsigen.org/eliom),
[Ocsigen Start](https://ocsigen.org/ocsigen-start), [Ocsigen Toolkit](https://ocsigen.org/ocsigen-toolkit),
[TyXML](https://ocsigen.org/tyxml), [Lwt](https://ocsigen.org/lwt), and the
[tutorial](https://ocsigen.org/tuto/latest/manual/basics).

## Conventions

- Skills follow the Claude Code skill authoring guidance: a `description` that says what the
  skill does and when to use it, an imperative body under 500 lines, details in
  `references/`, one level deep.
- English, no personal workflow rules, no machine-specific paths.
- Each plugin ships eval cases under `evals/`; run them with `claude plugin eval plugins/ocsigen`
  (or `plugins/lwt`) after changing a skill.

## Related

[avsm/ocaml-claude-marketplace](https://github.com/avsm/ocaml-claude-marketplace) covers
general OCaml development (project setup, testing, odoc, Eio, cmdliner, and more). These
plugins are meant to sit next to it.

## License

ISC.
