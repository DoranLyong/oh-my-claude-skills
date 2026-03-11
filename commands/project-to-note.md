# Skill: Project-to-Note (Paper-Code Integrated Walkthrough)

> A repeatable methodology for converting a research paper + its code repository
> into an integrated walkthrough that maps every equation to its exact implementation.
> Reference output: `doc/overview.md` (ViT³ project)

---

## 1. What This Skill Produces

A single Markdown document that walks through a research project by placing
**paper equations and code side by side**. The reader sees the math on the left
and the PyTorch (or other framework) implementation on the right, with
inline annotations explaining every non-obvious mapping.

**Target audience**: someone who has the paper open in one tab and the code
open in another, and wants to understand how one becomes the other.

**How it differs from Paper-to-Note (study_note.md)**:

| | Paper-to-Note | Project-to-Note |
|---|---|---|
| **Focus** | Concepts and their relationships | Equation ↔ code mapping |
| **Structure** | 4-facet mind-map (Definition/Properties/Application/Links) | Paper storyline + side-by-side code blocks |
| **Source** | Paper only (code refs are supplementary) | Paper + full codebase (equal weight) |
| **Language** | English | User's preferred language (Korean in reference) |
| **Code** | `file:line` references | Full code snippets with inline annotations |
| **Training results** | Not included | Included (actual experiment logs) |
| **Typical size** | 800–1200 lines | 600–1000 lines |

---

## 2. Document Skeleton

```
# {Paper Title} — Paper-Code Integrated Walkthrough

> **{Paper Title}** ({arXiv ID})
> {one-line description of what this document does}

---

## Table of Contents (with anchor links)

## 0. {Core Concept} — Universal Background
## 1. {Motivation} — Why This Paper Exists
## 2. {Paper Structure} — Section-by-Section Flow Summary
## 3. {Key Evolution} — How Ideas Build on Each Other
## 4. {Core Mechanism} — The Main Technical Idea
## 5. {Design Choices} — Insights/Ablations with Code Pointers
## 6–K. {Equation-Code Matching} — One Section per Major Component
## K+1. {Full Architecture} — End-to-End Model + Code
## K+2. {Training Strategy} — Special Recipes + Code
## K+3. {Loss Analysis} — Mathematical Details + Code
## Appendix: {Config Reference Table}
## Last: {Training Experiment Results} — Actual Run Data
```

---

## 3. The Core Format: Side-by-Side Equation ↔ Code

This is the defining feature. Every technical section uses this layout:

### 3.1 Basic Pattern

````markdown
### {Paper Reference}: {What This Computes}

```
Equation:                              Code ({file}:{lines}):
─────                                  ─────────────────────────
z₁ = K · W₁                           z1 = k @ w1
z₂ = K · W₂                           z2 = k @ w2
σ(z₂) = sigmoid(z₂)                   sig = F.sigmoid(z2)
```
````

Rules:
- Left column: paper notation (Unicode math symbols, no LaTeX)
- Right column: actual code, copy-pasted from the repo
- Alignment: corresponding lines at the same vertical position
- `─────` separator under each column header

### 3.2 Annotated Code Block Pattern

For longer code, use annotated code blocks instead of side-by-side:

````markdown
```python
# {file}:{line_range}

self.qkv = nn.Linear(dim, dim * 3 + head_dim * 3, bias=qkv_bias)
#                         ↑ Q₁,K₁,V₁    ↑ Q₂,K₂,V₂

# ──────────────────────────────────────────────────────────────
# Paper Eq.(5): inner model parameters W₀ (learned end-to-end)
# ──────────────────────────────────────────────────────────────
self.w1 = nn.Parameter(torch.zeros(1, num_heads, head_dim, head_dim))  # W₁
self.w2 = nn.Parameter(torch.zeros(1, num_heads, head_dim, head_dim))  # W₂
```
````

Rules:
- First line: `# {file}:{line_range}` — exact location
- `# ──────` separator lines to group related parameters
- Paper reference as a comment above each group
- `← arrow` or `↑ arrow` annotations for non-obvious mappings
- `# W₁`, `# W₂` at end of lines to link variables to paper notation

### 3.3 Derivation Pattern

For hand-derived gradients or multi-step math:

````markdown
### ∂L/∂W₁ derivation:

