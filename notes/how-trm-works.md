# How TRM works

*A mechanism walk-through of Tiny Recursive Models. Companion to the
[README](../README.md); see also the [literature map](literature-map.md).*

> Re-verified **2026-06-20**. Based on *Less is More: Recursive Reasoning with Tiny
> Networks* ([arXiv:2510.04871](https://arxiv.org/abs/2510.04871)) and its official
> MIT code. All reported metrics are the authors', not mine. Sources at the
> [bottom](#sources).

---

## The one-sentence version

TRM is a **single, tiny (~7M-param, 2-layer) network** that, instead of being deep
or large, is **run in a loop**: it repeatedly refines a hidden "scratchpad" `z` and a
candidate answer `y` for the same problem, so that *test-time iteration* — not
parameters or layers — supplies the effective reasoning depth.

## The pieces

- **`x` — the question.** The task input (e.g. an ARC grid, a Sudoku, a maze),
  embedded once.
- **`y` — the current answer.** An embedding of the candidate output. It starts from
  an initial guess and is improved over the loop.
- **`z` — the latent scratchpad.** A hidden state that carries the model's
  "working notes" about how to fix `y`.
- **`f` — the tiny shared network.** The *same* 2-layer network is reused at every
  step (weight-shared across iterations); there is no separate per-step model and no
  two-module hierarchy (that was HRM).

## The loop

At each recursion step, the shared network looks at the question, the current answer
and the current scratchpad, and updates both:

```
initialise y (a first guess), z (empty scratchpad)
repeat for a fixed budget of supervision steps:
    z ← f(x, y, z)      # refine the scratchpad given question + current answer
    y ← f(x, y, z)      # revise the answer given the refined scratchpad
return y                 # the final, refined answer
```

Conceptually it's **draft → critique → revise**, repeated: propose an answer, update
the internal notes about what's wrong with it, then rewrite the answer. The number of
steps is a fixed budget (on the order of ~16 in the paper's setup), and supervision
is applied so the model learns to *make progress* across steps rather than only at the
end.

## How it's trained

- **From scratch, per task family** — not few-shot prompting of a pretrained LLM.
- **Heavy data augmentation** — the rule-preserving transforms standard for these
  puzzles (rotations, reflections, colour permutations for ARC) multiply the
  effective training signal.
- **Deep supervision over the recursion** — the model is trained to improve the
  answer across steps, which is what lets a 2-layer network behave as if it had much
  greater depth.

## What TRM simplified vs HRM

TRM is, in large part, *HRM with the complicated bits removed*:

| | HRM | TRM |
|---|---|---|
| Modules | Two coupled modules at different timescales | One shared tiny network |
| Training objective | Fixed-point / equilibrium recurrence | Direct deep supervision over unrolled steps |
| Net effect | Strong but intricate | Simpler, smaller, and (reportedly) better |

The lesson the paper draws: much of HRM's machinery wasn't load-bearing; a single
small network, supervised across a recursive loop, does more with less.

## Reading the results honestly

The mechanism is elegant, but the *numbers* deserve the caveats in the
[README's critical reading](../README.md#3-a-critical-reading) and the
[literature map](literature-map.md): a large share of the ARC-AGI-1 score is
attributable to **aggressive test-time sampling** (~1000-sample voting ≈ +11pp) and
**puzzle-ID conditioning** (remove it and accuracy collapses to zero), and the
recursion is **shallow in practice** (most gains by the first step). The efficiency is
genuine; "deep iterative reasoning" overstates what the loop is doing. Hold both
ideas at once.

---

## Sources

- TRM — *Less is More: Recursive Reasoning with Tiny Networks* — <https://arxiv.org/abs/2510.04871>
- HRM — *Hierarchical Reasoning Model* — <https://arxiv.org/abs/2506.21734>
- TRM critique — <https://arxiv.org/abs/2512.11847>
- Official TRM code (MIT) — <https://github.com/SamsungSAILMontreal/TinyRecursiveModels>

*Prose © 2026 Antonio Rodriguez-Moral, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).*
