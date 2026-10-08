# Bash Scripting Interview: Questions & Answers (Senior DevOps)

A question bank with model answers for interviewing senior DevOps candidates. Each answer lists the key points a strong candidate should hit.

---

## 1. Fundamentals and Robustness

### Q1. What do `set -e`, `set -u`, `set -o pipefail` and `set -x` do? Why is `set -e` alone not reliable?

- `set -e` (errexit): exit when a command returns non-zero.
- `set -u` (nounset): treat unset variables as an error.
- `set -o pipefail`: a pipeline returns the exit status of the last command that failed, not just the last command.
- `set -x` (xtrace): print each command before executing it (useful for debugging).

**Why `set -e` is unreliable:**
- It is ignored in conditions: `if cmd`, `while cmd`, `until cmd`, and in `cmd1 && cmd2 || cmd3` for all but the last command.
- It is ignored for commands negated with `!`.
- It is suppressed inside functions called from those contexts.
- `local x=$(failing_cmd)` and `export x=$(failing_cmd)` mask the failure, because the exit status is that of `local`/`export`. Declare first, then assign.
- Command substitutions do not inherit `-e` unless `shopt -s inherit_errexit` (Bash 4.4+).

Best practice: `set -Eeuo pipefail` plus explicit error checks and a `trap ... ERR`.

---

### Q2. What is the difference between `$@` and `$*`, quoted and unquoted?

- `"$@"`: each argument stays a separate word (almost always what you want).
- `"$*"`: all arguments joined into one string, separated by the first character of `IFS`.
- Unquoted `$@` and `$*`: both are word-split and glob-expanded, so arguments with spaces break apart.

```bash
set -- "a b" "c"
for x in "$@"; do echo "[$x]"; done   # [a b] [c]
for x in "$*"; do echo "[$x]"; done   # [a b c]
```

---

### Q3. Explain `[ ]` vs `[[ ]]` vs `(( ))`.

| Construct | Notes |
|---|---|
| `[ ]` | The POSIX `test` command. Portable, but variables must be quoted and `-a`/`-o` are fragile. |
| `[[ ]]` | Bash keyword. No word splitting or globbing on variables, supports `&&`/`||`, pattern matching (`==  *.log`) and regex (`=~`). Not POSIX. |
| `(( ))` | Arithmetic evaluation. `(( count > 5 ))`, `(( i++ ))`. Returns 0 if the result is non-zero. |

Use `[ ]` for POSIX `sh` scripts, `[[ ]]` in Bash scripts, and `(( ))` for numeric work.

---

### Q4. How do you handle filenames with spaces, newlines or leading dashes safely?

- Always quote expansions: `"$file"`.
- Use `--` to end option parsing: `rm -- "$file"`; or prefix with `./`.
- Never parse `ls`. Use globs (`for f in ./*.log`) or `find`.
- Use null delimiters: `find . -type f -print0 | xargs -0 cmd`.
- Read null-delimited input: `while IFS= read -r -d '' f; do ...; done < <(find . -print0)`.
- Use `read -r` so backslashes aren't interpreted, and `IFS=` to preserve leading/trailing whitespace.

---

### Q5. What is the difference between `source script.sh`, `./script.sh` and `bash script.sh`?

- `source script.sh` (or `. script.sh`): runs in the **current shell**. Variables, functions and `cd` persist. No exec bit needed.
- `./script.sh`: runs as a **child process** using the shebang. Needs the executable bit. Changes don't affect the parent.
- `bash script.sh`: child process using Bash explicitly, ignoring the shebang. No exec bit needed.

---

### Q6. How do exit codes work? How do you propagate the right code from a pipeline or function?

- `0` = success, `1-255` = failure. `$?` holds the last command's status.
- `128+N` conventionally means killed by signal N (e.g. 137 = SIGKILL, 143 = SIGTERM).
- Pipelines: use `set -o pipefail` or inspect `"${PIPESTATUS[@]}"`.
- Functions: `return N` sets the function's status; the function's status defaults to that of its last command.
- Scripts: `exit "$rc"`. Capture `rc=$?` immediately, because the next command overwrites `$?`.

