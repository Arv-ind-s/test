The documents were written before the build and never edited by it, so they still describe every `CUT-01`
rung the gates may since have pulled. Read the cut lines out of prompt 3's gate reports and the
merge-evidence file and fill in `<...>` below — it is the one fact this briefing cannot get from a source,
and the rehearsal it feeds is the two minutes `spec §22` says a final is decided in.

```text
Use only the two uploaded documents and the cut list below. Write a study guide that teaches me my
own product fast and prepares me to answer questions about it on stage.

WHAT WAS CUT ON THE DAY, and it is not in either document — <the `CUT-01` rungs pulled, one per line,
each with what the demo shows instead; or `nothing was cut`>. Treat every one of these as not built:
never describe it as working, drop it from the demo beats and the feature list, and put it under
limits with what replaced it. A rung that is not on this list was not cut.

GROUNDING — this matters more than completeness:
- Every fact comes from the documents or that cut list. Add the section in brackets, like [spec §13];
  a fact that came off the cut list is tagged [cut list] instead — it has no section to cite.
- Nothing is invented — no number, tool, model name, benchmark, customer or comparison that is not
  written there. If something is missing, write `NOT IN THE DOCUMENTS` and move on.
- The documents mark how solid each claim is, and you keep that mark. `MEASURED` means somebody
  measured it. `USER-TESTED` means somebody outside the team was watched using it and the result is
  what they did, not what we predicted — say it that way, and never soften it into an estimate.
  `ESTIMATED` and `PROJECTED` mean nobody has measured or watched anything: say so in plain words —
  "we estimate this, we have not measured it." An `[ASSUMED-NN]` is something we assumed and have
  not checked. An `[OPEN-NN]` is a question nobody has answered yet — list it as one. A `LATER:`
  line, and any requirement marked `later`, is specced and not built. Never upgrade any of these.

STYLE: plain English, short sentences, no jargon unless you explain it in one line. Bullets and small
tables, not long paragraphs. Bold the words I must remember. Readable in ten minutes.

WRITE THESE SECTIONS:
1. The pitch — five sentences: who has the problem, what it costs them, what we built, what changes,
   why it is different.
2. The problem and the people who have it, with the evidence and numbers from the research.
3. What the product does — the user's journey step by step, in ordinary words.
4. What is new here — how it differs from what already exists, one line each.
5. The business case — the value, the success metrics, and how each is measured.
6. The tech stack — each layer, the exact version, and why it was chosen over the alternative.
7. How it works end to end — follow one request from the click to the answer on screen, naming each
   part it passes through and what it does there. Say which steps code decides and which the model
   decides; that split is the whole product.
8. The AI — which model, where it is called, what it decides, what it may never decide, how answers
   are grounded and checked, and what happens when it fails or the network is down.
9. Data — what is stored, where, which fields are personal, and what protects them.
10. Limits — what is deliberately out of scope and why, what is specced and not built, plus the main
    risks and our answer to each.
11. The demo — the run beat by beat, with what to say at each beat, as it stands after the cuts above.
12. Numbers to memorise — a table: the number · what it means · where it comes from · whether it was
    measured or estimated.
13. Judge questions — start from the six questions `spec §22` already wrote, keeping each answer and
    the `§N` it is read off. Then add nine more of the same kind, three sentences each, each naming
    the section its answer comes from. Include the hard ones: why does this need AI, what is actually
    new, does it scale, what breaks, what is not built yet.
14. Glossary — every term a non-expert would stumble on, one line each.
