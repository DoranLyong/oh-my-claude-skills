# Skill: Paper-to-Note (Concept Mind-Map)

> A repeatable methodology for converting a research paper into a structured study note.
> Reference output: `doc/study_note.md` (ViT³ paper — 21 sections, ~1100 lines)

---

## 1. What This Skill Produces

A single Markdown document that organizes **every key concept** in a paper as a mind-map.
Each concept is broken into four standard facets, cross-referenced with all other concepts,
and supplemented with derivations, tables, code pointers, and visual diagrams.

**Target audience**: the author's future self (or a collaborator) who wants to
understand the paper deeply, not just skim the abstract.

**Typical output**: 15–25 `##` sections, 800–1200 lines of Markdown.
The exact count depends on how many distinct concepts the paper introduces.

---

## 2. Document Skeleton

```
# {Paper Title} Study Note — A Concept Mind-Map

> Paper: {title} ({arXiv ID})
> Authors: {names}
> Affiliations: {institutions}

This note organizes every key concept in the paper as a mind-map.
Each concept is broken down into four facets:

- **Definition** — what it is, stated plainly
- **Properties** — its mathematical or behavioral characteristics
- **Application** — how the paper (or the field) uses it
- **Links** — connections to other concepts in this map

---

## 0. The Big Picture              ← ASCII art: paper's core story arc

## 1–K. {Concept Sections}        ← 4-facet breakdown for each concept

## K+1. {Prior Art Comparison}     ← what existed, limitations, what this paper changes
## K+2. {Quick Reference Card}     ← insights/contributions summary table
## K+3. {Open Questions}           ← future work identified by the paper
## K+4. {Concept Dependency Graph} ← ASCII art: how all concepts connect
## K+5. {Key Equations}            ← summary table of all equations
## K+6. {Reference Map}            ← citations grouped by topic
```

---

## 3. The 4-Facet Framework

Every concept section **must** include these four subsections:

### 3.1 Definition
- Plain-language explanation of what the concept is.
- Include the core equation with `$$...$$` LaTeX and `\tag{Eq.N}` numbering.
- If the concept has multiple variants, list them.

### 3.2 Properties
- Mathematical characteristics (complexity, capacity, constraints).
- Behavioral properties (when it works, when it fails).
- **Insights from the paper**: quote the paper's numbered insights verbatim,
  then explain the evidence (table data, figures).

### 3.3 Application
- How the paper uses this concept in its model.
- **Code references**: `file.py:line` format pointing to the implementation.
- Experimental results (reproduce key table rows).

### 3.4 Links
- Bullet list of connections to other concepts in the note.
- Format: `→ **{Concept Name}**: {relationship description}`
- Every link should be bidirectional — if A links to B, B should link back to A.

---

## 4. Special Subsections

Beyond the 4 facets, add these when applicable:

### 4.1 Derivation (inline)
- Use when one equation is derived from another within the same concept section.
- Format: `### Derivation: Eq.X → Eq.Y`
- Break into numbered steps: **Step 1**, **Step 2**, ...
- Each step: state what you do, show the result equation.
- Add explanatory blocks:
  - **Why {X} blocks this**: explain the mathematical obstacle.
  - **Complexity consequence**: before vs. after.
  - **Trade-off**: what is gained and lost.

Example from ViT³ note — inside §2 Linear Attention:
```markdown
### Derivation: Eq.1 → Eq.3

**Step 1.** Write Eq.1 for the i-th output token: ...
**Step 2.** Replace the Softmax kernel with a linear kernel: ...
**Step 3.** Factor Q_i out of the summation: → Eq.3

**Why Softmax blocks this**: Q_i is trapped inside exp(·)...
**Complexity consequence**: O(N²d) → O(Nd²)
**Trade-off**: speed gained, expressive power lost
```

### 4.2 Detailed Derivations (appendix-style)
- For lengthy proofs (e.g., loss function derivatives), create a sub-section
  under the relevant concept.
- Show each case separately with equation numbering: **(1)**, **(2)**, ...
- End with a **Summary pattern** table.

