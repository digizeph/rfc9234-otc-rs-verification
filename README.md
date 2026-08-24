# Strict Only to Customer (OTC) Verification on Route Server Sessions

`draft-herdes-idr-otc-rs-verification-00` — an Internet-Draft that **updates RFC
9234**. Individual submission targeting the IDR working group.

## The change

[RFC 9234][rfc9234] validates the OTC apex property asymmetrically:

| Route received from | OTC must equal remote AS? |
| ------------------- | ------------------------- |
| Peer (lateral)      | Yes — ingress rule 2      |
| Route Server        | **No — unchecked**        |
| Provider            | No, and correctly so      |

A Route Server is always the apex of any path that traverses it, exactly like a
lateral peer. A route arriving from an RS whose OTC value names some other AS
has two apexes, is not valley-free, and is a leak. RFC 9234 accepts it.

This draft adds the missing check as ingress rule 3:

> If a route with the OTC Attribute is received from an RS (i.e., remote AS with
> an RS Role) and the Attribute has a value that is not equal to the remote
> (i.e., RS's) AS number, then it is a route leak and MUST be considered
> ineligible.

That is the only normative change. Rules 1, 2 and 4 and the egress procedure are
unchanged.

**Why it matters.** The check is performed entirely by the RS-Client from the
session's remote AS number and the OTC value already on the route, so it still
works when the route server does not implement RFC 9234 — the case where RFC
9234's own protection (ingress rule 1 at the RS) never fires. No new capability,
no wire change, no coordination with the IXP or other clients.

**Why it is safe.** In a compliant deployment the OTC value seen over an RS
session is always the RS's AS number, whether the RS set it on egress or the
client set it on ingress. Rule 3 rejects leaks and nothing else. See Section 6.

**Rejected alternative.** Appendix A documents the stricter scheme where the
RS-Client also marks on egress to the RS and the RS verifies OTC against its own
AS. It is not specified: a client emitting OTC towards a conforming RS trips
ingress rule 1, so that RS would reject *every* route from the upgraded client.
It needs a per-IXP flag day ordered across independent parties.

## Building

```sh
make setup    # one-time: creates .venv with xml2rfc
make          # builds .txt and .html
make check    # offline checks (line length, non-ASCII)
make idnits   # submits the .txt to IETF author-tools
```

Current build: **0 errors, 0 flaws, 0 comments** (idnits 2.17.1). The remaining
warning is the expected "couldn't figure out when the document was first
submitted", which clears on posting.

BibXML references are vendored under `refs/` so builds work offline and behind
TLS-intercepting proxies; refresh with `make refs`.

## Scope

The document is scoped `Updates: 9234`, not `Obsoletes: 9234` — it specifies a
single delta to the ingress procedure rather than replacing RFC 9234. If it is
later respun as a full replacement, the header and the Section 5 framing need to
change.

[rfc9234]: https://www.rfc-editor.org/rfc/rfc9234.html
