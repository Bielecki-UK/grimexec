# grimexec & grimepint

> Run commands on dirty files inside containers - fix legacy codebases one commit at a time.

When working with **older** or **legacy repositories** that don't fully follow **PSR 12** yet, I prefer to `pint` **only the dirty files** during each commit.  
It lets me gradually bring the codebase up to standard without blindly reformatting everything - keeping commit history clean and trackable.

In containerised setups, where the git repo lives outside the container, tools like `pint` can't use the `--dirty` argument.
**grimepint** solves this by manually detecting dirty files on the host and passing them into the container, so you can lint or format just what changed.  
**grimexec** is a tiny bash helper that runs commands only on your *dirty (changed)* files inside a Docker container.
Perfect for formatting, linting, or any operations where you want to target just your changes instead of reprocessing the whole codebase every time.

---

## What it does

- Detects staged and unstaged changes (via `git diff` and `git diff --cached`).
- Filters to PHP files (`*.php`) — can easily be modified to match other file types.
- Prepares file paths, adjusting them from your local project structure to the container's structure.
- Runs your command inside the specified Docker container, passing all dirty files as arguments.
- Exits with `1` if changes (like formatting fixes) were made, or `0` otherwise — good for CI usage or as your pre-commit hook!

---

## Requirements

- `docker compose`
- `git`
- Dirty PHP files in your repo (obviously)

---

## Configuration

At the top of the script, you can configure:

```bash
container_name="php"             # Your service/container name
base_command="vendor/bin/pint"   # Command to run
local_base_path="src/"           # Local base path to strip
container_base_path="/var/www/html/"  # Container base path prefix
```

---

## Usage

### grimepint

Run pint (or any default command set in base_command) against all dirty PHP files:

```bash
./grimepint
```

You can pass additional arguments to your base command:

```bash
./grimepint --test
```

The script automatically appends the list of dirty files to the command.

### gimexec

More generic version of the script where the whole command is passed when calling:

```bash
./grimexec vendor/bin/phpstan analyse
```

Or anything you want:

```bash
./grimexec vendor/bin/phpunit
```

The script appends dirty file at the end of the command and repeats for each file.

---

## Notes

- Default setup assumes your PHP files are under `src/`.
- Container paths must match exactly how they appear inside the container.
