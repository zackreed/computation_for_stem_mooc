# Translate MATLAB Activity to Ximera Walkthrough

## Purpose

This document provides step-by-step instructions for translating a MATLAB Live Script activity (`.m` file with `%[text]` markup) into a Ximera LaTeX document (`.tex` file using `\documentclass{ximera}`). The target audience is an AI copilot that will be given a new MATLAB activity and must produce a corresponding Ximera document.

---

## 1. Overview of the Two Formats

### 1.1 MATLAB Live Script (`.m`)
- Uses `%[text]` comment blocks for rich-text narration (Markdown-like with inline LaTeX via `$…$`).
- Executable code cells separated by `%%`.
- Code calls custom "course functions" (e.g., `animate_segment_cases`, `plot_curve_pieces`, `break_into_pieces`) that produce plots and animations.
- Inline LaTeX uses `{"editStyle":"visual"}` annotations — these are MATLAB editor metadata and should be stripped during translation.
- Students interact by editing code cells, filling in MATLAB expressions, and pressing "Run Section".

### 1.2 Ximera LaTeX Document (`.tex`)
- Document class is `ximera`: `\documentclass{ximera}`.
- Provides interactive online content via several Ximera environments:
  - **Answer blanks**: `\answer[tolerance=…]{value}` for numeric/symbolic fill-in.
  - **Multiple choice (inline)**: `\wordChoice{\choice{wrong}\choice[correct]{right}}` for conceptual questions embedded in prose.
  - **Multiple choice (block)**: `\begin{multipleChoice}…\end{multipleChoice}` with `\choice` and `\choice[correct]` items for standalone multiple-choice questions.
  - **Select-all**: `\begin{selectAll}…\end{selectAll}` with `\choice[correct]{…}` and `\choice{…}` items.
  - **Free response**: `\begin{freeResponse}…\end{freeResponse}` for open-ended reflection.
  - **YouTube embeds**: `\youtube{VIDEO_ID}` for animations and video walkthroughs.
- Pedagogical wrapper environments include:
  - `\begin{problem}…\end{problem}` — graded interactive blocks (contain `\answer`, `\wordChoice`, `\begin{selectAll}`, etc.).
  - `\begin{example}…\end{example}` — guided demonstrations showing expected results and process descriptions.
  - `\begin{remark}…\end{remark}` — conceptual asides, deeper explanations, or important nuances that supplement the main narrative.
- Static images (`.png`) replace MATLAB-generated plots and animations.
- Narrative text lives in normal LaTeX prose; code is *not* directly executable.
- Mathematical content is rendered with standard LaTeX math environments (`$…$`, `$$…$$`, `\[ … \]`).

---

## 2. Pre-Translation: Reading the MATLAB Source

Before writing any LaTeX, carefully read the entire MATLAB `.m` file and build a mental model:

1. **Identify the narrative arc.** The MATLAB script typically follows: Introduction → Concept motivation → Guided example → Reflection questions → Student exercises ("Your Turn"). Note each major section and its heading level (`#`, `##`, `###`).
2. **Catalog every code cell.** For each cell determine:
   - *Purpose*: Is it defining variables, calling a course function for a plot/animation, performing a calculation, or left blank for student input?
   - *Outputs*: Does it produce a plot, a numeric result, or an animation?
   - *Interactivity*: Does the student need to fill in code (`%finish the approximation…`), or is it pre-filled?
3. **List all course functions used** (e.g., `verify_setup`, `animate_segment_cases`, `plot_curve_pieces`, `plot_curve_mass_pieces`, `break_into_pieces`). These will become either:
   - YouTube video embeds (for animations),
   - Static images (for plots), or
   - Removed entirely (for setup/verification utilities).
4. **Collect all LaTeX math** embedded in `%[text]` blocks. Strip the `{"editStyle":"visual"}` metadata.
5. **Identify Reflection Questions.** These are marked with `# Reflection Questions` headings. They become either `\begin{problem}…\end{problem}`, `\begin{example}…\end{example}`, or inline prose questions in the Ximera version.
6. **Identify student-input code cells.** These contain comment prompts like `%r =`, `%rho=`, `%finish the approximation…`. In Ximera, these become either:
   - Guided `\begin{example}…\end{example}` blocks describing what the student should do,
   - `\answer{}` blanks if a specific numeric/symbolic answer is expected, or
   - Omitted (when the work remains solely in the MATLAB live script).

---

## 3. Document Structure: Mapping MATLAB Sections to Ximera

