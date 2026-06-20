# Literature map: recursive reasoning (HRM → TRM → critiques & variants)

*A guided reading list for the recursive-reasoning thread. Companion to the
[README](../README.md); see also the [mechanism explainer](how-trm-works.md).*

> Re-verified **2026-06-20**. All metrics are the cited authors', not mine.
> Sources at the [bottom](#sources).

---

## The lineage at a glance

```
HRM (2506.21734)                Hierarchical Reasoning Model — two coupled
   │                            modules + fixed-point recurrence
   ▼
TRM (2510.04871)                Tiny Recursive Models — one tiny shared net,
   │                            no hierarchy, no fixed-point math; ARC Prize
   │                            2025 Paper Award
   ├──► critique  (2512.11847)  "...Inductive Biases, Identity Conditioning,
   │                            and Test-Time Compute" — what the result is
   │                            really made of
   └──► variant   (2602.12078)  Mamba-2 attention hybrid
```

## 1. HRM — Hierarchical Reasoning Model

**[arXiv:2506.21734](https://arxiv.org/abs/2506.21734)**

The starting point for this thread. HRM proposes solving hard puzzles with a
recurrent system of **two coupled modules** operating at different timescales,
trained with a fixed-point / equilibrium-style objective rather than by unrolling
many steps. It showed that small recurrent models could do surprisingly well on
ARC-style puzzles, Sudoku and mazes — establishing the "iterate, don't scale" idea
that TRM later sharpened.

**Why read it:** it frames the central hypothesis (depth via recurrence) and the
machinery (two modules, fixed-point training) that TRM deliberately strips away.

## 2. TRM — Tiny Recursive Models

**[arXiv:2510.04871](https://arxiv.org/abs/2510.04871)** · Alexia Jolicoeur-Martineau,
Samsung SAIL Montréal · **ARC Prize 2025 Paper Award (1st)**

TRM's thesis is *less is more*: a **single shared ~7M-parameter, 2-layer network**
that recursively refines a latent state and a candidate answer beats both HRM and
much larger LLMs on several public reasoning evals. It removes HRM's two-module
hierarchy and fixed-point math in favour of a simpler, directly-supervised recursive
update.

**Reported results (the paper's):** ~**45%** ARC-AGI-1, ~**8%** ARC-AGI-2,
**87.4%** Sudoku-Extreme, **85.3%** Maze-Hard, reportedly ahead of DeepSeek-R1,
o3-mini-high and Gemini 2.5 Pro on those evals with **<0.01%** of their parameters.

**Why read it:** it's the headline result of the thread, and the mechanism is clean
enough to understand fully — see [how-trm-works.md](how-trm-works.md).

**Code:** [`SamsungSAILMontreal/TinyRecursiveModels`](https://github.com/SamsungSAILMontreal/TinyRecursiveModels)
(MIT) — what the round-2 reproductions will build on.

## 3. The critical follow-up

**[arXiv:2512.11847](https://arxiv.org/abs/2512.11847)** — *"Tiny Recursive Models on
ARC-AGI-1: Inductive Biases, Identity Conditioning, and Test-Time Compute"*

The essential counter-reading. It argues TRM's ARC-AGI-1 number comes from an
**interaction of efficiency, task-specific conditioning, and heavy test-time
compute** rather than deep internal reasoning, with concrete evidence:

- **Test-time compute:** ~1000-sample voting adds roughly **+11 percentage points**
  over a single pass.
- **Identity conditioning:** blanking or randomising the **puzzle ID** drops accuracy
  to **zero** — the model depends on task identifiers, not grids alone.
- **Shallow recursion:** most accuracy appears at the **first** recursion step and
  plateaus quickly.
- **Real efficiency:** substantially lower memory and higher throughput than a
  fine-tuned Llama-3-8B baseline.

**Why read it:** it's the difference between repeating a press-release claim and
understanding the result. Both the efficiency *and* the caveats are real.

## 4. Variants & adjacent work

- **Mamba-2 attention hybrid** — **[arXiv:2602.12078](https://arxiv.org/abs/2602.12078)**,
  *"Tiny Recursive Reasoning with Mamba-2 Attention Hybrid"*: explores swapping the
  backbone, probing how much the recursive *recipe* (vs. the specific architecture)
  drives the result.
- **ARC Prize 2025: Technical Report** — **[arXiv:2601.10904](https://arxiv.org/abs/2601.10904)**:
  context for the Paper Award and where TRM sits among 2025 submissions.

## Reading order I'd suggest

1. **TRM** (2510.04871) — get the claim and the mechanism.
2. **how-trm-works.md** — make sure the loop is clear.
3. **The critique** (2512.11847) — see what the number is actually made of.
4. **HRM** (2506.21734) — backfill the lineage TRM simplified.
5. **Variants** (2602.12078) — see how robust the recipe is to architecture changes.

---

## Sources

- HRM — <https://arxiv.org/abs/2506.21734>
- TRM (*Less is More*) — <https://arxiv.org/abs/2510.04871>
- TRM critique — <https://arxiv.org/abs/2512.11847>
- Mamba-2 hybrid variant — <https://arxiv.org/abs/2602.12078>
- ARC Prize 2025: Technical Report — <https://arxiv.org/abs/2601.10904>
- Official TRM code (MIT) — <https://github.com/SamsungSAILMontreal/TinyRecursiveModels>

*Prose © 2026 Antonio Rodriguez-Moral, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).*
