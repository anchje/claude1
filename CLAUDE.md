# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> claude1
> Agenten-Repo Claude Code

## Repository status

Contains one shipped project: **TermScan**, a browser-based barcode/QR scanner PWA
for Android. Full architecture, deployment, and Supabase schema docs live in
`README.md` — read that first for anything beyond the quick facts below.

Live: https://anchje.github.io/claude1/termscan-live.html

## Quick facts

- **No build/lint/test tooling.** Everything is plain static HTML/CSS/JS
  (`termscan-live.html`, `index.html`, `manifest.json`, `sw.js`, `icon.svg`). Edit
  files directly; there's no compile step.
- **Deployment is automatic**: GitHub Pages serves `main` at the repo root on every
  push — no CI/workflow file involved.
- **Backend is Supabase** (project `qiqjywcyqcrccgbocbdq`), table `public.scans`.
  Schema, RLS policies, and the security trade-offs of the open `anon` access are
  documented in `README.md`. The Supabase MCP tools available in this environment
  run with DDL blocked (read-only default) — schema changes need to go through the
  Supabase SQL Editor manually (see README for the exact SQL).
- **`TermScan-Scanner-standalone.html`** is the original design mockup (visual
  reference only, no working logic) — don't confuse it with `termscan-live.html`,
  which is the real app.
- **`termscan-2026-08.md`** documents the earlier local test setup (WSL2 + adb +
  Android Wireless Debugging), from before the app moved to GitHub Pages. Kept for
  historical reference; not needed for day-to-day work anymore.
- The repo is **public**. Treat anything committed here as world-readable — the
  Supabase key already in `termscan-live.html` is a public *publishable* key by
  design, not a secret; don't add anything that isn't meant to be public.