Example from ViT³ note — under §5.1 Inner Loss:
```markdown
#### Detailed Derivations (Appendix §8, Eq.8–17)

**(1) Dot Product Loss (Eq.8–9)**
  formula... derivative... conclusion.

**(2) MSE / L2 Loss (Eq.10–11)**
  formula... derivative... conclusion.

...

**Summary pattern**:
| Loss | ∂²L/∂V∂V̂ | Behavior | Top-1 |
|---|---|---|---|
| Dot Product | constant | ✓ | 78.9 |
| MAE | 0 (a.e.) | ✗ | 76.5 |
```

---

## 5. Section Types and Ordering

### 5.1 Section Ordering Strategy

Follow the paper's **logical dependency chain**, not the section numbers:

```
 1. Big Picture (§0)            — ASCII art showing the paper's narrative arc
 2. Foundation concepts         — what the reader must know first
 3. Core paradigm               — the paper's main contribution
 4. Inner/Outer structure       — if the paper has nested loops or levels
 5. Design choices / Insights   — experimental findings that shaped the design
 6. Architecture details        — the final model, layer by layer
 7. Implementation details      — hand-derived backward, custom CUDA kernels, etc.
 8. Architecture variants       — model family (Tiny/Small/Base, flat/hierarchical)
 9. Downstream tasks            — detection, segmentation, generation, etc.
10. Training strategy           — special training recipes (MESA, distillation, etc.)
11. Efficiency analysis         — speed/memory comparisons
12. Prior Art comparison        — what existed before, limitations, what changed
13. Quick Reference Card        — insights summary table
14. Open Questions              — future work
15. Concept Dependency Graph    — ASCII art of all connections
16. Key Equations               — summary table
17. Reference Map               — citations by topic
```

### 5.2 Grouping Rule
- If a concept has sub-concepts, make them sub-sections under one heading.
- Each sub-section still uses the 4-facet framework where applicable.

```
## 5. Inner Training Configuration
### 5.1 Inner Loss Function        ← 4-facet + derivation appendix
### 5.2 Inner Batch Size and Epochs ← 4-facet
### 5.3 Inner Learning Rate         ← 4-facet

## 6. Inner Model Design
### 6.1 Scaling Width               ← Properties + Links
### 6.2 Scaling Depth               ← Properties + Links
### 6.3 Convolution as Inner Model  ← 4-facet
```

### 5.3 Section Types Beyond 4-Facet Concepts

Some sections follow different formats. Recognize and apply these:

**a) Architecture Flow Section** (e.g., "The TTT Block — Putting It All Together")
- Definition: what the block does, inputs/outputs
- ASCII architecture diagram showing data flow with branches
- Properties: shape-preserving, complexity, parallelizability
- Application: code reference to the full class

**b) Downstream Task Sections** (e.g., "Object Detection", "Semantic Segmentation")
- **Setup**: framework (Mask R-CNN, UPerNet) + backbone + schedule
- **Key result**: one focused table + one-sentence interpretation
- Keep these concise — they demonstrate generality, not dive deep

**c) Training Strategy Section** (e.g., "MESA")
- Definition + How it works (ASCII diagram showing student/teacher flow)
- Properties: overhead, improvement magnitude
- Application: config values + code locations

**d) Prior Art Comparison Section** (see §6 below for full specification)

**e) Quick Reference Card**
- Single summary table: `| # | Insight | Design Choice | Evidence |`
- One row per key finding

**f) Efficiency Analysis**
- Definition: what is being compared
- Key data points: speedup, memory reduction at specific resolutions
- Keep brief — this is a reference section

---

## 6. The Prior Art Comparison Section

This section is critical and was not obvious at first. Every paper positions itself
against existing work, and the note should capture this positioning clearly.

### Structure (4 subsections)

```markdown
## K+1. What Existed Before and What This Paper Changes

### K+1.1 Prior Approaches and Their Limitations
- For each major prior approach (3–5 approaches):
  - What it is (1 sentence)
  - What it does well (1 sentence)
  - Where it falls short (2–3 sentences with specific evidence)
- Write in narrative paragraphs, not bullet lists — this section tells a story.

### K+1.2 What This Paper Contributes
- Number each contribution explicitly (Contribution 1, 2, 3).
- For each: what gap it fills, what it found/built, why it matters.
- Include tables where appropriate (e.g., "What they tested | What they found | Why it matters").

### K+1.3 Side-by-Side: Prior Art vs. This Paper
- One table comparing all approaches on key dimensions:
  | Dimension | Approach A | Approach B | This Paper |
- One table comparing this paper with its closest predecessor:
  | Dimension | Predecessor | This Paper |

### K+1.4 The Core Shift in Thinking
- 2–3 paragraphs explaining the fundamental conceptual change.
- Not a list of improvements — a narrative about what changed and why it matters.
```

