# AGENTS.md

See `CLAUDE.md` for the full architecture/development guide (kernel + plugin design, node tree, job pipeline, code style, testing patterns). The notes below are specific to running this project inside the Cursor Cloud VM.

## Cursor Cloud specific instructions

MeerK40t is a single-process Python desktop app (kernel + plugins). There is no database, broker, or separate backend service; optional network servers (web/telnet/UDP) run inside the same process and are not needed for development or testing.

### Environment
- Python 3.12 on Ubuntu 24.04. The startup update script installs all deps system-wide with `pip --break-system-packages` (no venv). `~/.local/bin` is added to `PATH` in `~/.bashrc`; prefer `python3 -m <tool>` invocations to be PATH-independent.
- `wxPython` (the GUI) must be installed from the prebuilt Linux wheel index (`-f https://extras.wxpython.org/wxPython4/extras/linux/gtk3/ubuntu-24.04`). A plain `pip install wxPython` / `pip install -r requirements-dev.txt` tries to build from source and fails. The update script installs the wheel first so the later `requirements-dev.txt` install sees `wxPython` already satisfied and skips the source build.

### Run / test / lint (standard commands; see `CLAUDE.md`)
- Tests: `python3 -m unittest discover test -v` (CI parity) or `pytest -v`.
  - Known pre-existing failure unrelated to setup: `test_esp3d_upload.TestESP3DExecute.test_execute_file_success` fails on clean `main` (a mock assertion, not environmental).
- Lint (all `continue-on-error` in CI; the repo currently has many style warnings): `python3 -m pflake8 meerk40t`, `python3 -m black --check meerk40t`, `python3 -m isort meerk40t -c --diff`, `pylint meerk40t`.

### Running the application
- Entry point is the root script `meerk40t.py` (run `python3 meerk40t.py`). There is no `meerk40t/__main__.py`, so `python -m meerk40t` does NOT work.
- Headless / scripted (no display needed): `python3 meerk40t.py -z -e "<console cmd>" -e "quit"`. `-e` is repeatable; commands run in order. Example that creates a rect and exports G-code:
  `python3 meerk40t.py -z -e "service device start -i grbl 0" -e "rect 2cm 2cm 1cm 1cm engrave -s 15 plan copy-selected preprocess validate blob preopt optimize save_job out.gcode" -e "quit"`
- GUI: a TigerVNC desktop is available on `DISPLAY=:1`. Launch with `DISPLAY=:1 python3 meerk40t.py`. The Gtk pixbuf/`ClientToScreen`/"lost focus" debug warnings on startup are harmless. A first-run "Tips" dialog opens on top of the main window; close it to reach the canvas.
