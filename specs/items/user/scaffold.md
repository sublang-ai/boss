<!-- SPDX-License-Identifier: Apache-2.0 -->
<!-- SPDX-FileCopyrightText: 2026 SubLang International <https://sublang.ai> -->

# SCAF: User-Facing Scaffold Behavior

## Intent

This spec defines user-visible behavior of the `boss scaffold`
command.

## Target Resolution

### SCAF-1

Where a user runs `boss scaffold <path>`, the CLI shall create the
specs directory structure inside the specified path. The path must
exist and be a directory; otherwise the CLI shall exit non-zero.

### SCAF-2

Where a user runs `boss scaffold` without a path argument inside a
git repository, the CLI shall create the specs directory structure
at the repository root.

### SCAF-3

Where a user runs `boss scaffold` without a path argument outside
any git repository, the CLI shall create the specs directory
structure in the current working directory.

## Idempotency

### SCAF-4

Where a user runs `boss scaffold` and target directories or
template files already exist, the CLI shall skip those entries with
an `(already exists)` indicator, leaving existing content
unmodified.

## Agent Instructions

### SCAF-5

Where a user runs `boss scaffold`, the CLI shall update agent spec
instructions in `CLAUDE.md` and `AGENTS.md`. When neither file
exists, the CLI shall create both; when only one exists, only that
file shall be updated. When a file contains a matching specs
section heading, the CLI shall replace that section in place; when
the replacement is identical, the CLI shall skip the file.

## Error Handling

### SCAF-6

Where `boss scaffold` encounters an unrecoverable error, the CLI
shall print an error message to stderr and exit non-zero.