### Writing Style for This Section
- **No LLM filler words**: avoid "notably", "importantly", "it is worth noting",
  "furthermore", "comprehensive", "robust", "leverages", "facilitates".
- **Lead with specifics**: "Linear Attention compresses context into a d×d matrix"
  not "Linear Attention offers an efficient alternative".
- **Name the limitation precisely**: "the d×d linear state is too small"
  not "it has limited capacity".

---

## 7. Visual Elements

### 7.1 Big Picture (§0)
- ASCII art showing the paper's high-level story.
- Use box-drawing characters: `┌ ┐ └ ┘ │ ─ ├ ┤ ┬ ┴ ┼ ▼ ▲ ◄ ►`
- Show: problem → approaches → this paper's solution → applications.

```
Example (from ViT³ note):
                         Vision Transformers
                               |
                    "O(N²) is too expensive"
                               |
               ┌───────────────┼───────────────┐
               ▼               ▼               ▼
        Linear Attention    Mamba (SSM)    Test-Time Training
               │                               │
               └───────────────┼───────────────┘
                               ▼
                    6 Practical Insights → ViT³ Model
```

### 7.2 Concept Dependency Graph
- ASCII art showing how **all** concepts in the note connect.
- Use arrows `──→` to show dependency direction.
- Group related concepts vertically.
- The graph should be readable as a "table of contents with relationships."

### 7.3 Architecture Diagrams
- For model architectures, use vertical flow diagrams inside the Definition subsection:
```
Input x
  │
  ├── Branch 1 (SwiGLU)
  │     │
  │    Inner train → Inner infer → x1
  │
  ├── Branch 2 (DWConv)
  │     │
  │    Inner train → Inner infer → x2
  │
  └──→ Concat [x1, x2] → Proj → Output
```

### 7.4 Training Flow Diagrams
- For training strategies, show the data flow between components:
```
Student (model)            Teacher (EMA model)
┌──────────────┐          ┌──────────────────┐
│  θ_model     │──update──▶ θ_ema            │
└──────┬───────┘          └────────┬──────────┘
       │                           │
   logits_s                    logits_t
       │                           │
       ▼                           ▼
L = (1-r)·CE(logits_s, label) + r·KL(logits_s, logits_t)
```

---

## 8. Tables

### 8.1 Experimental Results
- Reproduce key tables from the paper.
- Use Markdown table syntax with alignment.
- **Bold** the paper's chosen design / best result.
- Include column headers: Model | Params | FLOPs | Metric.
- Include ALL model scales reported in the paper (Tiny, Small, Base),
  not just the smallest — the scaling story matters.

### 8.2 Comparison Tables (Design Choices)
- Format: Choice | Config | Result.
- Show the trade-off clearly.
- Include all rows from the paper — missing rows lose information.

### 8.3 Summary Tables
- For insights or equation collections, create compact reference tables.
- Example: `| # | Insight | Design Choice | Evidence |`

### 8.4 Side-by-Side Comparison Tables
- For Prior Art section: Dimension | Method A | Method B | This Paper
- Keep dimensions to 5–8 rows for readability.

---

## 9. Equations

- Use `$$...\tag{Eq.N}$$` for display equations with paper's numbering.
- Use `$...$` for inline math.
- When the paper's equation numbering is referenced elsewhere, keep it consistent.
- For derived quantities not in the paper, use descriptive names instead of numbers.
- For appendix equations, use the paper's appendix numbering (e.g., Eq.8–17).

---

## 10. Code References

- Format: `file.py:line` or `file.py:start-end`.
- Place in the **Application** subsection.
- Map paper concepts to exact code locations:
  ```
  - In code: `ttt_block.py:40-41` — self.w1 and self.w2 are the two weight matrices.
  - Inner training: `ttt_block.py:54-88` — hand-derived forward/backward.
  ```
- When a concept spans multiple code files, list all locations.
- For config values, reference both the yaml file and the code that reads it:
  ```
  - MESA ratio: `h_vittt_t_mesa.yaml:6` — `MESA_RATIO: 1.0`
  - EMA model creation: `main_ema.py:63` — `ModelEma(model, decay=0.9996)`
  ```

