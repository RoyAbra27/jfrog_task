# Contextual Analysis, in my own words

## What it is

A traditional SCA scanner answers one question: does this artifact contain a
component version that has a known CVE? That is a lookup, not an analysis.
It tells you the vulnerable code is *present*, not that it is *reachable*.

Contextual analysis asks the second question: given this specific artifact,
can the vulnerable code actually be triggered? For each CVE it knows how to
check, JFrog Advanced Security scans the first-party code and configuration
in the image for the conditions the exploit needs - is the vulnerable
function called, is the risky configuration flag set, does attacker input
reach it - and stamps the CVE **Applicable** or **Not Applicable**, with the
evidence it found.

The result is not a different list. It is the same list with an
exploitability verdict attached to each line.

## A concrete before/after from this repo

The baseline image of this app produced **171 unique CVEs, 86 of them High
or Critical**. That is the "before": a wall of red that no team fixes this
sprint, so in practice a raw list gets triaged by severity score and gut
feeling.

The "after", with contextual analysis: **15 of those 86 High/Critical CVEs
are Applicable**, and only **2 are in the application's own npm tree**:

- `body-parser` (CVE-2024-45590) is Applicable because this app really does
  enable extended URL-encoded body parsing
  (`app.use(express.urlencoded({ extended: true }))` in `src/index.js`),
  which is precisely the exploit's precondition.
- `moment` (CVE-2022-31129) is Applicable because the library is imported
  and executed on every request in this app.

Compare that with `jsonwebtoken@8.1.0`: the scan marked its two High CVEs
(CVE-2022-23539, key-type algorithm confusion; CVE-2022-23540, unset
algorithms in verification) **Not Applicable** here, because their exploit
paths run through `jwt.verify` with attacker-influenced keys or algorithm
lists, and this app only calls `jwt.sign` with a fixed secret. Same
component, same CVE database, opposite verdict - because the verdict is
about this code, not about the component in the abstract.

## Why this changes remediation prioritization

1. **The queue shrinks to what can hurt you.** 86 High/Critical findings is
   a quarter of planning; 2 applicable app-level Highs is an afternoon. The
   63 Not Applicable ones are still recorded and re-evaluated on rescan,
   but they stop consuming urgent attention.
2. **Severity stops being the only sort key.** CVSS scores rate the
   vulnerability in the abstract; applicability rates it in your context. An
   Applicable Medium in code that parses untrusted input can matter more
   than a Not Applicable Critical.
3. **The evidence is auditable.** Each Applicable verdict points at the file
   and line that satisfies the exploit's precondition. When someone asks
   "why are we patching this first?", the answer is a code reference, not a
   feeling. The inverse also holds: Not Applicable verdicts carry the
   absence-of-precondition evidence, which is what makes deprioritizing
   defensible to a security reviewer.
4. **Honest limits.** Applicable does not mean "exploit proven", it means
   the known preconditions are present; Not Applicable is a verdict only for
   CVEs the analyzer has coverage for, and **Not Covered / Undetermined
   findings (8 of our 86 High/Critical) must keep their raw severity** -
   contextual analysis narrows the unknown, it does not license ignoring it.
   And it is a point-in-time verdict: code changes can flip a Not Applicable
   to Applicable, which is why it belongs in CI, not in a one-off audit.

## One sentence version

SCA tells you what is present; contextual analysis tells you what is
reachable in your code, with evidence - so remediation effort goes where an
attacker could actually get in, instead of where the CVSS number is largest.
