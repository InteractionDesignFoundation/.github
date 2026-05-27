# Renovate

Shared [Renovate](https://github.com/renovatebot/renovate) presets for InteractionDesignFoundation repositories. Distributed as [shareable presets](https://docs.renovatebot.com/config-presets/).

Three configs:

| File | Use case |
| --- | --- |
| `renovate-config.json` | Default. Security critical. Frequent updates, OSV alerts, pinned digests, weekly lockfile maintenance, runs outside office hours. |
| `renovate-config-slow-updates.json` | Low priority repos. Monthly schedule, everything grouped into one PR, manual merge. |
| `renovate-config-security-updates-only.json` | Frozen repos. Only security and PHP runtime updates run. |

## Onboarding

Mend Renovate app opens a `Configure Renovate` PR. Replace its body with one of:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["local>InteractionDesignFoundation/.github:renovate-config"]
}
```

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["local>InteractionDesignFoundation/.github:renovate-config-slow-updates"]
}
```

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["local>InteractionDesignFoundation/.github:renovate-config-security-updates-only"]
}
```

## Repo requirements

1. CI runs on `push` to branches matching `renovate/*`.
2. Composer lockfile generated with Composer >= 2.2.
3. PHP version set in `composer.json` at [`config.platform.php`](https://getcomposer.org/doc/06-config.md#platform).

## Default config (`renovate-config.json`)

Built on Renovate's [`config:best-practices`](https://docs.renovatebot.com/presets-config/#configbest-practices) preset (recommended for advanced users). It pulls in:

* `config:recommended` (dependency dashboard, monorepo grouping, ignore tests, changelog helpers, `:semanticPrefixFixDepsChoreOthers`).
* `docker:pinDigests`, `helpers:pinGitHubActionDigests` (pin Docker images and GitHub Actions to SHAs).
* `:pinDevDependencies` (pin dev deps for reproducible builds).
* `:configMigration` (auto PR when config options get deprecated).
* `abandonments:recommended` (flag abandoned packages).
* `security:minimumReleaseAgeNpm` (3 day wait on npm, malware window).
* `:maintainLockFilesWeekly` (refresh lockfile weekly).

Additional presets on top:

| Preset | Effect |
| --- | --- |
| `group:allNonMajor` | One PR per scheduled run for all non major updates. |
| `:separateMultipleMajorReleases` | One PR per intermediate major version (e.g. v1 to v2, v2 to v3 separately). |
| `:combinePatchMinorReleases` | Patch and minor for the same package combined. |
| `:automergeMinor` | Non major automerged once tests pass. |
| `:automergeBranch` | Automerge type is branch (PR only opens on test failure). |
| `:rebaseStalePrs` | Stale PRs rebased automatically. |
| `:enableVulnerabilityAlerts` | Open PRs for GitHub Vulnerability Alerts. |
| `:timezone(UTC)` | Schedules use UTC. |
| `:gitSignOff` | Sign off commits (DCO). |
| `:label(dependencies)` | Label PRs with `dependencies`. |
| `schedule:nonOfficeHours` | Runs Mon to Fri 22:00 to 05:00 UTC and anytime on weekends. Vulnerability alerts ignore this and run any time. |

Semantic commit prefixes (`fix(deps):` for prod, `chore(deps-dev):` for dev, `chore(deps):` for everything else) inherited from `config:recommended` via `:semanticPrefixFixDepsChoreOthers`. Watch out for any IxDF library repo using `semantic-release` on `main`: prod dep bumps will trigger automatic patch releases. Override per repo with `":semanticCommitsDisabled"` if unwanted.

Top level options:

| Option | Value | Why |
| --- | --- | --- |
| `osvVulnerabilityAlerts` | `true` | OSV database alerts for direct deps. Catches malicious packages. |
| `commitBodyTable` | `true` | Update table in commit body. |
| `platformAutomerge` | `true` | Use GitHub native merge, fall back to Renovate. |
| `prBodyTemplate` | custom Handlebars template | Renders the default sections (header, table, warnings, notes, changelogs, configDescription, controls) and adds an IxDF docs link plus a security review reminder near the top of every PR. Replaces the old `prFooter`. |
| `rangeStrategy` | `"replace"` | PR only when new version falls outside `composer.json` constraint. |
| `rollbackPrs` | `true` | If a package is revoked, downgrade PR opens. |
| `vulnerabilityAlerts.rangeStrategy` | `"update-lockfile"` | Patch lockfile only, ship security fix fast, no manifest churn. |
| `vulnerabilityAlerts.extends` | manual review presets | Security PRs require human review and carry security labels. |

### Vulnerability alerts setup

Required on each consuming repo:

1. Enable **Dependency graph** and **Dependabot alerts** under Settings, Security and analysis.
2. Grant the Renovate app read access to **Vulnerability alerts** in app permissions.
3. From then on Renovate raises fix PRs when GitHub reports vulnerabilities.

Details: [renovatebot docs](https://docs.renovatebot.com/configuration-options/#vulnerabilityalerts).

## Slow updates config (`renovate-config-slow-updates.json`)

For repos with rare updates and lower security stakes. Extends the default preset, then overrides:

| Option | Value | Why |
| --- | --- | --- |
| `extends: schedule:monthly` | first of month, before 04:00 UTC | Overrides the default's `schedule:nonOfficeHours`. One run per month. |
| `extends: :maintainLockFilesMonthly` | monthly lockfile refresh | Less churn than the default weekly. |
| `minimumReleaseAge` | `"21 days"` | Wait 3 weeks before flagging any update. Extra stability. |
| `prConcurrentLimit` | `3` | Cap open Renovate PRs. |
| `prHourlyLimit` | `2` | Throttle PR creation. |
| `separateMajorMinor` / `separateMultipleMajor` / `separateMinorPatch` | `false` | Merge all update types into one PR. |
| `packageRules` | groupName `all dependencies`, `automerge: false` | One monthly PR with manual review. |

Vulnerability alerts from the default preset still apply. Critical security PRs are not gated by the monthly schedule.

## Security only config (`renovate-config-security-updates-only.json`)

For frozen repos. Extends the default, disables lockfile maintenance, then via `packageRules` disables every package and re enables PHP runtime.

| Rule | Effect |
| --- | --- |
| `matchPackageNames: ["*"], enabled: false` | No normal updates. |
| `matchPackageNames: ["php"], enabled: true` | PHP platform version stays current. |
| Inherited `vulnerabilityAlerts` | Security PRs still raised. |

## Per-repo overrides

The shared preset is a baseline. Each repo's `renovate.json` can override any option by adding it after the `extends` entry. Examples:

### Disable a single package

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["local>InteractionDesignFoundation/.github:renovate-config"],
  "packageRules": [
    {
      "description": "Pinned to v6: v7 breaks our Pest config.",
      "matchPackageNames": ["brianium/paratest"],
      "enabled": false
    }
  ]
}
```

### Require Dependency Dashboard approval before opening a PR

```json
{
  "extends": ["local>InteractionDesignFoundation/.github:renovate-config"],
  "packageRules": [
    {
      "description": "Major updates wait for explicit approval in the Dependency Dashboard issue.",
      "matchUpdateTypes": ["major"],
      "dependencyDashboardApproval": true
    }
  ]
}
```

### Change schedule for one repo

```json
{
  "extends": ["local>InteractionDesignFoundation/.github:renovate-config"],
  "schedule": ["before 6am on monday"]
}
```

Renovate cron syntax: see [docs.renovatebot.com/key-concepts/scheduling](https://docs.renovatebot.com/key-concepts/scheduling/).

### Tighten the malware window (composer)

```json
{
  "extends": ["local>InteractionDesignFoundation/.github:renovate-config"],
  "packageRules": [
    {
      "description": "Wait 3 days after composer release before raising the PR.",
      "matchManagers": ["composer"],
      "minimumReleaseAge": "3 days"
    }
  ]
}
```

### Disable automerge for production deps, keep it for dev deps

```json
{
  "extends": ["local>InteractionDesignFoundation/.github:renovate-config"],
  "packageRules": [
    {
      "description": "Manual merge for production composer requires.",
      "matchDepTypes": ["require"],
      "automerge": false
    }
  ]
}
```

### Pin GitHub Actions but exclude one

```json
{
  "extends": ["local>InteractionDesignFoundation/.github:renovate-config"],
  "packageRules": [
    {
      "description": "Internal action repo: stick with the float tag, no SHA pinning.",
      "matchManagers": ["github-actions"],
      "matchPackageNames": ["InteractionDesignFoundation/internal-action"],
      "pinDigests": false
    }
  ]
}
```

### Auto assign reviewers on every PR

```json
{
  "extends": ["local>InteractionDesignFoundation/.github:renovate-config"],
  "reviewers": ["team:platform"]
}
```

### Restrict managers (allowlist)

```json
{
  "extends": ["local>InteractionDesignFoundation/.github:renovate-config"],
  "enabledManagers": ["composer", "npm", "github-actions", "dockerfile", "docker-compose", "nvm"]
}
```

### Opt one repo back into office-hours schedule

```json
{
  "extends": ["local>InteractionDesignFoundation/.github:renovate-config"],
  "schedule": ["after 9am and before 5pm every weekday"]
}
```

Validate any local override before pushing:

```bash
npx --yes --package renovate -- renovate-config-validator renovate.json
```

## Links

* [Renovate on GitHub](https://github.com/renovatebot/renovate)
* [Renovate docs](https://docs.renovatebot.com)
* [Upgrade best practices](https://docs.renovatebot.com/upgrade-best-practices/)
* [`config:best-practices` preset](https://docs.renovatebot.com/presets-config/#configbest-practices)
* [Mend Renovate website](https://www.mend.io/free-developer-tools/renovate/)
* [Mend Renovate GitHub app](https://github.com/marketplace/renovate)