---

## 11. Cross-References (Links)

Every concept's **Links** subsection uses this format:

```markdown
### Links
- → **{Target Concept}**: {how this concept relates to the target}
- → **{Another Concept}** [{citation}]: {relationship}
```

Rules:
1. Every link must describe the **relationship**, not just name the target.
2. Use verbs: "is obtained by", "generalizes", "is a special case of", "replaces".
3. If concept A links to B, concept B should link back to A.
4. Include paper citation numbers `[N]` when referencing external work.

---

## 12. Language and Style

### 12.1 General Rules
- **Language**: English (default). Consistent throughout — do not mix languages.
- **Tone**: Technical but accessible. Define jargon on first use.
- **Conciseness**: Lead with the key point, then elaborate. No filler.
- **Emphasis**: Use `**bold**` for key terms, `*italic*` for nuance/emphasis.
- **Section separators**: Use `---` between every `##` section.

### 12.2 Words to Avoid (LLM-style writing)
These words signal machine-generated text. Replace them with specific, concrete language:

| Avoid | Use Instead |
|---|---|
| notably, importantly, interestingly | (just state the fact) |
| comprehensive, robust, holistic | (describe what it actually covers) |
| leverages, facilitates, utilizes | uses, applies, enables |
| it is worth noting that | (delete — start with the content) |
| furthermore, moreover, additionally | (start new sentence or use "also") |
| in this context | (delete or specify the context) |
| demonstrates, showcases | shows, achieves |
| a novel approach | (describe what makes it new) |

### 12.3 How to Write the Prior Art Section
The Prior Art section (§6 above) deserves special attention because it is
the section most prone to vague, filler-heavy writing.

Good:
> Linear Attention compresses the entire context into a single d×d matrix W = K^T V.
> That is too small a bottle for the information to pass through.

Bad:
> Linear Attention notably offers a more efficient alternative by leveraging
> a comprehensive compression strategy, though it demonstrates limitations
> in terms of capacity.

The difference: the good version names the specific object (d×d matrix),
the specific operation (compresses), and the specific problem (too small).
The bad version hides behind abstract nouns and filler adjectives.

---

## 13. Process: How to Build the Note

### Phase 1: Read and Inventory (paper → concept list)
1. Read the paper end-to-end. Do not write anything yet.
2. List every distinct concept. A "concept" = something that has its own definition
   and properties. Typical count: 10–20 for a methods paper.
3. Identify the dependency chain: which concepts build on which.
4. **Inventory all artifacts**: create a checklist of every equation, table, figure,
   remark/insight, and algorithm in the paper.

Example inventory:
```
Equations: Eq.1–7 (main), Eq.8–17 (appendix)
Tables: Tab.1–13
Figures: Fig.1–4
Insights: 1–6
Remarks: 1–6
```

### Phase 2: Build the Skeleton
1. Create §0 Big Picture with ASCII art.
2. Order concepts by dependency (foundations first).
3. Create empty sections with the 4-facet headers.
4. Decide grouping: which concepts become sub-sections of a parent.
5. Add placeholder sections for Prior Art, Quick Reference, Dependency Graph,
   Key Equations, and Reference Map.

### Phase 3: Fill Each Section (iterative)
1. Write **Definition** first — ground the concept.
2. Write **Properties** — what makes it tick mathematically.
3. Write **Application** — where and how it appears in practice.
4. Write **Links** — connect to everything else.
5. Add **Derivation** subsections where equations connect.
6. For each section, check off which artifacts (equations, tables, remarks) it covers.

### Phase 4: Add Meta-Sections
1. Write the **Prior Art Comparison** (§6 structure above).
2. Create the **Concept Dependency Graph** (ASCII art).
3. Create the **Key Equations** table.
4. Create the **Reference Map** (citations grouped by topic).
5. Create the **Quick Reference Card** (insights summary table).
6. Add **Open Questions** from the paper's conclusion.

### Phase 5: Verify Coverage
This is the most important phase. Without it, gaps accumulate silently.

1. **Equation check**: walk through the inventory. Every equation should appear
   in at least one section with correct numbering.
2. **Table check**: every table from the paper should be reproduced or summarized.
   Include all model scales (Tiny/Small/Base), not just one.
