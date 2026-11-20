# Engineering notes

Three decisions from the marketing-site and tooling work, with the numbers that drove
them. Written for publication; the working decision log they come from stays private.

## Islands over a rewrite

The homepage needed real motion. The obvious framing was "we need React", and the obvious
conclusion from that framing is a full rewrite.

That framing was wrong, and noticing why is the whole decision. The site is static-first
with tuned SEO output, a serverless contact endpoint, and content collections. A rewrite
would have replaced all of it to change **zero pixels**, because a framework does not
change how a page looks. Motion was the requirement; the framework was a proposed
implementation that had quietly been promoted to a requirement.

So React went in as **islands**, scoped to the one page that needed it:

- An animated-height FAQ disclosure, opt-in per page so other routes keep the native
  element
- A spring pointer-tilt wrapper around server-rendered cards
- A viewport-triggered reveal component

Everything hydrates lazily on visibility. React ships on the homepage only. Other routes
stay entirely framework-free.

Results: 24 pages, largest-contentful-paint effectively unchanged (~140 ms against a 133 ms
baseline), accessibility held at its existing score.

**The generalizable bit:** when someone proposes a rewrite, check whether the stated
problem is a property of the framework or a property of the design. "It looks the same" is
a design problem. No framework migration has ever fixed one.

## What a motion pass actually has to hold

A later pass added a scroll-reveal system to a different build. The interesting part is
not the animation, it is the constraints it had to survive:

- **Visible by default.** Reveal elements render visible and are hidden only once the
  observer is running. Get this backwards and reduced-motion users, or anyone with a
  script failure, get a permanently blank page. This is the single most common way
  scroll-reveal breaks, and it fails closed in the worst direction: invisible content
  looks like a working page.
- **The LCP element is never animated.** Whatever the browser picks as the
  largest-contentful-paint node stays untouched, or the metric measures the animation
  rather than the page.
- **CSS-only wherever possible.** Hovers and tints need no JavaScript.

Verified: LCP 1365 ms and CLS 0.00 on simulated mid-range mobile, Lighthouse mobile
accessibility, best-practices and SEO all 100, reveals firing 13 of 13 with no stuck
content in either theme, and a background payload of 81.6 KB.

CLS 0.00 is the one to defend. Motion that shifts layout is not a polish improvement, it
is a regression with better marketing.

## Measure the tooling before trusting it

Agent tooling had made sessions slow and worse. Rather than guess, the config got audited.

What one plugin marketplace actually cost:

| Measure | Value |
|---|---|
| Skills / agents / commands shipped | 894 / 306 / 424 |
| Hook commands registered | 21, four matching `*` |
| Descriptions injected into every system prompt | ~197,000 chars, roughly 49K tokens |
| Times its skills were invoked across 249 logged commands | **zero** |

A `*` hook means a process spawned on every single tool call. Its gate hook blocked five
operations during the audit session alone, each demanding a written justification before a
directory listing or a file write.

Six plugins with zero recorded invocations were removed, along with three marketplaces.

**The lesson is the method, not the verdict.** Installed capability feels free and is not.
It costs context on every request and latency on every tool call, and the cost is invisible
because nothing reports it. The invocation count was the number that settled the argument,
and nobody had looked at it before. Instrument what you install, then let the measurement
decide.

One nuance worth keeping: the marketplace was kept on trial rather than removed, because
it had never actually been given a fair evaluation. A measurement that says "unused" is
evidence about the past, not proof about the future. Removing something unevaluated and
removing something evaluated are different acts.
