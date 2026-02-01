# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a static documentation repository containing technical cheat sheets and reference guides. There is no build system, tests, or deployable code - just plain text (.txt), Markdown (.md), and spreadsheet (.xlsx) files.

## Structure

- **howtos/** - Quick reference guides organized by topic:
  - `cplus/` - C++ development (modern C++, smart pointers, threads, design patterns)
  - `linux/` - Linux/Ubuntu administration and debugging (bash, gdb, lldb)
  - `ml/` - Machine learning and LLM resources
  - `general/` - General development tools (git, python, docker, vscode)
  - `llvm/` - LLVM compiler toolchain
  - Other subdirectories for specialized topics (aix, alpine, valgrind, windows, hw)

- **wiki/** - Longer-form technical articles (coding conventions, toolchain guides, architecture docs)

## Conventions

- Most documentation uses plain `.txt` format with informal structure
- Wiki articles use Markdown (`.md`) for more formal documentation
- The `ml/useful_links.txt` file is actively maintained with links to LLM/AI research papers
- C++ coding standards are documented in `wiki/C++_coding_conventions.md`