```bash
cmd1 | cmd2 | cmd3
echo "${PIPESTATUS[@]}"   # e.g. "0 1 0"
```

---

### Q7. What do `${var:-default}`, `${var:=default}`, `${var:?error}` and `${var%%pattern}` do?

- `${var:-default}`: use `default` if `var` is unset or empty (does not assign).
- `${var:=default}`: same, but also assigns it to `var`.
- `${var:?msg}`: if unset or empty, print `msg` to stderr and exit (non-interactive shell).
- `${var%pattern}` / `${var%%pattern}`: remove the shortest / longest matching suffix.
- `${var#pattern}` / `${var##pattern}`: remove the shortest / longest matching prefix.

```bash
f="/a/b/file.tar.gz"
echo "${f##*/}"    # file.tar.gz  (basename)
echo "${f%%.*}"    # /a/b/file    (strip all extensions)
echo "${f%.*}"     # /a/b/file.tar
```

---

## 2. Process Control and Error Handling

### Q8. How do you use `trap` for cleanup? Which signals would you trap in a deployment script?

```bash
tmpdir=$(mktemp -d)
cleanup() { rm -rf -- "$tmpdir"; }
trap cleanup EXIT
trap 'echo "Error on line $LINENO" >&2' ERR
trap 'echo "Interrupted" >&2; exit 130' INT TERM
```

