# Renovate

Shared [Renovate](https://github.com/renovatebot/renovate) presets for InteractionDesignFoundation repositories. Distributed as [shareable presets](https://docs.renovatebot.com/config-presets/).

Three configs:

| File | Use case |
| --- | --- |
| `renovate-config.json` | Default. Security critical. Frequent updates, OSV alerts, pinned digests, weekly lockfile maintenance. |
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

* `config:recommended` (dependency dashboard, monorepo grouping, ignore tests, changelog helpers).
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
| `:semanticCommitsDisabled` | No semantic commit prefixes. |
| `:enableVulnerabilityAlerts` | Open PRs for GitHub Vulnerability Alerts. |
| `:timezone(UTC)` | Schedules use UTC. |
| `:gitSignOff` | Sign off commits (DCO). |
| `:label(dependencies)` | Label PRs with `dependencies`. |

Top level options:

| Option | Value | Why |
| --- | --- | --- |
| `osvVulnerabilityAlerts` | `true` | OSV database alerts for direct deps. Catches malicious packages. |
| `commitBodyTable` | `true` | Update table in commit body. |
| `platformAutomerge` | `true` | Use GitHub native merge, fall back to Renovate. |
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
| `extends: schedule:monthly` | first of month, before 04:00 UTC | One run per month. |
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

## Links

* [Renovate on GitHub](https://github.com/renovatebot/renovate)
* [Renovate docs](https://docs.renovatebot.com)
* [Upgrade best practices](https://docs.renovatebot.com/upgrade-best-practices/)
* [`config:best-practices` preset](https://docs.renovatebot.com/presets-config/#configbest-practices)
* [Mend Renovate website](https://www.mend.io/free-developer-tools/renovate/)
* [Mend Renovate GitHub app](https://github.com/marketplace/renovate)