```
∂L/∂W₁ = ∂L/∂V̂ · ∂V̂/∂z₁ · ∂z₁/∂W₁

V̂ = z₁ ⊙ a  →  ∂V̂/∂z₁ = a  (elementwise)
z₁ = K·W₁   →  ∂z₁/∂W₁ = Kᵀ

Therefore:
∂L/∂W₁ = Kᵀ · (e ⊙ a)

Equation:                              Code ({file}:{line}):
─────                                  ─────────────────────────
g₁ = Kᵀ · (e ⊙ a)                     g1 = k.transpose(-2,-1) @ (e * a)
```
````

---

## 4. Section Types

### 4.1 Universal Background (§0)

Explains the core concept in broader context, independent of this specific paper.
Written for someone who has never heard the term.

Structure:
- **Origin and definition**: who proposed it, what it means
- **Two-paradigm comparison**: before vs. after (ASCII diagram)
- **Multiple interpretations**: if the concept has different meanings in different fields
- **Intuitive analogy**: a non-technical metaphor (ASCII diagram)
- **Academic connections**: table of related fields and how they connect

```markdown
### 0.1 Origin and Definition
{paragraph + ASCII diagram comparing old vs. new paradigm}

### 0.2 Multiple Interpretations
{ASCII diagram showing (A) interpretation vs. (B) interpretation}
{state which one this paper uses}

### 0.3 Intuitive Analogy
{ASCII diagram with a metaphor}

### 0.4 Academic Connections
| Related Field | Connection |
|---|---|
| ... | ... |
```

### 4.2 Motivation (§1)

Why the paper exists. What problem it solves.

Structure:
- **Limitation of current approach**: specific numbers (O(N²), memory)
- **Why alternatives fall short**: concrete failure mode
- **This paper's answer**: one-sentence thesis

### 4.3 Paper Structure Summary (§2)

A vertical flowchart showing the paper's section-by-section storyline.

```markdown
### 2.1 Paper Storyline

```
§1 Introduction — "Why?"
│
│  {2-3 bullet points summarizing the section}
│
▼
§2 Related Work — "Where from?"
│
│  {2-3 bullet points}
│
▼
§3 Method — "How?" ★ Core
│
│  {detailed breakdown with sub-bullets}
│
▼
§4 Experiments — "Does it work?"
│
│  {key results}
│
▼
§5 Conclusion
```

### 2.2 Three-Line Summary
{numbered list of 3 core contributions}

### 2.3 Comparison with Prior Work
{table: this paper vs. predecessor on 5-6 dimensions}
```

### 4.4 Equation-Code Matching Sections (§6–K)

The heart of the document. One section per major component.

Each section follows this order:
1. **Paper reference**: which section/figure/equation
2. **What it does**: one-sentence plain-language summary
3. **Code: `__init__`**: parameter definitions with paper annotations
4. **Code: `forward`**: computation steps with side-by-side equations
5. **Implementation notes**: non-obvious choices (optimizations, tricks)

```markdown
## N. Equation-Code Matching: {Component Name}

### Paper §X, Fig.Y: {Component} Structure
{1-2 sentence description}
{bullet list of sub-components}

### Code: {ClassName}.__init__
```python
# {file}:{lines}
{annotated code}
```

### Code: {ClassName}.{method} — {what it computes}
```
Equation:                              Code ({file}:{lines}):
─────                                  ─────────────────────────
{side-by-side mapping}
```

> **Implementation note**: {why the code differs from the naive equation}
```

### 4.5 Architecture Section

Maps the full model (not just one block) to code.

Structure:
- **Paper table ↔ code constructor**: model dimensions, depths, heads
- **Block structure**: paper figure ↔ code forward pass
- **Stem/Downsampling**: how input is tokenized
- **Classification head**: how output is produced

Use this pattern for paper-table ↔ code matching:
```
Paper Table N:                         Code ({file}:{lines}):
─────────────                          ────────────────────────────
Stage1: Stem↓4, B(64,2)×1             model = Model(
Stage2: Down↓2, B(128,4)×3                dim=[64, 128, 320, 512],
Stage3: Down↓2, B(320,10)×9               depths=[1, 3, 9, 4],
Stage4: Down↓2, B(512,16)×4               num_heads=[2, 4, 10, 16],
                                       )
```

### 4.6 Training Strategy Section

For papers with special training recipes (e.g., MESA, distillation).

Structure:
- **Paper result**: before vs. after applying the strategy
- **Config ↔ code**: yaml settings mapped to code lines
- **Loss computation ↔ code**: equation vs. implementation
- **Update rule ↔ code**: weight update equation vs. code

### 4.7 Config Reference Table (Appendix)

A single table mapping every important hyperparameter to its code location:

