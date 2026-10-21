# civics-test-kit

The complete **USCIS naturalization civics test** as a dependency-free TypeScript engine. Built for [Oath](https://github.com/nityanand123gupta/oath-alexa-civics-coach) during the Amazon Developer Hackathon 2026 and extracted as a standalone package so any citizenship-prep tool — voice agent, mobile app, classroom tool, Discord bot — gets the hard parts for free.

## What's inside

- **Both official question banks**, transcribed verbatim from USCIS publications and machine-verified (answer counts asserted per question against the source PDFs):
  - `bank-2008.json` — 100 questions (applies if Form N-400 was filed **before Oct 20, 2025**)
  - `bank-2025.json` — 128 questions (filed **on/after Oct 20, 2025**; [Federal Register 2025-18050](https://www.federalregister.gov/documents/2025/09/18/2025-18050/notice-of-implementation-of-2025-naturalization-civics-test))
  - per-question flags: `senior` (65/20-designated) and `dynamic` (answers that change with elections/location, with USCIS's own notes)
- **Deterministic answer matcher** — checks a spoken/typed answer against the official acceptable answers: parenthetical expansion ("twenty-seven (27)" ⇒ "27"), number words ⇄ digits, initialism collapse ("D.C." ⇒ "dc"), article/word-order tolerance, multi-answer counting ("Name three…" needs three distinct matches), numeric exact-match guard, and **asymmetric coverage** (never symmetric fuzz — "to spread communism" must not match "to stop the spread of communism"). Shipped with an 87-case labeled eval (0 false-accepts).
- **Authentic exam mechanics** — version-correct state machines with the real early-stop rules (2008: up to 10 asked, pass at 6, fail at 5; 2025: up to 20, pass at 12, fail at 9; 65/20: designated-only, 10 asked, pass at 6).
- **SM-2 spaced repetition** + per-question mastery estimates.
- **Seeded Monte Carlo readiness** — simulates the learner's actual exam N times against their mastery for a reproducible pass probability.

## Install & use

```bash
npm install civics-test-kit
```

```ts
import {
  getBank, testVersionForFilingDate, qualifies6520,
  matchAnswer, createExam, advanceExam, currentQuestionId,
  newProgress, recordAttempt, estimateReadiness, seededRng,
} from "civics-test-kit";

const version = testVersionForFilingDate("2026-01-15");    // "2025"
const bank = getBank(version);

// authentic mock interview
let exam = createExam(bank, "mock_interview", { rng: seededRng(42) });
const q = bank.questions.find(x => x.id === currentQuestionId(exam))!;
const verdict = matchAnswer(q, "the constitution");         // correct | incorrect | self_check
exam = advanceExam(exam, bank, verdict.kind === "correct"); // stops early exactly like the real exam

// readiness
const report = estimateReadiness(bank, new Map(), { simulations: 10000, seed: 7 });
console.log(report.passProbability);
```

## Quality

```bash
npm install && npm test   # 41 unit tests
npm run eval              # 87-case labeled matcher eval — 100%, 0 false-accepts
```

Data provenance (sources, verification method, spot-checks) is documented in the [Oath repository](https://github.com/nityanand123gupta/oath-alexa-civics-coach/tree/main/docs/data-provenance).

## Honest limitations

- The matcher is deliberately **stricter than a USCIS officer** (a coach should over-prepare you): some officer-acceptable paraphrases are rejected, never the reverse.
- `dynamic` questions ("Who is the President?") return `self_check` — current officeholders must be verified at [uscis.gov/citizenship/testupdates](https://www.uscis.gov/citizenship/testupdates).
- Covers the civics portion only, not the English reading/writing tests.

## License

MIT. The USCIS questions and answers are US-government works (public domain, 17 U.S.C. § 105). Not affiliated with USCIS; not legal advice.
