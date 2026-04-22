# ESP chatbot (`esp-chatbot.html`) — session notes

Single-file in-browser bot: `KNOWLEDGE_BASE`, `SUPPORT_CASES`, and ordered handlers in `getBotReply()`.

## Related question paraphrases (same intent, different wording)

- **`applyRelatedQuestionParaphrases(normalized)`** runs after **`normalizeText()`**; **`getBotReply()`** uses **`tokenizeFromNormalized()`** so tokens match the paraphrased text.
- Adds explicit phrase mappings only (not fuzzy similarity): e.g. attend/watch webinar → join, when is next class/course → next webinar, refund wording, digital/video case study variants, textbook price phrasing, X-ZONE “tell me about”, CME “where are my”, Alliance synonyms.
- **`collapseSpacedTypos()`** — before paraphrases; fixes **`r e fund` → `refund`**, spaced **`cards` / `ca res`**, etc. **`looksLikeSpacedRefundIntent()`** is a second-line match on spaced **`refund`** spellings.
- **Garbled KB guard**: weak `KNOWLEDGE_BASE` hit (score < 3) for `registration_includes` or `study_tools` plus fragmentary letter-spacing → `CLARIFY_REPHRASE_MISSPELLING`.
- **Low KB score (< 1.7)**: `CLARIFY_REPHRASE_MISSPELLING` only when `seemsFragmentaryLetterSpacing`; otherwise `CONTACT_OR_LIVE_AGENT`. Early `seemsGibberish` uses the same contact prompt + live agent. `FALLBACK_ANSWER` matches that string.
- **`getWorkbookExplainer()`** (in **`getSpecificTopicReply`**) — “what are the workbooks” style (not shipping-only). **`getPurchaseOrderReply()`** — place/make/order/purchase phrasing (excludes cancel/change order, bulk textbook orders).

## Webinar login / join vs “next webinar” dates

- **`asksWebinarLoginOrJoin()`** + **`getWebinarLoginJoinReply()`** — run early (after `getPriorityPolicyReply`). Answer: log in → dashboard → Webinars / webinar block → Join (Zoom; green ~30 min before); email a few days before.
- **`getGeneralNextWebinarReply()`** — skips when login/join intent is detected so “log on to the **next** webinar” does not return May/June date blurbs.

## Textbook “how much?” follow-ups vs webinar pricing

- **`getContextualShortPriceReply()`** — runs after **`getRefundTransferPriorityReply()`**, before **`getPriorityPolicyReply()`**. Short price asks (≤8 words, no named product) use **`lastTopic`**: e.g. after **`textbooks`**, return textbook prices ($129 / $175 + bulk tiers), not generic webinar pricing.
- Vague first message (“how much” / “cost” with no topic, ≤4 words) → **clarify** which product; **`clarify_price`** is ignored when resolving context for a repeat.
- **`KNOWLEDGE_BASE` `pricing`** — removed bare **`how much`** / **`cost`**; added specific phrases + **`cost of webinar`**, **`webinar cost`**, etc.

## Refunds and typos (“ar efund”)

- **`getRefundTransferPriorityReply()`** — runs very early. Detects **`refund`**, **`efund`** (typo), **`re fund`**, money back, cancel registration, transfer course, refund policy, etc.
- **`registration_includes`** — removed loose **`what do i get`** from patterns (it overlapped “can i get…” via token **`get`**). Longer phrases kept; exact **`what do i get`** / **`what will i get`** / **`what am i getting`** handled in **`getSpecificTopicReply()`**.

## Quick retest phrases

- “How do I log on to the webinar?” / “…to the **next** webinar?” → join instructions, not dates.
- After “what are the textbooks?” → “how much” → textbook prices.
- “Can I get ar efund” → refund policy, not registration-includes blurb.

## Normalization & course aliases

- **`normalizeText()`** — `whens`→`when`, `whats`→`what`, `wheres`→`where`, `quiz cares`→`quiz cards` (typo fix only).
- **`findCourseKey()`** — token **`adult`** → **AE** (e.g. “for adult” after a next-webinar flow), **except** when the message looks like video/digital case studies (`video case`, `case stud`, `digital case`, `test and learn`).

## Digital vs video case studies

- **`getSpecificTopicReply()`** — **`asksDigitalCaseStudies`** answers first: digital case studies = Adult Echo video case studies / Test & Learn (same product).
- **`asksWebinarScheduleQuestion`** — suppresses video-case product replies when the user is clearly asking **when/next** for a **webinar**.

## X-ZONE tier pricing

- **`getXzoneSubscriptionTierFromMessage()`** — parses 1/5/15-day tiers.
- **`getXzoneContextPriceReply()`** — runs **early** (before **`getContextualShortPriceReply`**). In X-ZONE context: tier → single price; “how much for xzone” → full table; bare “how much” → `null` so contextual can clarify.
- **`getSpecificPriceReply()`** — tier + **xzone** in message returns that tier even without “how much” (e.g. “5 day xzone”).

## CME after webinar

- **`getCmeContextReply()`** — after-webinar + how to get CMEs → SDMS roster / ~2 weeks / free account / email troubleshooting. Also: **how many CMEs**, **never got CME email** (narrower), then generic paths.

## Alliance Program

- **`getAllianceProgramReply()`** — runs after **`getRefundTransferPriorityReply()`** when `alliance` / `academic alliance` appears.

---

*Last updated: 2026-04-20.*
