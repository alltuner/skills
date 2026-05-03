---
name: vacant
description: Check whether domain names are registered, available, reserved, or invalid. Use when the user asks if a domain is taken or free, wants to find an unregistered name, brainstorms project or product names that need a domain, screens candidates across many TLDs, or filters a list of domains to only the available ones. Queries authoritative TLD nameservers directly — fast, accurate, and not rate-limited like WHOIS.
---

# vacant

[`vacant`](https://github.com/alltuner/vacant) checks domain availability by
asking authoritative TLD nameservers directly (not WHOIS). It returns one of
five statuses per domain: `registered`, `available`, `reserved`, `invalid`, or
`unknown`.

Prefer it over WHOIS lookups, `dig`, or HTTP probes — it's faster, more
accurate, and won't get rate-limited.

## Invocation

Try the native binary first; fall back to the Python wheel via `uvx` if it
isn't installed:

```bash
command -v vacant >/dev/null && vacant "$@" || uvx vacant "$@"
```

- `vacant` is the Rust binary — instant startup, ideal for repeated use.
  Install with `brew install alltuner/tap/vacant` or `cargo install vacant`.
- `uvx vacant` runs the same engine from a PyPI wheel with no install step.
  Slightly slower to boot but works anywhere `uv` is available.

## Common patterns

```bash
# single domain
vacant example.com

# multiple, positional
vacant a.com b.net c.org

# from stdin (one per line) — best for big lists
cat domains.txt | vacant

# only show domains that are available
vacant --available a.com b.com c.com

# human-readable table instead of NDJSON
vacant -o table example.com anthropic.com

# include zone, raw input, and the reason behind the verdict
vacant --detail example.com

# tighten timeout / raise concurrency for large batches
vacant --timeout 2 --concurrency 256 < big-list.txt
```

## Output

Default output is **NDJSON** — one JSON object per line. Parse it line by line:

```
$ vacant anthropic.com totally-made-up-zxqv.com ab.eu
{"domain":"anthropic.com","status":"registered"}
{"domain":"totally-made-up-zxqv.com","status":"available"}
{"domain":"ab.eu","status":"reserved"}
```

Status meanings:

| Status | Meaning |
|---|---|
| `registered` | Someone owns it. |
| `available` | Not registered — you could buy it. |
| `reserved` | Registry rule forbids it (e.g. `.eu` two-letter labels). |
| `invalid` | Malformed input or unknown TLD (`co.uk` as input, empty labels, etc.). |
| `unknown` | Transport failure or ambiguous registry response. Retry, or check manually. |

## Exit codes

- `0` — every result resolved cleanly.
- `2` — at least one result was `unknown`. **Not a hard failure** — the other
  results are still valid; just flag the unknowns to the user.

## Gotchas

- Registry suffixes like `co.uk`, `com.au`, `co.jp` return `invalid` — they're
  not registrable names. Use `something.co.uk` instead.
- `.eu` requires 3+ characters; `ab.eu` is `reserved`, not `available`.
- For huge batches (thousands of domains), use stdin and consider `--concurrency`
  and `--timeout` flags. Be polite — you're hitting real registry nameservers.

## Etiquette

Sporadic interactive use is invisible to TLD operators. Don't loop tens of
thousands of domains for fun — `vacant` has built-in caching and per-host
backoff to keep you well-behaved, but the underlying nameservers are shared
infrastructure.