3. **Remark/Insight check**: every numbered insight or remark should appear,
   either as a quoted block or integrated into Properties.
4. **Figure check**: key figures should be described or converted to ASCII art.
   Note: bar charts and line plots can be described in words; architecture diagrams
   should be converted to ASCII.
5. **Cross-reference check**: for every link A→B, verify B→A exists.
6. **Language consistency**: scan for mixed languages, LLM filler words,
   inconsistent terminology.

### Phase 6: Iterate
Real note-building is not linear. Expect to:
- Discover missing concepts while writing Links (a concept is referenced but has no section).
- Reorganize sections when the dependency chain becomes clearer.
- Expand a Properties subsection when the derivation reveals more than expected.
- Merge or split sections as the right granularity becomes apparent.

The note is done when the Phase 5 checklist passes cleanly.

---

## 14. Coverage Verification Method

When verifying that a study note covers the full paper, use this systematic approach:

### Step 1: Build a Source Inventory
From the paper (PDF or markdown), list:
- All equations with their numbers
- All tables with their numbers and what they show
- All numbered insights, remarks, or theorems
- All figures and what they illustrate

### Step 2: Cross-reference Against the Note
For each item in the inventory, find where it appears in the study note.
Mark each as:
- **Complete**: fully reproduced with all rows/columns
- **Partial**: present but abbreviated (e.g., only Tiny scale from a Tiny/Small/Base table)
- **Missing**: not found anywhere in the note

### Step 3: Prioritize Gaps
Not all gaps are equal. Priority order:
1. **Missing equations** — these are the paper's core language
2. **Missing insights/remarks** — these are the paper's claimed contributions
3. **Abbreviated tables** — especially classification/benchmark tables where the scaling story matters
4. **Missing figure descriptions** — especially architecture figures
5. **Missing training protocol details** — optimizer, schedule, augmentation

---

## 15. Adapting This Skill to a New Paper

When given a new paper, follow these steps:

### Step 1: Identify the Paper's Type
Different paper types emphasize different section types:

| Paper Type | Heavy Sections | Light Sections |
|---|---|---|
| Methods paper (new architecture) | Architecture, Design Choices, Ablations | Theory |
| Theory paper (new loss/objective) | Derivations, Proofs | Architecture, Downstream |
| Systems paper (efficiency) | Efficiency, Engineering | Novel concepts |
| Empirical paper (benchmark) | Results Tables, Comparisons | Derivations |

### Step 2: Adjust Section Ordering
The ordering in §5.1 is a template. Adapt it:
- If the paper has no nested inner/outer structure, skip that section type.
- If the paper has no code, skip code references (but still note where code would go).
- If the paper introduces a novel training recipe, give it a full section.
- If the paper's main contribution is empirical, the results sections should be more detailed.

### Step 3: Calibrate Depth
- **Core contribution sections**: 4-facet + derivations + detailed tables. Go deep.
- **Context sections** (prior art, related work): narrative + comparison tables. Medium depth.
- **Application sections** (downstream tasks): Setup + Key result. Keep brief.
- **Meta sections** (dependency graph, equations table, reference map): formulaic, just fill them in.

### Step 4: Handle Papers Without Code
If no code repository is available:
- Replace the Application → Code References pattern with
  Application → Implementation Notes (how you would implement it).
- Or leave the code reference slots empty with a `TODO: map to code` marker.

---

## 16. Checklist

Before considering the note complete, verify:

- [ ] §0 Big Picture exists with ASCII art
- [ ] Every concept has all 4 facets (Definition, Properties, Application, Links)
- [ ] All paper equations are included with correct numbering
- [ ] Derivations show step-by-step reasoning
- [ ] Key tables from the paper are reproduced with all model scales
- [ ] Code references use `file:line` format (if code is available)
- [ ] Cross-references are bidirectional
- [ ] Prior Art section compares approaches with specific limitations
- [ ] Quick Reference Card summarizes key insights
- [ ] Concept Dependency Graph covers all concepts
- [ ] Key Equations summary table is complete
- [ ] Reference Map groups all citations by topic
- [ ] Language is consistent throughout (no mixed languages, no LLM filler)
- [ ] No concept is mentioned without being defined in its own section
- [ ] Coverage verification (§14) has been performed against the source paper
