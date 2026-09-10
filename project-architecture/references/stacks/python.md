# Checklist — Python projects

Where the non-derivable facts hide on this stack. This is a list of places to look, not a list of
facts to write: everything found here still has to pass the admission rule in
`references/produced-file.md`.

## Interpreter and environment

- Is there a `venv/`, `.venv/`, `.python-version`, `uv.lock`, `Poetry.lock`, `Pipfile`?
- Is there any dependency manifest at all — `pyproject.toml`, `requirements.txt`, `setup.py`?
- If there is none and the code still imports third-party packages, find out which interpreter
  actually runs it (`python3 -V`, `python3 -c "import <pkg>"`). A project running on the system
  interpreter with no manifest is admissible: the default move is to create a venv and install, and
  that is the wrong move.

## Test command

- Which runner does the project actually use: `pytest`, `python -m unittest`, `tox`, `nox`?
- Is the runner an agent would reach for first even installed? Check, do not assume.
- Is there a wrapper — `Makefile`, `justfile`, `tox.ini`, `[project.scripts]`, a `scripts/` shell
  file — that chains several stages under one name? If so, record what it chains and what a red
  result does NOT mean.
- Test file naming that a default `pytest` invocation would miss, or vice versa.

## Generated files

The highest-value find on this stack, because hand-editing one is silent work lost.

- Grep for writers: `open(`, `.write_text(`, `.write(`, `json.dump(`, `yaml.dump(`, `ET.write(`,
  `csv.writer`, template renders.
- For each output path, check whether that file is committed. A committed, generated file is an
  entry: name the generator and name what to edit instead.

## Source of truth among the data files

- Root-level `*.yml`, `*.toml`, `*.json`, `*.xml` that are not manifests: which one is written by
  hand and which is produced?
- If two files describe the same thing, say which one loses on the next run.

## Superseded documents

- A `README.md` describing an earlier design while another document describes the current one. An
  agent reads `README.md` by default, so the supersession is admissible and belongs in Traps.
- Check the date and the opening paragraph of each root-level document before trusting it.

## Import and layout mechanics

- Flat modules versus a package: is there an `__init__.py`? Are modules imported by path from the
  repo root?
- Any `sys.path` manipulation, `conftest.py` that inserts paths, or reliance on the current working
  directory. Scripts that only work when run from the repo root are admissible.

## Not admissible on this stack

Everything the agent can read the moment it needs it:

- the dependency list, whatever file holds it;
- the framework or library names in use;
- a listing of the module layout — the rule that governs where a new module belongs is admissible
  in `## Shape`, the inventory is not;
- function signatures and docstrings.
