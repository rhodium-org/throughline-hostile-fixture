# throughline-hostile-fixture

A test fixture: three [throughline](https://pypi.org/project/throughline/) intent graphs
whose displayable fields are hostile on purpose.

An application that opens somebody else's repository has to display what it finds there.
A title, an item's text, an attribute value, a declared source's address and a tool's
own finding about the graph are all repository content, and a reader's browser will be
asked to render every one of them. This repository supplies content that will be
*acted on* rather than *shown* if any of those paths is wrong, so that a test can tell
the difference.

## Nothing here is an attack

Every payload does exactly one thing: it sets

```js
document.documentElement.dataset.smokePwned = '<which field>'
```

on the document that executed it. No network request, no storage, no navigation, no
third party. The sentinel exists because a test cannot otherwise tell *"the browser
executed this"* from *"the browser displayed the words of it"* — searching the page for
the payload's own text finds it in both cases, and finding it is the correct outcome in
one of them. Each field carries a distinct sentinel value, so a test that goes red names
the field that escaped.

Do not compose these graphs into a real project.

## Three graphs, three claims

There is **no graph at the repository root**, deliberately: a root graph would ingest the
subdirectories' registers and swallow their items. Each graph is opened by pointing an
application at its own directory.

### `sound/` — content is displayed, never obeyed

A graph that is **valid under `tl-compose check --strict`** and hostile in every field a
reader is shown: the project name, an intent's title and text, a requirement's title,
text and rationale, and both items' attribute values. The soundness is the point — a red
run must mean the encoding failed, and never that the fixture did.

It declares one source at

```
http://github.com/rhodium-org/throughline-medium-web
```

`http`, not `https`, and that is the reason it is here. An application that links what a
repository declared has to decide whether the browser will merely fetch the address or
act on it, and a fixture whose every address is benign cannot tell a live guard from a
removed one — the assertion passes either way. This address is fetched perfectly well
(GitHub redirects, and a browser client builds its own `https` requests from the owner
and repository names) while remaining an address no reader should be handed as a link.

An application should compose it and display it, and should hand the reader that address
as text rather than as a link.

### `findings/` — a tool's own sentence is displayed too

A graph that is **unsound on purpose**. A finding is a third kind of text a reader sees,
and unlike a title or a body it is assembled by the tools rather than copied from a
field — so an assertion about titles, text and attributes does not cover it. This graph
links to a target that does not exist and whose *name* is the payload, so the finding
quotes it:

```
[ERROR] SR-0001  dangling-link  link target 'INT-0002<img src=x data-injected onerror="…">' does not exist
```

The target carries no `/`. A target containing one is an external reference the core
deliberately leaves opaque, and an opaque target produces no finding to quote it in.

`tl check` reports three errors here: `unserved-root` on `INT-0001`, `dangling-link` and
`orphan` on `SR-0001`.

### `unfetchable/` — the address is refused before anything opens

A **sound** graph that declares a source at an address naming a scheme a browser
executes:

```
javascript:document.documentElement.dataset.smokePwned='source-url'
```

A browser application resolves declared sources into repositories while it is staging the
graph, so this address is parsed as a repository *before* any page that could render it as
a link exists — and parsing fails. Nothing in this graph is ever displayed: not the
project name, not the items, not a verdict, not the address itself. **The refusal is the
observable behaviour**, and an application should stop, name the address it could not
fetch, and open nothing.

The graph is otherwise sound so the refusal can only be read one way. If the items were
broken, an application that opened the graph and then reported errors would look the same
as one that refused the address.

## What a test should assert

Against `sound/` and `findings/`, after the page has rendered:

- the payload's **text** is on the page — displaying it is correct;
- `document.querySelectorAll('[data-injected]')` is **empty** — nothing became an element;
- no `[href]`, `[src]` or `[action]` value matches `/^\s*(javascript|data|vbscript)\s*:/i`;
- `document.documentElement.dataset.smokePwned` is **undefined**;
- for `sound/`, the declared address appears as text and **no anchor carries it**.

Against `unfetchable/`:

- the graph does not open, the failure names the address, and no item, verdict or source
  listing from it is rendered.

## Licence

Apache-2.0. See [`LICENSE`](LICENSE) and [`NOTICE`](NOTICE).
