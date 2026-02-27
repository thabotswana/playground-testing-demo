# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A WordPress plugin demo project for a WordCamp EU 2025 workshop on "Building Automated Tests with WordPress Playground." Tests run against a live WordPress instance spun up via WordPress Playground CLI.

## Setup

```bash
nvm use          # Node v20.18.3 (see .nvmrc)
npm ci
npx playwright install --with-deps
```

## Commands

```bash
# Run all tests
npm test

# Integration tests only (Vitest, watch mode)
npm run test:integration

# E2E tests with Playwright UI
npm run test:e2e

# E2E tests headless (as in CI)
npx playwright test --config=tests/end-to-end/playwright.config.ts
```

## Architecture

**WordPress Plugin** (`playground-testing-demo.php` → `lib/`):
- `lib/admin.php` — Registers an admin menu page at `?page=workshop-tests`, renders a message list and form, includes inline JS that POSTs to the REST API
- `lib/api.php` — Registers `POST /wp-json/PTD/v1/message` (requires admin auth), sanitizes input, persists messages to WordPress options under key `PTD_messages`

**Test Suites** (`tests/`):
- `tests/integration/` — Vitest-based tests; exercise plugin behavior via PHP/HTTP using `fetch-cookie` for authenticated sessions against a Playground-hosted WordPress instance
- `tests/end-to-end/` — Playwright-based browser tests; drive the admin UI at `?page=workshop-tests`

Both test suites target a WordPress site started locally by WordPress Playground CLI (`@wp-playground/cli`). The Playground instance URL must be configured in the respective test config files before running.

**CI** (`.github/workflows/ci.yml`): Runs on every push — installs deps, runs integration tests, installs Playwright browsers, runs E2E tests headless (1 worker, 2 retries).