- `EXIT`: runs on any exit; best place for cleanup.
- `ERR`: runs when a command fails (use with `set -E` so it's inherited by functions).
- `INT` / `TERM`: Ctrl-C and `kill`; convert to a clean exit so `EXIT` runs.
- `SIGKILL` cannot be trapped.
- Keep the trap handler idempotent and fast.

---

### Q9. How do you run tasks in parallel and wait for all of them, while capturing each exit status?

```bash
pids=()
for host in "${hosts[@]}"; do
  deploy "$host" &
  pids+=($!)
done

fail=0
for pid in "${pids[@]}"; do
  wait "$pid" || fail=1
done
exit "$fail"
```

Alternatives: `xargs -P N`, GNU `parallel`, and `wait -n` (Bash 4.3+) to react to the first finished job. Mention limiting concurrency to avoid overloading targets.

---

### Q10. Difference between a subshell `( )` and a command group `{ }`? Why does a variable set in `cmd | while read` disappear?

- `( ... )`: runs in a **subshell** (a forked child). Variable changes and `cd` don't affect the parent.
- `{ ...; }`: runs in the **current shell**. Needs a space after `{` and a `;` or newline before `}`.
- Each side of a pipeline runs in a subshell (by default), so the `while` loop's variables are lost.

**Fixes:**
```bash
# Process substitution
while read -r line; do count=$((count+1)); done < <(cmd)

# Or lastpipe (non-interactive Bash 4.2+)
shopt -s lastpipe
cmd | while read -r line; do count=$((count+1)); done
```

---

### Q11. How do you prevent two instances of the same script running at once?

Use `flock`, which is atomic and released automatically when the process dies:

```bash
exec 200>/var/lock/myscript.lock
flock -n 200 || { echo "Already running" >&2; exit 1; }
```

Weak approaches: PID files (race conditions between check and write, stale PIDs, PID reuse) and `[ -f lockfile ]` checks (not atomic). If `flock` isn't available, `mkdir lockdir` is atomic.

---

### Q12. How do you implement retries with exponential backoff?

```bash
retry() {
  local max=$1 delay=$2; shift 2
  local attempt=1
  until "$@"; do
    if (( attempt >= max )); then
      echo "Failed after $attempt attempts: $*" >&2
      return 1
    fi
    echo "Attempt $attempt failed; retrying in ${delay}s..." >&2
    sleep "$delay"
    delay=$(( delay * 2 ))
    attempt=$(( attempt + 1 ))
  done
}

retry 5 1 curl -fsS --max-time 5 https://example.com/health
```

Bonus: add jitter (`$((RANDOM % 3))`), only retry on retryable errors (5xx, timeouts), and set an overall deadline.

---

### Q13. What are zombie and orphan processes, and how can a script create them?

- **Zombie**: a child that has exited but whose parent hasn't called `wait()`, so its entry remains in the process table. Scripts create them by backgrounding jobs and never calling `wait` in a long-running parent.
- **Orphan**: a child whose parent has exited; it is adopted by PID 1 (`init`/`systemd`). Scripts create them by starting background jobs and exiting without killing or waiting for them.
- In containers, PID 1 that doesn't reap children causes zombie buildup; use `tini` or `--init`.
- Use `trap 'kill 0' EXIT` or `wait` to avoid leaving stray children.

---

## 3. Text Processing and Data Handling

### Q14. When would you choose `grep`, `sed`, `awk`, `cut` or `jq`?

| Tool | Best for | Example |
|---|---|---|
| `grep` | Filtering lines by pattern | `grep -E 'ERROR\|WARN' app.log` |
| `sed` | Stream edits, substitutions | `sed -i 's/old/new/g' file` |
| `awk` | Column-based processing, aggregation | `awk '{sum+=$3} END {print sum}' data.txt` |
| `cut` | Simple fixed-delimiter field extraction | `cut -d: -f1 /etc/passwd` |
| `jq` | JSON parsing and transformation | `jq -r '.items[].metadata.name'` |

---

### Q15. How would you parse JSON or YAML? Why is `grep`/`sed` on JSON a bad idea?

- Use `jq` for JSON and `yq` for YAML.
- Regex can't handle nesting, escaping, whitespace or key-order differences, so it breaks silently when the format changes.

```bash
name=$(jq -r '.name // empty' <<< "$json")
kubectl get pods -o json | jq -r '.items[] | select(.status.phase!="Running") | .metadata.name'
```

Use `-r` for raw output, `//` for defaults, and `--arg` to pass variables safely instead of string interpolation.

---

### Q16. Explain process substitution `<( )` and here-documents / here-strings.

- **Process substitution**: `<(cmd)` exposes a command's output as a file path (e.g. `/dev/fd/63`).
  ```bash
  diff <(sort a.txt) <(sort b.txt)
  ```
- **Here-document**: multi-line input to a command.
  ```bash
  cat <<'EOF' > config.yml
  name: $NOT_EXPANDED
  EOF
  ```
  Quoting the delimiter (`'EOF'`) disables expansion.
- **Here-string**: `grep foo <<< "$var"` passes a string on stdin without needing `echo | `.

---

### Q17. How do you use arrays and associative arrays? What Bash version is required?

```bash
# Indexed
arr=(one "two words" three)
echo "${arr[1]}"        # two words
echo "${#arr[@]}"       # 3
for x in "${arr[@]}"; do echo "$x"; done

# Associative (Bash 4.0+)
declare -A ports=([http]=80 [https]=443)
echo "${ports[https]}"
for k in "${!ports[@]}"; do echo "$k=${ports[$k]}"; done
```

Portability point: macOS ships Bash 3.2 (no associative arrays, no `mapfile`, no `${var^^}`). Mention `brew install bash` or avoiding those features if portability matters.

---

### Q18. How do you read a file line by line without losing whitespace or backslashes?

```bash
while IFS= read -r line; do
  printf '%s\n' "$line"
done < file.txt
```

- `IFS=` prevents trimming leading/trailing whitespace.
- `-r` prevents backslash interpretation.
- If the last line has no trailing newline: `|| [[ -n $line ]]` in the loop condition.
- Beware that commands inside the loop reading stdin (e.g. `ssh`) will consume the input; use `ssh -n` or a separate file descriptor (`read -u 3 ... 3< file`).

---

## 4. Real-World DevOps Scenarios

### Q19. Write a script that health-checks a list of URLs and exits non-zero if any fail, with timeout and retries.

```bash
#!/usr/bin/env bash
set -uo pipefail

urls=("$@")
(( ${#urls[@]} )) || { echo "Usage: $0 url..." >&2; exit 2; }

failed=0
for url in "${urls[@]}"; do
  if curl -fsS -o /dev/null --max-time 5 --retry 3 --retry-delay 2 "$url"; then
    echo "OK    $url"
  else
    echo "FAIL  $url" >&2
    failed=$((failed + 1))
  fi
done

(( failed == 0 ))
```

Look for: `-f` (fail on HTTP errors), timeouts, not exiting on first failure, a summary, and the right exit code. Bonus: parallelise with `xargs -P`.

---

### Q20. Write a script to delete logs older than N days and report the space reclaimed.

```bash
#!/usr/bin/env bash
set -euo pipefail

dir=${1:?Usage: $0 DIR DAYS}
days=${2:?Usage: $0 DIR DAYS}

[[ -d $dir ]] || { echo "Not a directory: $dir" >&2; exit 1; }

total=0
while IFS= read -r -d '' f; do
  size=$(stat -c %s -- "$f")      # GNU stat; use -f %z on BSD/macOS
  rm -- "$f"
  total=$((total + size))
done < <(find "$dir" -type f -name '*.log' -mtime +"$days" -print0)

echo "Reclaimed $(( total / 1024 / 1024 )) MiB"
```

Look for: null-safe iteration, validating inputs, a dry-run option (bonus), and avoiding `rm -rf` on unvalidated paths.

---

### Q21. How would you write a safe deployment script that supports rollback?

Key design points:
- **Immutable releases** in timestamped directories plus a `current` symlink.
- **Atomic switch**: `ln -sfn releases/$ts current.tmp && mv -T current.tmp current`.
- Health check after the switch; on failure, re-point the symlink to the previous release.
- Keep the last N releases for rollback; prune the rest.
- `trap ERR` to trigger rollback automatically.
- Locking (`flock`) to prevent concurrent deploys.
- Idempotency, dry-run mode, logging, and a clear exit code.
- Handle database migrations separately (backward-compatible migrations).

```bash
deploy() {
  local new=$RELEASES/$(date +%Y%m%d%H%M%S)
  prev=$(readlink -f "$CURRENT" || true)
  mkdir -p "$new" && tar -xzf "$ARTIFACT" -C "$new"
  ln -sfn "$new" "$CURRENT.tmp" && mv -T "$CURRENT.tmp" "$CURRENT"
  systemctl reload app
  health_check || { rollback "$prev"; return 1; }
}
```

---

### Q22. How do you manage secrets in Bash scripts? What are the risks of arguments or env vars?

**Risks:**
- Command-line args are visible via `ps` and `/proc/<pid>/cmdline`.
- Env vars are visible via `/proc/<pid>/environ` and inherited by all child processes; they can leak in crash dumps and logs.
- Secrets in shell history, in the script itself, or in git.
- `set -x` prints expanded secrets to the log.

**Better practices:**
- Fetch from a secrets manager (Vault, AWS Secrets Manager, SSM) at runtime.
- Read from files with restrictive permissions (`chmod 600`) or from stdin: `curl -H @<(echo "Authorization: ...")`.
- Use `--password-stdin` style options (e.g. `docker login --password-stdin`).
- Disable tracing around sensitive sections: `{ set +x; } 2>/dev/null`.
- `unset` secrets when done; mask in CI logs; rotate regularly.

---

### Q23. One-liner: top 10 IPs by request count in an nginx access log.

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
```

Or entirely in awk:
```bash
awk '{c[$1]++} END {for (ip in c) print c[ip], ip}' access.log | sort -rn | head -10
```

---

### Q24. How do you call AWS CLI, `kubectl` or Docker from a script and handle pagination, throttling and partial failures?

- **Pagination**: AWS CLI auto-paginates by default (control with `--max-items`/`--page-size`); for raw APIs follow `NextToken`/`continue` tokens in a loop. `kubectl get --chunk-size`.
- **Throttling**: retry with exponential backoff and jitter (see Q12); set `AWS_RETRY_MODE=adaptive` and `AWS_MAX_ATTEMPTS`.
- **Partial failures**: batch APIs can return success with a `Failed`/`UnprocessedItems` list, so check the payload with `jq`, not just the exit code.
- Use `--output json` and `jq`, never parse human-readable tables.
- Use `--query` (JMESPath) to reduce output; use `--dry-run` where available.
- Log which items failed and exit non-zero at the end, rather than aborting on the first item.

```bash
aws ec2 describe-instances --query 'Reservations[].Instances[].InstanceId' --output text
kubectl get pods -A -o json | jq -r '.items[] | select(.status.phase=="Failed") | "\(.metadata.namespace)/\(.metadata.name)"'
```

---

### Q25. A cron job works manually but fails under cron. What do you check?

1. **PATH**: cron uses a minimal PATH. Set it in the crontab/script or use absolute paths.
2. **Environment variables**: no profile/bashrc loaded; define everything needed.
3. **Working directory**: it's usually `$HOME`; use absolute paths or `cd` explicitly.
4. **Shell**: cron uses `/bin/sh` by default; set `SHELL=/bin/bash` or fix Bashisms.
5. **Permissions**: user, exec bit, access to files and sockets (e.g. no SSH agent, no TTY).
6. **Output**: redirect `>> /var/log/job.log 2>&1`; check `MAILTO`, syslog (`journalctl -u cron`).
7. **Syntax**: `%` must be escaped as `\%` in crontab lines.
8. **Overlap and timezone**: use `flock`; check the server timezone.
9. Reproduce with `env -i /bin/sh -c '/path/to/script'`.

---

## 5. Performance and Scalability

### Q26. A script processing a 5 GB log file is very slow. How do you diagnose and speed it up?

- **Diagnose**: `time`, `bash -x` with a timestamped `PS4`, `strace -c -f`, `perf`, or check if it's CPU- or I/O-bound.
- **Main culprit**: Bash `while read` loops spawning subprocesses per line (`echo | awk`, `$(...)`, `cut`, `sed`).
- **Fixes**:
  - Process the file in a single pass with `awk`, `grep`, or `sed`.
  - Use Bash builtins and parameter expansion instead of external commands.
  - `LC_ALL=C` for faster `grep`/`sort` on ASCII data.
  - Parallelise: `split` + `xargs -P`, or GNU `parallel`.
  - Filter early (`grep` before `sort`); use `sort -S` / `-T` for large sorts.
  - Consider `rg`/`mawk`, or rewriting in Python/Go if logic is complex.

---

### Q27. What is the cost of `cat file | grep` versus `grep file`, and of command substitution in loops?

- `cat file | grep` spawns an extra process and copies data through a pipe, for no benefit ("useless use of cat"). Impact is small once, large in loops.
- `$(...)` forks a subshell and runs an external command; inside a loop of 100k iterations that's 100k forks.
- Prefer builtins (`[[ ]]`, `${var//x/y}`, `printf -v`, `read`) over `basename`, `dirname`, `sed`, `expr`.

---

### Q28. How do you profile a Bash script?

- `time ./script.sh` for overall timing.
- Timestamped trace:
  ```bash
  PS4='+ $(date "+%s.%N") ${BASH_SOURCE}:${LINENO}: '
  set -x
  ```
  (or use `$EPOCHREALTIME` in Bash 5 to avoid forking `date`).
- `bash -x script.sh` or `bash -xv`.
- `strace -c -f -o out.txt ./script.sh` to count syscalls and forks.
- `bashdb` for step debugging; `shellcheck` to catch inefficient patterns.

---

## 6. Quality, Style and Maintainability

### Q29. How do you test Bash scripts?

- **Static analysis**: `shellcheck` and `shfmt` in pre-commit hooks and CI.
- **Unit tests**: `bats-core` (with `bats-assert` and `bats-support`) or `shunit2`.
- **Mocking**: override commands with functions or place stub executables earlier in `PATH`.
- **Structure for testability**: put logic in functions and guard `main`:
  ```bash
  if [[ ${BASH_SOURCE[0]} == "$0" ]]; then main "$@"; fi
  ```
- Run integration tests in containers; use `--dry-run` modes.

---

### Q30. Common `shellcheck` warnings and why they matter?

| Code | Issue | Why it matters |
|---|---|---|
| SC2086 | Unquoted variable | Word splitting and globbing bugs |
| SC2046 | Unquoted command substitution | Same as above |
| SC2155 | `local x=$(cmd)` masks return value | Hides failures under `set -e` |
| SC2164 | `cd` without `|| exit` | Subsequent commands run in the wrong directory |
| SC2034 | Unused variable | Often a typo |
| SC2162 | `read` without `-r` | Backslashes mangled |
| SC2012 | Parsing `ls` | Breaks on unusual filenames |
| SC2181 | Checking `$?` indirectly | Fragile; test the command directly |

---

### Q31. How do you structure a large script?

```bash
#!/usr/bin/env bash
set -Eeuo pipefail
IFS=$'\n\t'

readonly SCRIPT_NAME=${0##*/}

usage() { cat <<EOF
Usage: $SCRIPT_NAME [-e env] [-n] [-h]
EOF
}

log()  { printf '%s [%s] %s\n' "$(date -u +%FT%TZ)" "$1" "${*:2}" >&2; }
die()  { log ERROR "$*"; exit 1; }

parse_args() {
  while getopts ":e:nh" opt; do
    case $opt in
      e) ENVIRONMENT=$OPTARG ;;
      n) DRY_RUN=1 ;;
      h) usage; exit 0 ;;
      *) usage; exit 2 ;;
    esac
  done
}

main() {
  parse_args "$@"
  # ...
}

main "$@"
```

Key points: `main()`, small functions, `local` variables, `readonly` constants, `usage()`, argument validation, consistent logging, no global-state sprawl, and `getopts` (short options) or a manual `while/case` loop for long options.

---

### Q32. Bash vs POSIX `sh` vs Python/Go: when should a script be rewritten?

**Bash is a good fit when** it's mostly gluing CLI tools, is short (< ~100-200 lines), and runs in a known environment.

**Rewrite when:**
- Complex data structures (nested JSON, maps of lists) or heavy string logic.
- You need robust error handling, retries, concurrency, or unit testing.
- Calling many HTTP APIs or using SDKs (boto3, Kubernetes client).
- Performance-critical or long-lived services.
- Multiple contributors need maintainable code.
- Cross-platform requirements (Windows, macOS).

**POSIX `sh`** for minimal containers (Alpine/BusyBox) and maximum portability; avoid Bashisms there.

---

### Q33. What is your approach to logging in scripts?

- Send logs and diagnostics to **stderr**; keep **stdout** for data so output can be piped.
- Use levels (DEBUG/INFO/WARN/ERROR) with a `LOG_LEVEL` variable.
- ISO-8601 UTC timestamps; include script name or PID.
- Optionally structured (JSON) output for log aggregators.
- Use `logger` to write to syslog/journald.
- Redirect everything to a file when needed: `exec > >(tee -a "$LOG") 2>&1`.
- Never log secrets.

---

## 7. Security

### Q34. What is command injection in shell scripts and how do you prevent it?

Untrusted input being interpreted as shell code, e.g. `eval "ls $user_input"` with input `; rm -rf /`.

**Prevention:**
- Quote all expansions; pass arguments as separate words, not concatenated strings.
- Avoid `eval`, `bash -c "$string"`, and `sh -c` built from user input.
- Validate input with allow-lists: `[[ $name =~ ^[a-zA-Z0-9._-]+$ ]]`.
- Use `--` before user-supplied arguments, so they can't be treated as options.
- Use arrays for building commands: `cmd=(ls -l -- "$dir"); "${cmd[@]}"`.
- Run with least privilege; be careful with `sudo` and `ssh host "$var"`.

---

### Q35. Dangers of `rm -rf "$VAR/"` when `$VAR` is empty? How do you guard against it?

With `VAR` empty it becomes `rm -rf /`. Guards:

```bash
set -u                                   # error on unset (not on empty!)
rm -rf -- "${VAR:?VAR must be set}/"     # error on unset OR empty
[[ -n $VAR && -d $VAR ]] || exit 1       # explicit validation
```

Also: validate the path is under an expected prefix, avoid building paths from untrusted input, prefer `mktemp -d` plus `trap`, and print or dry-run before deleting. (GNU `rm` has `--preserve-root`, which is on by default.)

---

### Q36. How do you safely create temp files?

```bash
tmpfile=$(mktemp) || exit 1
tmpdir=$(mktemp -d) || exit 1
trap 'rm -rf -- "$tmpfile" "$tmpdir"' EXIT
```

- `mktemp` creates files with unpredictable names and mode 600, atomically.
- Avoid predictable names like `/tmp/myscript.$$` or `/tmp/out.txt` (symlink attacks, collisions).
- Respect `$TMPDIR`; always clean up with `trap`.

---

### Q37. What is a TOCTOU race condition in a script context?

**Time-of-check to time-of-use**: the state changes between a check and the action based on it.

```bash
if [ ! -e /tmp/lock ]; then   # check
  touch /tmp/lock             # use: another process may have created it in between
fi
```

An attacker can also swap a file for a symlink between check and use.

**Fixes:** use atomic operations (`mkdir`, `flock`, `mktemp`, `set -o noclobber` with `>`, `ln -s`), use `mv` for atomic replace, and avoid world-writable directories.

---

## 8. Hands-On Live Exercise

**Task: review and fix this script.**

```bash
#!/bin/bash
for f in $(ls /var/log/*.log); do
  if [ $(wc -l $f) -gt 1000 ]; then
    cat $f | grep ERROR > /tmp/errors.txt
  fi
done
```

### Problems to identify

1. Parsing `ls` output; breaks on spaces/special characters (use a glob).
2. Unquoted variables: word splitting and globbing.
3. `wc -l $f` prints `1234 /path/file`, so the integer comparison fails. Use `wc -l < "$f"`.
4. Useless use of `cat`.
5. `>` overwrites `/tmp/errors.txt` each iteration, so only the last file's errors survive.
6. Predictable `/tmp` filename (race and symlink risk); use `mktemp`.
7. No `set -euo pipefail`; also `grep` returns 1 when nothing matches, which would kill the script under `set -e` unless handled.
8. `#!/bin/bash` vs `#!/usr/bin/env bash` (minor portability).

### Model solution

```bash
#!/usr/bin/env bash
set -euo pipefail

out=$(mktemp)
trap 'rm -f -- "$out"' EXIT

for f in /var/log/*.log; do
  [[ -f $f ]] || continue
  lines=$(wc -l < "$f")
  if (( lines > 1000 )); then
    grep -H 'ERROR' -- "$f" >> "$out" || true   # no match is not a failure
  fi
done

cat -- "$out"
```

---

## Evaluation Guide

| Level | Signs |
|---|---|
| **Strong senior** | Quotes by habit; uses `trap`, `mktemp`, `flock`; runs `shellcheck`; explains trade-offs; knows when *not* to use Bash; thinks about security, idempotency and failure modes unprompted. |
| **Mid-level** | Writes working scripts but misses edge cases (quoting, `pipefail`, cleanup, locking). |
| **Weak** | Copy-paste style; parses `ls`; ignores exit codes; no error handling; relies on `eval` or unquoted variables. |

### Suggested interview flow (60 minutes)

1. **10 min**: fundamentals (Q1-Q7)
2. **15 min**: process control and error handling (Q8-Q13)
3. **15 min**: pick 2-3 real-world scenarios (Q19-Q25)
4. **15 min**: live exercise (Section 8)
5. **5 min**: quality, security and candidate questions
