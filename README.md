# Phase I: Issue Selection

### Chosen Issue
- **Project:** Qiskit
- **Issue Link:** [Qiskit Issue #3436](https://github.com/Qiskit/qiskit/issues/3436)

### Problem Summary
The `qiskit.transpiler.passes` module currently lists individual transpiler passes but completely lacks high-level module documentation, structural categorization, and clear usage examples. Because it is presented as a massive, flat list of code components, developers struggle to understand what individual passes do or how to leverage them outside of preset optimization configurations. The goal of this issue is to rewrite the module-level documentation to categorize these passes logically and provide clear example workflows for custom compilation routines.

### Why I Chose This Issue
I chose this issue because it aligns beautifully with my dual focus in Computer Science and Applied Physics. Working on Qiskit's compilation pipeline provides an incredible entry point into an active, enterprise-grade quantum computing framework. 

I'm interested in this specific task because:
1. **Strong Architectural Grounding:** The transpiler is core to how quantum circuits are optimized for physical hardware; diving deep into its passes will give me a fundamental understanding of Qiskit's compiler design.
2. **High-Impact Documentation:** Clear documentation directly lowers the barrier to entry for other research developers trying to build custom PassManagers.
3. **Manageable Scope:** Because this is a documentation and example-focused task, I can focus heavily on reading the codebase, mapping module structures, and learning the developer workspace without getting immediately bottlenecked by complex logic refactoring.

From studying the issue history, I see that while this is an older, highly requested documentation gap, it has recently seen active maintainer discussion and a fresh community PR open. My contribution will focus on building out an independent, comprehensive layout for these missing category docs and examples.

I am tracking the project status and am ready to set up my local environment next to look at how Qiskit's Sphinx documentation builds.

# Phase II: Reproduce & Plan

## Reproduction Process

### Environment Setup
- **OS:** Microsoft Windows 11 Home, version `10.0.26200`, 64-bit
- **Project fork:** [pathseekingscholar/qiskit](https://github.com/pathseekingscholar/qiskit)
- **Working branch:** [fix-issue-3436](https://github.com/pathseekingscholar/qiskit/tree/fix-issue-3436)
- **Local checkout:** `C:\Users\vedar\Documents\Codex\2026-06-17\you-are-an-expert-software-engineering\work\qiskit`
- **Current branch commit tested:** `0b156c49bff801870249aa8eae98fcfb59180849`
- **Python environment:** Python `3.12.13` virtual environment at `.venv`
- **pip:** `26.1.2`
- **tox:** `4.55.1`

I cloned my Qiskit fork, added the upstream remote, fast-forwarded local `main` from `Qiskit/qiskit`, created the branch `fix-issue-3436`, and pushed it to my fork.

Qiskit's current `CONTRIBUTING.md` says API docs are built with Sphinx through:

```powershell
tox -e docs
```

The current `tox.ini` expands that docs environment to:

```text
make cheader
doxygen docs/Doxyfile
sphinx-build -W -j auto -T --keep-going -b html docs/ docs/_build/html
```

The local docs build is currently blocked before Sphinx runs because Qiskit must build native Rust extensions from source. The exact blocker from both `pip install -e .` and `tox -e docs` is:

```text
error: can't find Rust compiler
```

Additional missing native tools found on PATH:

- `cargo`
- `rustc`
- `make`
- `doxygen`

This is useful reproduction evidence because the issue is a documentation issue: reproducing it means reaching the local documentation build path and identifying the project-specific tooling required to render the Sphinx HTML page.

### Steps to Reproduce Documentation State
1. Clone my fork:

   ```powershell
   git clone https://github.com/pathseekingscholar/qiskit.git
   cd qiskit
   ```

2. Add the upstream project remote and update from upstream `main`:

   ```powershell
   git remote add upstream https://github.com/Qiskit/qiskit.git
   git fetch upstream main --tags
   git checkout main
   git pull --ff-only upstream main
   ```

3. Create and publish the working branch:

   ```powershell
   git checkout -b fix-issue-3436
   git push -u origin fix-issue-3436
   ```

4. Create and activate a virtual environment. I used Python `3.12.13` because current Qiskit `pyproject.toml` allows Python `>=3.10`, and its documentation dependency group specifically requires Python `>=3.12`.

   ```powershell
   python -m venv .venv
   .\.venv\Scripts\Activate.ps1
   python -m pip install -U pip setuptools wheel
   ```

5. Follow Qiskit's current contribution instructions for editable install and developer dependencies:

   ```powershell
   python -m pip install -e .
   python -m pip install --group dev
   ```

   On my Windows setup, `python -m pip install -e .` currently fails because Rust is not installed:

   ```text
   error: can't find Rust compiler
   ```

6. Install tox and run the documented API docs build command:

   ```powershell
   python -m pip install tox
   python -m tox -e docs
   ```

   This reaches Qiskit's packaging step but fails with the same Rust compiler error before `sphinx-build` can produce `docs/_build/html`.

7. Locate the page that renders the relevant documentation:

   ```powershell
   Get-Content docs\apidoc\transpiler_passes.rst
   Get-Content qiskit\transpiler\passes\__init__.py
   ```

   `docs/apidoc/transpiler_passes.rst` renders `qiskit.transpiler.passes` with `automodule`. The content comes from the module docstring in `qiskit/transpiler/passes/__init__.py`.

8. Inspect the current documentation state:

   ```powershell
   rg -n "autosummary|Transpiler Passes|Layout Selection|Routing|Optimizations|Synthesis" qiskit/transpiler/passes/__init__.py docs/apidoc/transpiler_passes.rst
   ```

   Current upstream `main` already includes category headings such as Layout Selection, Routing, Basis Change, Optimizations, Scheduling, Circuit Analysis, Synthesis, Post Layout, and Additional Passes. However, the page is still mostly `autosummary` lists. It lacks a strong module-level explanation of how passes fit into the transpiler pipeline and lacks cohesive usage examples for running passes directly or composing them in a `PassManager`.

### Reproduction Evidence

- **Working branch:** [https://github.com/pathseekingscholar/qiskit/tree/fix-issue-3436](https://github.com/pathseekingscholar/qiskit/tree/fix-issue-3436)
- **Docs page source:** `docs/apidoc/transpiler_passes.rst`
- **Rendered module content source:** `qiskit/transpiler/passes/__init__.py`
- **Expected docs output after toolchain setup:** `docs/_build/html`
- **Current build blocker:** missing Rust compiler, followed by likely missing `make` and `doxygen`

## Solution Approach

### Implementation Plan

**Understand:**  
This issue is about improving documentation for `qiskit.transpiler.passes`. The current API page is generated from the module docstring in `qiskit/transpiler/passes/__init__.py`. On current upstream `main`, the page is partially categorized, so the remaining gap is not simply "make one flat list into headings." The stronger fix is to add module-level explanation, category guidance, and short examples that help users understand when and how to use passes outside of preset pass managers.

**Match:**  
The existing docs system already uses Sphinx `automodule` and `autosummary`. The relevant pattern is:

- `docs/apidoc/transpiler_passes.rst` delegates to `qiskit.transpiler.passes`.
- `qiskit/transpiler/passes/__init__.py` uses reStructuredText headings and `.. autosummary::` blocks.
- `qiskit/transpiler/passes/synthesis/plugin.py` is a useful related example of a module-level documentation page that includes conceptual explanation before API references.

**Plan:**

1. Edit `qiskit/transpiler/passes/__init__.py`.
2. Add a concise top-level overview explaining that transpiler passes are reusable `AnalysisPass` and `TransformationPass` units run through a `PassManager`.
3. Add a short "Using transpiler passes" section with examples showing:
   - running an analysis pass and reading the pass manager property set,
   - chaining layout and routing passes,
   - running a basis translation or optimization pass.
4. Keep the current category structure, but add a short explanatory paragraph before each category's `autosummary` block.
5. Make the category descriptions map to the real transpiler workflow:
   - Layout selection and allocation
   - Routing
   - Basis translation and decomposition
   - Synthesis
   - Optimization
   - Scheduling and timing
   - Circuit analysis
   - Validation and utility passes
6. Avoid changing public API exports or runtime behavior. This should be a documentation-only change.
7. After installing Rust, Visual Studio C++ build tools, GNU Make, and Doxygen, run `tox -e docs` and preview the generated HTML page under `docs/_build/html`.

**Implement:**  
Implementation will happen on branch [fix-issue-3436](https://github.com/pathseekingscholar/qiskit/tree/fix-issue-3436). The expected primary file to modify is:

- `qiskit/transpiler/passes/__init__.py`

Potential supporting files to inspect but probably not edit:

- `docs/apidoc/transpiler_passes.rst`
- `docs/conf.py`
- individual pass modules under `qiskit/transpiler/passes/*`

**Review:**  
I will self-review against Qiskit's `CONTRIBUTING.md`, especially the docs build guidance. Since Sphinx is run with `-W`, all warnings count as errors, so I will check that cross-references resolve and that the examples use stable public APIs.

**Evaluate:**  
The main verification command is:

```powershell
python -m tox -e docs
```

Successful evaluation means:

1. The Sphinx HTML build completes without warnings-as-errors.
2. `docs/_build/html` is generated.
3. The `qiskit.transpiler.passes` page renders the new overview, category descriptions, and examples.
4. No runtime API or exported names change.