```markdown
| Paper Item | Value | Code Location |
|---|---|---|
| Inner loss | Dot product | `ttt_block.py:72,78` |
| Inner batch size | N (full-batch) | `ttt_block.py:54-88` |
| Outer optimizer | AdamW | `optimizer.py:29-30` |
| Weight decay | 0.05 | `config.py:67` |
| MESA ratio | 1.0 | `h_vittt_t_mesa.yaml:6` |
| ...  | ... | ... |
```

This table is exhaustive — include every config value that appears in the paper.

### 4.8 Training Experiment Results (Last Section)

If the user has actually trained the model, document the results.

Structure:
```markdown
### N.1 Training Environment
```
Model:       {name}
Dataset:     {name} ({train size} / {val size})
GPU:         {count}× {type}
Batch Size:  {N}
Epochs:      {N}
Start:       {datetime}
End:         {datetime}
Total Time:  {duration}
```

### N.2 Training Speed
{per-epoch timing, noting any phase changes}

### N.3 Accuracy Curve
{ASCII chart showing accuracy over epochs}
{table of accuracy at key epochs}

### N.4 Analysis
- Final performance vs. paper's reported numbers
- Difference analysis (batch size, GPU count, LR scaling)
- Strategy effectiveness (MESA, augmentation, etc.)
- Training stability observations

### N.5 Checkpoint Files
```
{directory tree of saved files with sizes and descriptions}
```
```

---

## 5. Visual Elements

### 5.1 ASCII Paradigm Diagrams
Use box-drawing characters for comparing paradigms:
```
┌─── Old Paradigm ──────────────────┐
│  Training:  learn W                │
│  Inference: fix W, predict         │
│  Problem:   one-size-fits-all      │
└────────────────────────────────────┘

┌─── New Paradigm ──────────────────┐
│  Training:  learn W                │
│  Inference: adapt W per input!     │
│  Benefit:   input-specific         │
└────────────────────────────────────┘
```

### 5.2 Paper Flow Diagrams
Use `│`, `▼`, `├─`, `└─` for vertical section flows (see §4.3).

### 5.3 Nested Loop Diagrams
For bi-level optimization (outer loop / inner loop):
```
┌─ Outer Loop ────────────────────────┐
│  for epoch in range(300):           │
│    for batch in dataloader:         │
│      output = model(input)          │
│      ┌─ Inner Loop ──────────────┐  │
│      │  {inner computation}      │  │
│      └───────────────────────────┘  │
│      loss.backward()                │
└─────────────────────────────────────┘
```

### 5.4 Accuracy Curve (ASCII)
```
Acc@1
  ↑
83│                    ★ 83.13% (best)
82│
81│              ···
80│         ····
79│      ···
  │    ··
75│  ·
  │
  └──────────────────────→ epoch
    1   50  100  150  200  250  300
```

---

## 6. Language and Style

### 6.1 General Rules
- **Language**: user's preferred language (Korean in the reference output).
  Technical terms (Softmax, SiLU, DWConv) stay in English.
- **Code blocks**: always in English (variable names, comments match the codebase).
- **Annotations inside code**: can be in the user's language.
- **Section titles**: bilingual is fine — `## 6. 수식-코드 1:1 매칭: TTT Block`

### 6.2 Annotation Style for Code
- Use `←` arrows for brief inline notes: `self.scale = 9 ** -0.5  ← 1/√d`
- Use `↑` arrows for notes on the line above: `#  ↑ Q₁,K₁,V₁    ↑ Q₂,K₂,V₂`
- Use `# ──────` separator lines to group related code
- Use `# Paper §X:` comments to link code sections to paper sections
- Bold the paper equation reference at the start: `# 논문 Eq.(5):` or `# Paper Eq.(5):`

### 6.3 Explanation Blocks
Use `> **{label}**:` for implementation notes:
```markdown
> **Why hand-derived gradient?**
> The inner loss has shape [B, num_heads] — it is vector-valued, not scalar.
> `torch.autograd.backward` only supports scalar losses, so the code derives
> closed-form gradient expressions manually.
```

---

## 7. Process: How to Build the Note

