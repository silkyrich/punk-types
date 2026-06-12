# punk-types — the sourcetype library

Community-evolvable parsing knowledge for [punk-it](https://github.com/silkyrich/punk-it):
one directory per **sourcetype**, each holding the full parsing definition and a
pool of real sample lines to evolve it against.

```
<sourcetype>/
  def.json        the type definition:
                    ts       timestamp spec (locator regexes + strftime formats)
                    brk      event line-breaking (line | timestamp | starts+regex)
                    snapshot the cascading regex tree: roots classify, children
                             refine; every node has a name, named-capture regex,
                             capture types, state and origin
  samples.jsonl   sample pool: {"path": <matched pattern path or null>, "line": ...}
```

## The contract

- **source ≠ sourcetype.** A source is a stream URI (`file:///...`,
  `docker://host/plex`); a sourcetype is the *parsing identity* (`pihole-ftl`,
  `systemd-journal`). Streams declare their type with `?type=` on the URI.
- **Patterns are hypotheses.** Trees here carry zero runtime statistics; an
  engine importing them runs non-seed roots as *shadow candidates* that must
  earn graduation on live traffic (hit rate + sampled spot-check precision).
  Submitting a bad pattern can't hurt anyone's index.
- **Types normalize.** Captures named/typed mac, ipv6, level, date, time are
  canonicalized at extraction (`28-6B-..` → `28:6b:..`, `warn` → `WARN`).
- **Samples are evidence.** Improve a tree by testing against the pool
  (punk-it's Workbench tab, or any regex tool). Grow the pool with lines your
  pattern *fails* on. Redact anything sensitive before submitting.

## Using the library

```sh
git clone https://github.com/silkyrich/punk-types
punk-it types-import --lib punk-types --data-dir ./punk-data   # pull
# ...engine learns, workbench curates...
punk-it types-export --lib punk-types --data-dir ./punk-data   # push back
cd punk-types && git diff                                       # review evolution
```

## Contributing

PRs welcome: new sourcetypes, better trees, richer sample pools. A good
submission has (1) a named, anchored root that classifies the format cheaply,
(2) children extracting typed semantics (named captures, not f1/f2), (3) a ts
spec if the format's clock is unusual, (4) a brk spec if events are multiline,
(5) ≥20 redacted sample lines including edge cases.