### 3.1 Preamble

```latex
\documentclass{ximera}

\title{<Title from the MATLAB # heading>}
\author{<Author>}

\begin{document}
\begin{abstract}
<1–3 sentence summary derived from the Introduction section of the MATLAB file.>
\end{abstract}
\maketitle
```

**Rules:**
- The `\title{}` comes from the top-level `#` heading in the MATLAB file.
- The `\begin{abstract}` is a concise restatement of the MATLAB introduction, written in third-person or second-person.

### 3.2 Section Headings

| MATLAB `%[text]` heading | Ximera LaTeX |
|---|---|
| `# Heading` | `\section*{Heading}` |
| `## Heading` | `\section*{Heading}` or `\subsection*{Heading}` depending on nesting |
| `### Heading` | `\subsection*{Heading}` |
| `#### Heading` | `\subsection*{Heading}` (Ximera rarely uses deeper nesting) |

- Use `*` (unnumbered) variants unless the document needs numbered sections.
- When a MATLAB heading is a step label like "Step 1: Visualize…", keep the step label in the Ximera heading.

### 3.3 Narrative Prose

- Copy the prose from `%[text]` blocks almost verbatim but:
  - Remove all `{"editStyle":"visual"}` annotations.
  - Convert MATLAB-flavored Markdown (`*italic*`, `**bold**`) to LaTeX (`\emph{}`, `\textbf{}`).
  - Convert bullet lists (`- item`) to `\begin{itemize}\item …\end{itemize}`.
  - Convert numbered lists to `\begin{enumerate}\item …\end{enumerate}`.
  - Inline code references (e.g., `*break_into_pieces*`) become `\texttt{break\_into\_pieces}` or are rephrased as natural language if the Ximera version is not code-focused.
- Preserve the pedagogical tone: second person ("you"), encouraging, step-by-step.

### 3.4 Mathematical Content