### Phase 1: Map the Codebase
1. Identify the core source files (typically 3–5 files that implement the paper's method).
2. Read each file. Note the class names, method names, and line ranges.
3. Build a rough file-to-paper mapping:
   ```
   ttt_block.py    → Paper §3-4 (TTT mechanism, inner training)
   h_vittt.py      → Paper §5 (architecture, Table 12)
   main_ema.py     → Paper §5.1 (MESA training)
   config.py       → Paper Table 11 (hyperparameters)
   ```

### Phase 2: Map Equations to Code
1. For each equation in the paper, find the exact code line(s) that implement it.
2. Note where the code diverges from the equation (optimizations, shape tricks).
3. Create the side-by-side blocks.

### Phase 3: Write Context Sections (§0–§5)
1. §0: Explain the core concept for a newcomer.
2. §1: Motivate the paper — what problem, why existing solutions fail.
3. §2: Summarize the paper's section flow as a vertical diagram.
4. §3: Show the conceptual evolution (e.g., Attention → Linear Attention → TTT).
5. §4: Explain the core mechanism (e.g., inner loop vs. outer loop).
6. §5: List the paper's key design insights with code pointers.

### Phase 4: Write Equation-Code Matching Sections (§6–K)
1. One section per major component (init, forward, backward, inference).
2. Follow the code's execution order, not the paper's section order.
3. For each section: paper reference → code annotation → side-by-side → implementation notes.

### Phase 5: Write Architecture + Training Sections
1. Full architecture: paper table ↔ code constructor.
2. Training strategy: config ↔ code, loss ↔ code, update ↔ code.
3. Config reference table: exhaustive hyperparameter mapping.

### Phase 6: Add Training Results (if available)
1. Parse training logs for accuracy, timing, and loss data.
2. Create ASCII accuracy chart.
3. Compare with paper's reported numbers.
4. Analyze discrepancies (batch size, GPU count, LR scaling).

### Phase 7: Review
1. Verify every code line reference is correct (line numbers change!).
2. Verify every paper equation number matches.
3. Run the code mentally through the side-by-side blocks — does the mapping make sense?
4. Check that the table of contents links work.

---

## 8. Sources Required

| Source | What It Provides | Priority |
|---|---|---|
| Paper PDF / markdown | Equations, tables, insights, architecture definitions | Required |
| Core implementation files | Forward pass, backward pass, model class | Required |
| Config files (yaml/json) | Hyperparameter values | Required |
| Training script | Optimizer, LR schedule, augmentation | Required |
| Training logs | Accuracy curves, timing, loss values | Optional (for §Results) |
| Model checkpoint | Validation accuracy verification | Optional |

---

## 9. Adapting to a New Project

### Step 1: Identify the Core Loop
Every ML project has a core computational loop. Find it.
- For a Transformer paper: the attention mechanism
- For a GAN: the generator/discriminator forward pass
- For a diffusion model: the denoising step
- For a loss paper: the loss computation

This becomes the central equation-code matching section.

### Step 2: Count the Components
List every distinct component that has both a paper equation and a code implementation.
Each gets its own equation-code matching section.

Typical count: 3–6 components for a methods paper.

### Step 3: Decide the Execution Order
The equation-code sections should follow the **code's execution order**
(init → forward → backward → inference), not the paper's presentation order.
This makes the document usable as a "reading the code" guide.

### Step 4: Decide the Context Depth
- If the concept is well-known (e.g., standard attention): brief §0, focus on code.
- If the concept is novel (e.g., TTT): detailed §0 with analogies and connections.
- If the paper has many ablations: detailed §5 (insights) with code pointers.
- If the paper has a special training recipe: full training strategy section.

### Step 5: Handle Missing Elements
- **No training logs**: skip the Results section entirely.
- **No config files**: extract hyperparameters from the paper and note them.
- **Multiple implementations**: pick the primary one, note alternatives.
- **Framework other than PyTorch**: adapt the annotation style but keep the side-by-side format.

---

## 10. Checklist

Before considering the note complete, verify:

- [ ] Table of contents with working anchor links
- [ ] §0 explains the core concept for a newcomer
- [ ] Paper storyline diagram shows §1→§N flow
- [ ] Every paper equation appears in a side-by-side block with correct code
- [ ] Code line numbers are accurate (verified against the actual files)
- [ ] Annotated code blocks have `# {file}:{lines}` headers
- [ ] Paper ↔ code variable name mapping is explicit (W₁ ↔ self.w1, etc.)
- [ ] Implementation notes explain non-obvious divergences from equations
- [ ] Architecture section maps paper table to code constructor
- [ ] Config reference table is exhaustive
- [ ] Training results (if available) include accuracy chart and paper comparison
- [ ] Language is consistent throughout (no unintended mixing)
- [ ] Every `> **Note**:` block answers a "why" question
