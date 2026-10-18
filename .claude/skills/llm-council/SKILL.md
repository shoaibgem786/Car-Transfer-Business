---
name: llm-council
description: Answer a hard question (business decision, strategy, evaluation) by convening a "council" of independent subagents, having them anonymously peer-review and rank each other, then synthesizing a chairman's final answer. Adapted from karpathy/llm-council. Use when the user asks for a council, multiple opinions, or invokes /llm-council.
---

# LLM Council

Adapted from https://github.com/karpathy/llm-council. The original is a web app that
fans a query out to several vendors' models via OpenRouter. This skill runs the same
three-stage protocol with Claude subagents. Since every member is the same model
family, diversity comes from giving each member a distinct expert lens, and from
keeping them blind to each other in Stage 1.

## Stage 0: Frame the question
- Restate the user's question in full, with all context they gave (links, notes,
  facts already researched). Every member gets the identical question text.
- Pick 4 council members, each a distinct, relevant expert lens. For a business
  decision, a good default is:
  1. Strategy / market-sizing analyst
  2. Operator who has run this kind of business in the target market
  3. Legal, regulatory and insurance specialist for the target market
  4. Skeptical investor / devil's advocate
- Tell each member they may use web search, must cite sources, and must flag guesses
  as guesses.

## Stage 1: First opinions (independent, parallel)
Launch all members at once (one Agent call each, in a single message). Each sees only
the question and its own lens, never the other members. Collect the full responses.

## Stage 2: Anonymous peer review (parallel)
Shuffle the Stage 1 responses and label them "Response A", "Response B", ... Hide
which lens wrote which. Give every member all the labelled responses and this prompt
(from the original repo):

> You are evaluating different responses to the following question: {question}
> Here are the responses (anonymized): {responses}
> 1. First, evaluate each response individually: what it does well and what it does
>    poorly. Call out factual errors and unsupported claims.
> 2. Then, at the very end, give a final ranking, formatted exactly:
>    `FINAL RANKING:` followed by a numbered list, best first, one label per line
>    (e.g. `1. Response C`), with no other text.

Parse each ranking and compute the aggregate (average rank position per response).

## Stage 3: Chairman synthesis
Act as Chairman yourself (or use one more subagent). Using the question, all Stage 1
responses, all Stage 2 reviews and the aggregate ranking, write one final answer
that represents the council's collective view. Weight higher-ranked responses more,
keep points the reviewers agreed on, resolve or plainly flag disagreements, and drop
claims the reviewers showed to be wrong.

## Output to the user
1. The Chairman's final answer (the main deliverable).
2. A short "Council record": the lenses used, the de-anonymized aggregate ranking,
   and the main points of disagreement.
3. A note that all members were Claude subagents, not multiple vendors' models.
