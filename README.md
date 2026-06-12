# punk-types — the sourcetype library

Community-evolvable parsing knowledge for [punk-it](https://github.com/silkyrich/punk-it):
organized as **group/package/sourcetype** (so a thousand types stay navigable),
each type holding the full parsing definition and a pool of real (redacted)
sample lines to evolve it against.

```
<group>/<package>/<sourcetype>/   e.g. network/dns/pihole-ftl, os/linux/systemd-journal
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

## punkscript — trees as programs

Every tree has a textual form (mtail-flavored): nesting is the cascade,
`from <field>` steers a child to parse a parent capture instead of the raw
line, and statements compile to field ops. Edit trees as text, compile back:

```
type pihole-ftl in network/dns

`^(?P<date>\d{4}-\d{2}-\d{2}) (?P<time>\S+) (?P<level>[A-Z]+): (?P<msg>.*)$` as ftl_log {
    msg ?= "(empty)"                       # default
    `^Connection error \((?P<upstream_ip>[\d.]+)#(?P<dns_port>\d+)\)` from msg as upstream_err {
        subsystem = "dns-upstream"
        if level == "WARNING" { alert = "upstream-flap" }
    }
}
```

```sh
punk-it decompile pihole-ftl --lib .   # tree -> text
punk-it compile my-edit.punk --lib .   # text -> tree (upsert; stats kept)
punk-it serve --lib . --addr 127.0.0.1:7676   # design mode: workbench +
    # sample pools + install straight into this checkout, no engine needed
```

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