- Inline math: `$…$` stays as `$…$`.
- Display math: Use `$$…$$`, `\[ … \]`, or `\begin{equation}…\end{equation}` for numbered/important equations.
- Use `\begin{equation}` for key formulas that the student will refer back to (e.g., Newton's Law, the mass integral). Use `$$…$$` or `\[…\]` for inline derivation steps.
- Ensure all `\vec{}`, `\rho`, `\Delta`, `\Sigma`, `\int`, etc., are proper LaTeX commands.
- Replace MATLAB-specific notation:
  - `.*` → `\cdot` or contextual description.
  - `norm(v)` → `\|\vec{v}\|`.
  - `diff(r)` → narrative "take the derivative of the position vector".
  - `int(f,a,b)` → `\int_a^b f \, dt`.

---

## 4. Translating Interactive / Dynamic Elements

### 4.1 Animations → YouTube Embeds

MATLAB course functions that produce animations (e.g., `animate_segment_cases`) should be replaced by pre-recorded YouTube videos of those animations.

```latex
\begin{center}
\youtube{VIDEO_ID}
\end{center}
```

- If a YouTube video ID is known, embed it directly.
- If no video exists yet, leave a placeholder comment: `% TODO: Record and embed animation for animate_segment_cases(r, rho)`.

### 4.2 Static Plots → Images

MATLAB course functions that produce plots (e.g., `plot_curve_pieces`, `plot_curve_mass_pieces`) should be replaced by static `.png` screenshots.

```latex
\begin{center}
\includegraphics{descriptive_filename.png}
\end{center}
```

- Name images descriptively: `20_seg_wire.png`, `50_mass_approx.png`, etc.
- If images are not yet available, leave a placeholder comment: `% TODO: Generate image for plot_curve_pieces(r,20,rho)`.

### 4.3 `verify_setup` and Personalization

The MATLAB `verify_setup('Name')` system gives each student a personalized curve and density. In the Ximera version:
- Do **not** replicate the personalization system.
- Instead, pick one concrete example (or provide the general form) and state the curve and density explicitly in the text.
- If the activity has a "Your Turn" section referencing personalized constants, describe the general form and show a representative example.

### 4.4 Student Code Cells → Ximera Interactivity

| MATLAB pattern | Ximera translation |
|---|---|
| Blank code cell with comment prompt (`%r = `, `%finish the approximation…`) | `\begin{example}…\end{example}` describing the task, or `\answer{}` if a specific value is expected |
| Code cell that computes a specific numeric answer | `\answer[tolerance=…]{value}` |
| Code cell verifying a previous result | Narrative statement: "This should yield approximately…" |
| Code cell with `sum()`, arithmetic | Describe the operation and expected result in prose or in an `\begin{example}` block |
| `for` loop iterating over examples | Sequential `\begin{example}` blocks or a single block with multiple images |

### 4.5 MATLAB Text/Formula Output → LaTeX Display Math

Some course functions produce text output containing formulas (e.g., `write_error_function` prints the symbolic MSE expression). In Ximera:
- Convert the output formula to proper LaTeX display math.
- Present it inline in the narrative: "The error function is: $$ f(a_0, a_1) = \dots $$"
- If the MATLAB output includes metadata like "Computational Complexity", summarize it in prose rather than reproducing verbatim.

### 4.6 Data Generation / Randomization Functions

MATLAB functions like `generate_example_data`, `generate_student_data`, and `generate_random_model` produce randomized data or models. In Ximera:
- Replace with **fixed, representative examples** shown as static images.
- State the data properties explicitly in prose (e.g., "Consider a data set that linearly decreases from around $(-2, -2.6)$ to $(2, -3)$").
- When the MATLAB shows multiple random outputs (e.g., a `for` loop generating 10 random models), select 1–3 representative cases and show them as images with `\wordChoice` or prose questions about fit quality.

### 4.7 Reflection Questions → Ximera Problem/FreeResponse

MATLAB files contain `# Reflection Questions` sections with bullet-point prompts. These translate to one of:

```latex
\begin{problem}
<Question text>
\begin{freeResponse}
\end{freeResponse}
\end{problem}
```

or, if specific answers are expected:

```latex
\begin{problem}
<Question text with \answer{} blanks>
\end{problem}
```

**Decision rules:**
- If the reflection question asks for a **numeric or symbolic answer** → use `\answer{}`.
- If the reflection question asks for a **conceptual choice** (e.g., "is this a good or poor fit?") → use `\wordChoice{\choice{option A}\choice[correct]{option B}}` embedded in the prose. Multiple `\wordChoice` elements can appear in a single sentence or paragraph.
- If the reflection question asks for **explanation, reasoning, or discussion** → the Ximera version may either use `\begin{freeResponse}` or convert to a `\begin{selectAll}` / `\wordChoice` if the author can anticipate discrete answer options.
- If the reflection is more of a **conceptual aside** (explaining nuance, correcting a common misconception), wrap it in `\begin{remark}…\end{remark}` with embedded `\wordChoice` for engagement.
- **Many reflection questions from the MATLAB file are omitted in the Ximera version** and left only in the live script submission. This is a judgment call: include questions that test conceptual understanding or have verifiable answers; omit open-ended prompts that are better suited for written submissions.

---

## 5. Translating Specific Pedagogical Patterns

### 5.1 Recognizing the Pedagogical Arc

Different activities follow different pedagogical arcs. The copilot must **identify the arc in the MATLAB source first** and then preserve it in the Ximera version. Common arcs include:

#### Arc A: "Approximate → Refine → Integrate" (used in mass, gravity, work activities)
1. **Motivate** the basic model (e.g., mass = density × length).
2. **Show failure** of the basic model under variability.
3. **Break into pieces** (finite approximation / Riemann sum).
4. **Refine** by increasing N.
5. **Take the limit** → integral.
6. **Compute** the integral using FTC.

Key translation choices:
- Steps 1–2 use prose + YouTube videos of animations.
- Step 3 uses a mix of images, `\answer{}` blanks for student calculations, and `\begin{example}` blocks.
- Step 4 uses images of successive approximations and/or a table of N vs. approximation values.
- Steps 5–6 use display math and prose. The symbolic derivation is written out fully in LaTeX.

#### Arc B: "Context → Model → Optimize → Apply" (used in gradient descent / classification)
1. **Introduce the real-world problem** (e.g., model selection, fitting data).
2. **Define the mathematical model** (e.g., polynomial, error function).
3. **Motivate the optimization technique** (e.g., gradient descent).
4. **Walk through examples** at increasing complexity (e.g., linear → quadratic → cubic).
5. **Student applies** the technique to their own data.

Key translation choices:
- Steps 1–2 use prose with static images of example data and models.
- Step 3 uses `\youtube{}` embeds for gradient descent animations, `\begin{remark}` blocks for conceptual nuance about input vs. output spaces.
- Step 4 uses sequential `\begin{example}` blocks, each with images and/or videos, plus `\wordChoice` for comprehension checks.
- Step 5 maps to the "Your Turn" section with general instructions.

#### General Rule
Always preserve the MATLAB section order. The Ximera version should mirror the flow of the MATLAB file, translating each section type (motivation, guided example, practice, reflection) into the appropriate Ximera environment.

### 5.2 Video Walkthrough Insertion Points

The MATLAB live script does not explicitly reference videos, but the Ximera version inserts video walkthroughs at natural breakpoints. When translating:
- Insert a `\youtube{}` embed after the introduction, before the first interactive section.
- Insert additional `\youtube{}` embeds at transitions between major phases (e.g., after approximation, before integration).
- Preface each video with a one-line description: "The next video covers…".

### 5.3 Guided Code Examples → `\begin{example}`

When the MATLAB script walks students through a calculation step-by-step with pre-filled code, the Ximera version wraps the equivalent explanation and expected results in:

```latex
\begin{example}
<Description of what the student is asked to do in the live script.>
<Expected results or representative output.>
\end{example}
```

This tells the student what they should see and do without requiring code execution in the browser.

---

## 6. Handling the "Your Turn" / Student Exercise Sections

MATLAB activities typically end with a "Your Turn!" section where students apply the process to their own personalized data. In the Ximera version:

1. State the general task clearly.
2. Provide a representative example curve and density (from one `verify_setup` instance) so the student knows the expected form.
3. Show an example image of what a correct plot might look like.
4. Do **not** attempt to replicate `verify_setup` personalization — that remains in the MATLAB live script.
5. End with an encouraging closing statement.

---

## 7. Additional Translation Patterns

### 7.1 Parametric MATLAB Functions → Multiple Ximera Problem Blocks

When a single MATLAB course function takes different arguments to produce different outputs (e.g., `generate_solid_gravity()` producing sphere, cone, cylinder), the Ximera version should create **separate `\begin{problem}` blocks** for each variant. Each block should:
- Include its own static image.
- Pose `\wordChoice` or `\begin{multipleChoice}` questions specific to that variant.
- Build toward the student understanding the general principle across cases.

This is a common pattern: one MATLAB function ↔ multiple Ximera interactive blocks.

### 7.2 Shared Content Across Related Activities

When two activities share introductory content (e.g., `gravityWire` and `gravitySolid` both start with Newton's Law of Gravity and the `animate_two_masses()` animation):
- **Reuse the same `\youtube{}` embed ID** for the shared animation.
- Keep the shared prose consistent but allow minor rephrasing to match the specific context (wire vs. solid).
- The Ximera version need not duplicate every sentence; a briefer recap suffices if the student has seen the material before.

### 7.3 MATLAB Key-Value Arguments → Prose Descriptions

MATLAB functions sometimes accept key-value arguments (e.g., `generate_solid_gravity('gravity_mode','true','arrow_mode','net')`). These internal implementation details should not appear in the Ximera version. Instead:
- Describe the effect of the arguments in natural language.
- Show the resulting plot or animation as an image or video.

### 7.4 Optional Sections

If the MATLAB file has an optional/extension section (often at the end), preserve it in the Ximera version as a clearly marked optional subsection. These often introduce more advanced formulations (e.g., true vector gravity vs. simplified scalar gravity).

### 7.5 Coordinate System and Setup Discussions

MATLAB activities working with non-Cartesian coordinates (cylindrical, spherical) often leave the coordinate system implicit in the code. The Ximera version should **make these explicit**:
- State the coordinate system being used.
- Write out the coordinate transformation equations.
- Use `\wordChoice` to test whether the student understands how the coordinate system affects volume elements, density representations, and distance formulas.

### 7.6 Building Intuition with Sequences of `\wordChoice`

A powerful Ximera pattern is embedding 3–6 sequential `\wordChoice` elements within a `\begin{problem}` or `\begin{enumerate}` to walk the student through a chain of reasoning. This is often used when the MATLAB version has a single Reflection Question asking the student to "explain" something. The Ximera version decomposes the explanation into guided fill-in-the-blank reasoning steps.

**Example pattern:**
```latex
\begin{problem}
Fill in the following statements:
\begin{enumerate}
    \item At the top of the cone, the radius $r$ is \wordChoice{\choice{larger}\choice[correct]{smaller}} than at the bottom.
    \item Because of this, the volume of pieces at the top is \wordChoice{\choice{larger}\choice[correct]{smaller}}.
    \item This means the gravity at the top is \wordChoice{\choice{larger}\choice[correct]{smaller}}.
\end{enumerate}
\end{problem}
```

### 7.7 Animation + Static Image Pairing

A common pattern in the Ximera files is to pair a YouTube animation with a follow-up static image. The animation shows a dynamic process (e.g., density variation across a cube), and the static image provides a reference snapshot for the subsequent `\wordChoice` or `\answer{}` questions. When translating:
- Place the `\youtube{}` embed first.
- Follow it immediately with `\includegraphics{}` showing the final state or a representative frame.
- Then pose the interactive question referencing both.

### 7.8 Image Sizing

When images need to be smaller than full width, use the optional width argument:
```latex
\includegraphics[width=0.4\textwidth]{image.png}
```
Use this for contextual images (e.g., a photo of separated liquids) that are illustrative rather than primary data.

### 7.9 Instructor-Only Content in MATLAB

Some MATLAB files contain instructor-only comments (e.g., `% ****This script is ONLY FOR INSTRUCTORS...****`) or solution text. **Strip all instructor-only content** from the Ximera version. The Ximera document is student-facing.

### 7.10 MATLAB Comment Format Variations

MATLAB Live Scripts come in two comment formats:
1. `%[text]` blocks with `{"editStyle":"visual"}` annotations (newer format).
2. Standard `%` comments with Markdown-like formatting (older/exported format).

Both formats carry the same semantic content. The copilot should handle either format identically: extract the prose, strip metadata, and translate to LaTeX.

### 7.11 Cross-Referencing the MATLAB Live Script from Ximera

The Ximera document serves as a **companion** to the MATLAB live script, not a replacement. Several patterns make this relationship explicit:

- Use phrases like "In the livescript you are instructed to…", "The livescript walks you through…", "You are given course functions to…".
- When the MATLAB file has a multi-step manual calculation that the student must do in code, the Ximera version can summarize the process in a `\begin{remark}` block listing the steps, rather than reproducing the code.
- The Ximera version should tell students what they will **see** and **do** in the live script, and what the expected output should look like, without reproducing the code itself.

### 7.12 Multiple Basic Models in a Single Activity

Some activities introduce more than one basic model (e.g., the work/drag activity introduces both the drag force model and the work/dot-product model). When translating:
- Present each basic model in its own subsection with its own equation block.
- Clearly label them (e.g., "Basic Model 1: Drag Force", "Basic Model 2: Work").
- Show how they combine in the approximation step.

### 7.13 External References / Sources

MATLAB files may reference external sources (e.g., NASA URLs). These are generally **not carried** into the Ximera version unless they add pedagogical value. If a link is important, it can be included as a footnote or inline URL, but most are omitted.

### 7.14 Using `\begin{remark}` for Process Summaries

When the MATLAB file walks through a detailed multi-step calculation process, the Ximera version often wraps a summary of the steps in `\begin{remark}…\end{remark}`. This signals to the student: "Here is the general process you'll follow in the live script." This is distinct from `\begin{example}` (which shows a specific worked case) and `\begin{problem}` (which requires student input).

### 7.15 TikZ Diagrams for Inline Illustrations

When a MATLAB plot is simple enough to reproduce as a vector graphic (e.g., a single 3D vector, a simple geometry diagram), the Ximera version may use a `\begin{tikzpicture}` environment instead of a static image. Use TikZ for:
- Simple coordinate axes with labeled vectors.
- Geometric diagrams (lines, planes, basic shapes).
- Situations where a rasterized image would be unnecessarily heavy.

Do **not** use TikZ for complex data plots, 3D surfaces, or anything with many data points — use `\includegraphics` for those.

### 7.16 The "Fail-Then-Succeed" Pedagogical Pattern

Some activities deliberately show a method **failing** before presenting the correct approach. For example, the classification activity attempts linear separation on circular data (which fails at ~50% accuracy), then introduces polynomial features to succeed. When translating:
- Preserve this two-phase structure.
- Show the failure with an image or video, along with `\wordChoice` or `\begin{multipleChoice}` to confirm the student understands **why** it fails.
- Then present the solution approach with its own images/videos/examples.

### 7.17 Cross-Module References

MATLAB activities often reference earlier modules (e.g., "Unlike the Module 6 mini project…", "You know how to do this from Module 1"). Preserve these cross-references in the Ximera version. They help students connect concepts across the course.

### 7.18 Commented-Out MATLAB Code in Ximera

Some Ximera files include commented-out MATLAB code (e.g., `% \begin{verbatim}…\end{verbatim}`) as development artifacts. When creating a new Ximera file:
- **Do not include raw MATLAB code** in the final document.
- If the code is pedagogically useful (showing the student what they'll type), describe it in prose within a `\begin{remark}` or `\begin{example}` block instead.

### 7.19 Continuing Numbered Lists Across Environments

When a step-by-step process spans multiple `\begin{enumerate}` blocks (e.g., interrupted by prose or equations), use `\setcounter{enumi}{N}` to continue numbering:
```latex
\begin{enumerate}
  \setcounter{enumi}{3}
  \item Step 4…
  \item Step 5…
\end{enumerate}
```

### 7.20 Presenting Learned/Computed Results

When the MATLAB activity produces specific numeric results (e.g., learned parameters, error values, accuracy percentages), the Ximera version should:
- State the specific values explicitly (e.g., "The learned parameters are $W = [-0.099, 2.309, -0.325]$").
- Show the accuracy or error metric.
- Display the resulting plot as an image.
- These concrete values help the student verify their own work in the live script.

---

## 7. Final Checklist

Before considering the Ximera document complete, verify:

**Document structure:**
- [ ] `\documentclass{ximera}` is used.
- [ ] `\title{}`, `\author{}`, `\begin{abstract}`, `\maketitle` are present.
- [ ] Section headings mirror the MATLAB structure using `\section*` / `\subsection*`.
- [ ] The pedagogical arc (whichever type) is intact and in the correct order.

**Content translation:**
- [ ] All MATLAB `%[text]` prose is translated to LaTeX prose.
- [ ] All `{"editStyle":"visual"}` metadata is stripped.
- [ ] MATLAB Markdown formatting (`*italic*`, `**bold**`) is converted to LaTeX (`\emph{}`, `\textbf{}`).
- [ ] Bullet and numbered lists use `\begin{itemize}` / `\begin{enumerate}`.
- [ ] Mathematical notation is valid LaTeX (no MATLAB syntax leaking through).
- [ ] No executable MATLAB code remains in the `.tex` file (no commented-out `\begin{verbatim}` blocks).
- [ ] No instructor-only content is present.

**Interactive elements:**
- [ ] All animations have `\youtube{}` embeds (or `% TODO:` placeholders).
- [ ] All plots have `\includegraphics{}` (or `% TODO:` placeholders).
- [ ] All reflection questions with verifiable answers are translated to `\begin{problem}` with `\answer{}`, `\wordChoice`, `\begin{multipleChoice}`, or `\begin{selectAll}`.
- [ ] All `\answer{}` blanks have appropriate `[tolerance=…]` when numeric.
- [ ] Open-ended reflection questions use `\begin{freeResponse}` or are noted as live-script-only.
- [ ] Guided calculations use `\begin{example}` blocks with expected results.
- [ ] Conceptual asides use `\begin{remark}` blocks.

**Media and formatting:**
- [ ] Images are named descriptively and sized appropriately (`[width=…]` when needed).
- [ ] YouTube + static image pairings are used where animations precede questions.
- [ ] TikZ diagrams are used only for simple geometric illustrations.

**Pedagogical integrity:**
- [ ] Cross-module references are preserved.
- [ ] The "Your Turn" section describes the task without replicating `verify_setup` personalization.
- [ ] Specific numeric results (learned parameters, approximation values) are stated for verification.
- [ ] Optional sections are clearly marked as optional.

**Final validation:**
- [ ] The document compiles (or would compile) without errors under the `ximera` document class.
- [ ] A student reading both the Ximera page and the MATLAB live script would have a coherent, non-redundant experience.

---

## 8. Reference Files

The following folder pairs each contain a MATLAB source and its corresponding Ximera translation, and serve as reference examples:

| Folder | MATLAB file | Ximera file | Topic |
|---|---|---|---|
| `massWire/` | `mass_walkthrough.m` | `massOfWire.tex` | Mass of a wire with variable density |
| `massSolid/` | `solid_mass_walkthrough.m` | `massOfSolid.tex` | Mass of a solid |
| `gravityWire/` | `gravity_walkthrough.m` | `gravityWire.tex` | Gravitational force along a wire |
| `gravitySolid/` | `multivar_gravity.m` | `gravitySolid.tex` | Gravitational force of a solid |
| `gradientDescent/` | `gradient_descent_walkthrough.m` | `gradientDescent.tex` | Gradient descent / classification |
| `classification/` | `classification_final_project.m` | `gradientDescentClassification.tex` | Classification via gradient descent |
| `work/` | `drag_walkthrough.m` | `workAgainstDrag.tex` | Work against drag |
