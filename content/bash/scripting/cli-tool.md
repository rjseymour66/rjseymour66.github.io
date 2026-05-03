---
title: "Writing a CLI tool"
weight: 65
description: >
  Script layout, usage functions, flag parsing, subcommand dispatch, and input validation for production-ready CLI tools.
---

A CLI tool is a bash script with a defined interface: it accepts flags and arguments, validates input, produces clear output, and returns meaningful exit codes. The sections below show how to build one from the ground up.

## Script template

Every CLI tool starts with the same skeleton. Put all logic inside functions and call `main "$@"` at the bottom so the script is safe to source in tests:

```bash
#!/usr/bin/env bash
set -euo pipefail

# --- Constants ---
readonly SCRIPT_NAME=$(basename "$0")
readonly VERSION="1.0.0"

# --- Functions ---
usage() { ... }
# ... other functions ...

main() { ... }

main "$@"
```

`set -euo pipefail` catches three of the most common silent failures: a command returning non-zero, a reference to an unset variable, and a failure buried inside a pipeline. See [Execution flags](../structure#execution-flags) for details.

## usage function

The `usage` function documents the script's interface. Call it when the user passes `-h` or supplies invalid input:

```bash
usage() {
    cat <<EOF
Usage: $SCRIPT_NAME [OPTIONS] <command>

Commands:
  start     Start the service
  stop      Stop the service
  status    Show service status

Options:
  -e <env>  Target environment (default: development)
  -v        Verbose output
  -h        Show this help
EOF
}
```

Print usage to stdout when the user requests help, and to stderr when triggered by an error. Return exit code `0` for `-h` and `2` for invalid input:

```bash
if [[ $# -lt 1 ]]; then
    echo "$SCRIPT_NAME: command required" >&2
    usage >&2
    exit 2
fi
```

## Parsing flags with getopts

`getopts` is a bash built-in that parses short flags (`-v`, `-e production`) one at a time from the argument list. Each call to `getopts` reads the next flag and stores it in a variable you name — conventionally `opt`. You call it in a `while` loop so it processes every flag in sequence.

**The option string**

The option string (`'e:vh'`) lists every flag the script accepts. A colon after a letter means that flag requires an argument. In `'e:vh'`, `-e` requires an argument, `-v` and `-h` do not:

| Character | Meaning                             |
| :-------- | :---------------------------------- |
| `e:`      | `-e` accepts an argument            |
| `v`       | `-v` is a boolean flag, no argument |
| `h`       | `-h` is a boolean flag, no argument |

**Special variables**

| Variable | Set by  | Contains                                        |
| :------- | :------ | :---------------------------------------------- |
| `opt`    | you     | The current flag letter (`e`, `v`, `h`, or `?`) |
| `OPTARG` | getopts | The argument passed to a flag that requires one  |
| `OPTIND` | getopts | The index of the next unprocessed argument       |

When `getopts` encounters an unknown flag, it sets `opt` to `?`. The `*` pattern in `case` catches this and triggers the usage message.

**`shift $(( OPTIND - 1 ))`**

After the loop, `OPTIND` points to the first argument that was not a flag. `shift` removes all the parsed flags from `$@` so `$1` becomes the first positional argument — typically the subcommand.

Define accepted flags in the option string; append `:` to any flag that takes an argument:

```bash
VERBOSE=0
ENVIRONMENT="development"

while getopts 'e:vh' opt; do
    case "$opt" in
        e) ENVIRONMENT="$OPTARG" ;;
        v) VERBOSE=1 ;;
        h) usage; exit 0 ;;
        *) usage >&2; exit 2 ;;
    esac
done

shift $(( OPTIND - 1 ))    # remove parsed flags; $1 is now the first non-flag argument
```

After the loop, `shift $(( OPTIND - 1 ))` removes the flags from the argument list so `$1` becomes the first positional argument. This is typically the subcommand or target.

## Subcommand dispatch

Use `case` to route subcommands. Pass `$1` as the EXPRESSION and match each subcommand as a PATTERN:

```bash
readonly COMMAND=${1:-}

case "$COMMAND" in
    start)
        start_service
        ;;
    stop)
        stop_service
        ;;
    status)
        show_status
        ;;
    help|-h|--help)
        usage
        exit 0
        ;;
    "")
        echo "$SCRIPT_NAME: command required" >&2
        usage >&2
        exit 2
        ;;
    *)
        echo "$SCRIPT_NAME: unknown command: $COMMAND" >&2
        usage >&2
        exit 2
        ;;
esac
```

The empty string pattern `""` catches the case where the user runs the script with no arguments after flags are stripped.

## Validating input

Check all required arguments and preconditions at the top of `main`, before calling any other function. This fails fast with a clear error rather than a cryptic failure deep in the script:

```bash
main() {
    local source=${1:-}
    local dest=${2:-}

    [[ -z "$source" ]] && {
        echo "$SCRIPT_NAME: source required" >&2
        usage >&2
        exit 2
    }

    [[ ! -f "$source" ]] && {
        echo "$SCRIPT_NAME: file not found: $source" >&2
        exit 1
    }

    [[ ! -w "$dest" ]] && {
        echo "$SCRIPT_NAME: not writable: $dest" >&2
        exit 1
    }
}
```

## Complete example

A tool that manages a service with `start`, `stop`, and `status` subcommands and an optional `-e` environment flag:

```bash
#!/usr/bin/env bash
set -euo pipefail

# --- Constants ---
readonly SCRIPT_NAME=$(basename "$0")
readonly DEFAULT_ENV="development"

# --- Functions ---
usage() {
    cat <<EOF
Usage: $SCRIPT_NAME [-e <env>] [-v] <command>

Commands:
  start     Start the service
  stop      Stop the service
  status    Show service status

Options:
  -e <env>  Target environment (default: $DEFAULT_ENV)
  -v        Verbose output
  -h        Show this help
EOF
}

log() {
    [[ "$VERBOSE" -eq 1 ]] && echo "[$SCRIPT_NAME] $*"
    return 0
}

start_service() {
    log "Starting service in $ENVIRONMENT"
    systemctl start myapp
    echo "Service started"
}

stop_service() {
    log "Stopping service in $ENVIRONMENT"
    systemctl stop myapp
    echo "Service stopped"
}

show_status() {
    systemctl status myapp
}

main() {
    VERBOSE=0
    ENVIRONMENT="$DEFAULT_ENV"

    while getopts 'e:vh' opt; do
        case "$opt" in
            e) ENVIRONMENT="$OPTARG" ;;
            v) VERBOSE=1 ;;
            h) usage; exit 0 ;;
            *) usage >&2; exit 2 ;;
        esac
    done
    shift $(( OPTIND - 1 ))

    local command=${1:-}

    case "$command" in
        start)  start_service ;;
        stop)   stop_service ;;
        status) show_status ;;
        "")
            echo "$SCRIPT_NAME: command required" >&2
            usage >&2
            exit 2
            ;;
        *)
            echo "$SCRIPT_NAME: unknown command: $command" >&2
            usage >&2
            exit 2
            ;;
    esac
}

main "$@"
```
