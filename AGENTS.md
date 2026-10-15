# Repository Guidelines

## Project Structure & Responsibilities

`src/agilex_sdk_python/` contains the PiPER-X hardware layer. `reader.py` manages read-only connections; `driver.py` owns feedback validation and guarded hardware operations. `gello_follower.py` adapts six joints plus gripper to the seven-dimensional Robot protocol; `gello_server.py` exposes it over ZMQ. `cli.py` handles diagnostics and commands. `scripts/config_can.sh` configures SocketCAN; `tests/` mocks the SDK; `docs/` contains user references.

Keep GELLO device calibration, follower mapping, client composition, application lifecycle, and raw recording in agilex-controller. Timestamp alignment and dataset writing belong to lerobot-converter. Do not add those applications as SDK dependencies.

## Build, Test, and Development Commands

Use Python 3.11 and uv from this repository root. Package metadata permits newer interpreters, but the controller integration uses 3.11.

- `uv sync --frozen`: synchronize dependencies and development tools.
- `uv run --frozen agilex-sdk-python --help`: inspect commands without CAN access.
- `uv run --frozen pytest`: run hardware-free regressions.
- `uv run --frozen ruff check --isolated .`: validate Python lint.
- `uv run --frozen pyright`: check types using the local environment.
- `bash -n scripts/config_can.sh`: check shell syntax.
- `uv build`: build distributions.

## Coding & Testing Guidelines

Use four spaces, annotated public interfaces, snake_case functions/modules, and PascalCase classes. Tests use `test_*.py`; no coverage threshold or dedicated formatter is configured. Ruff has no repository configuration; use `--isolated` to avoid inheriting a parent checkout's rules. Inject fake SDK objects and clocks to test missing or stale feedback, errors, enable state, limits, command pacing, gripper mapping, and cleanup.

Read-only commands must not enable or move hardware. Preserve feedback validation and bounded waits. Closing a connection does not imply zeroing or disabling. Real hardware verification requires operator confirmation and documented results. The server enables hardware and normally writes motion limits; legacy confirmation flags do not provide interactive approval.

## Commits & Pull Requests

Prefer concise imperative subjects with conventional prefixes. Describe behavior, affected protocol/configuration, checks, hardware-validation status, and related issues. Update README, reference docs, and CHANGELOG with behavior changes.

Publish tested SDK commits before updating a controller gitlink. Standalone and submodule checkouts are independent. Never commit virtual environments, data, caches, or build artifacts. No project-wide license is selected; preserve dependency notices and do not invent a license grant.
