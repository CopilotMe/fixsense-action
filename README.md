# FixSense — AI Test Failure Analysis

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-FixSense-green?logo=github)](https://github.com/marketplace/actions/fixsense-ai-test-failure-analysis)
[![Playwright](https://img.shields.io/badge/Playwright-supported-brightgreen?logo=playwright)](https://playwright.dev)
[![Cypress](https://img.shields.io/badge/Cypress-supported-brightgreen?logo=cypress)](https://cypress.io)
[![Jest](https://img.shields.io/badge/Jest-supported-brightgreen?logo=jest)](https://jestjs.io)
[![pytest](https://img.shields.io/badge/pytest-supported-brightgreen?logo=python)](https://pytest.org)

**Stop debugging CI failures manually.** FixSense analyzes your failing tests with AI, explains the root cause, classifies app bugs vs test bugs, scores flakiness, and suggests fixes — all as a PR comment.

> Works with any test framework that produces JUnit XML reports.

## What You Get

## Screenshots

### Analysis Card
![FixSense Analysis Card](docs/analysis-card.png)

### Dashboard
![FixSense Dashboard](docs/dashboard-stats.png)


For every failing test, FixSense provides:

| Feature | Description |
|---------|-------------|
| **Root Cause** | AI explains *why* your test failed in 2-4 sentences |
| **App Bug vs Test Bug** | Is your application broken or just the test? |
| **Flakiness Score** | 0-100 score — high means likely flaky, not a real regression |
| **Suggested Fix** | Actionable steps to resolve the failure |
| **PR Comment** | All results posted as a single, auto-updating PR comment |

## Quick Start

Add to your workflow **after** your test step:

```yaml
- name: Run tests
  id: tests
  run: npx playwright test
  continue-on-error: true

- name: FixSense Analysis
  if: steps.tests.outcome == 'failure'
  uses: CopilotMe/fixsense-action@v1
  with:
    api-key: ${{ secrets.FIXSENSE_API_KEY }}
```

That's it. Next time a test fails, you'll get an analysis comment on your PR.

## Full Workflow Example

```yaml
name: Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - run: npm ci
      - run: npx playwright install --with-deps

      - name: Run Playwright tests
        id: tests
        run: npx playwright test
        continue-on-error: true

      - name: FixSense Analysis
        if: steps.tests.outcome == 'failure'
        uses: CopilotMe/fixsense-action@v1
        with:
          api-key: ${{ secrets.FIXSENSE_API_KEY }}
          results-path: test-results/

      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results
          path: test-results/
```

## PR Comment Preview

The action posts a summary table with expandable details for each failure:

| Test | Root Cause | Flakiness | Confidence |
|------|-----------|-----------|------------|
| `login.spec.ts > should login` | Timeout waiting for selector `.submit-btn` — element no longer exists after recent refactor | 🟡 65 | 🟢 high |
| `checkout.spec.ts > payment` | HTTP 500 from /api/payment — server error, not a test issue | 🔴 10 | 🟢 high |

<details><summary>Expanded details include:</summary>

- Full root cause explanation
- Suggested fix with code pointers
- Related code change (diff hint)
- Whether it's an app bug or test bug

</details>

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `api-key` | **Yes** | — | Your FixSense API key ([get one free](https://fix-sense.vercel.app/dashboard/settings)) |
| `results-path` | No | `test-results/` | Path to JUnit XML test results |
| `post-comment` | No | `true` | Post analysis as PR comment |
| `api-url` | No | `https://fix-sense.vercel.app` | API URL (for self-hosted) |

## Outputs

| Output | Description |
|--------|-------------|
| `analyses` | JSON array of analysis results |
| `failure-count` | Number of failures analyzed |

## Supported Test Frameworks

Any framework that produces **JUnit XML** reports:

| Framework | Reporter Config |
|-----------|----------------|
| **Playwright** | `reporter: [['junit', { outputFile: 'test-results/junit.xml' }]]` |
| **Cypress** | `cypress-junit-reporter` |
| **Jest** | `jest-junit` |
| **pytest** | `--junitxml=test-results/junit.xml` |
| **JUnit / TestNG** | Native XML output |
| **Mocha** | `mocha-junit-reporter` |
| **RSpec** | `rspec_junit_formatter` |
| **Go** | `go-junit-report` |
| **WebdriverIO** | `@wdio/junit-reporter` — `reporters: [['junit', { outputDir: './test-results' }]]` |
| **Dart / Flutter** | `dart test --machine \| tojunit` ([junitreport](https://pub.dev/packages/junitreport)) |

## Get Your API Key

1. Go to [fix-sense.vercel.app](https://fix-sense.vercel.app)
2. Sign in with GitHub or Google
3. Go to **Settings** and generate your API key
4. Add it as a repository secret named `FIXSENSE_API_KEY`

Free plan includes **350 analyses/month** — enough for most small-to-mid teams.

## Why FixSense?

- **Saves 10-30 min per failure** — no more scrolling through logs
- **Catches flaky tests** — high flakiness score = don't block the PR
- **Distinguishes real bugs** — HTTP 5xx, missing elements, broken flows flagged as app bugs
- **Works with any CI** — GitHub Actions, self-hosted runners, any JUnit XML producer
- **Dashboard included** — track failure trends, top flaky tests, auto-fix success rates at [fix-sense.vercel.app](https://fix-sense.vercel.app)

## Feedback & Issues

Found a bug, have a feature idea, or want to share feedback? Open an issue directly in this repo:

- **Bug?** → [Open a bug report](https://github.com/CopilotMe/fixsense-action/issues/new?template=bug_report.md)
- **Feature idea?** → [Request a feature](https://github.com/CopilotMe/fixsense-action/issues/new?template=feature_request.md)
- **General feedback?** → [Start a discussion](https://github.com/CopilotMe/fixsense-action/issues/new)

## Links

- [FixSense Dashboard](https://fix-sense.vercel.app/dashboard)
- [Documentation](https://fix-sense.vercel.app/docs)
- [GitHub Marketplace](https://github.com/marketplace/fixsense-ai-ci-failure-analysis-auto-fix)

## License

MIT
