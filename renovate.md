# Renovate integration

Repositories under the [`InteractionDesignFoundation/`](https://github.com/InteractionDesignFoundation/) scope receive regular dependency
upgrades through [renovate](https://github.com/renovatebot/renovate),
which automates the heavy lifting of uplading lock files, supported dependency ranges, etc.

This is defined in [`renovate-config.json`](./renovate-config.json), and imported in all applicable
repositories in the organization.

The configuration is heavily inspired by the
[`laminas/.github` one](https://github.com/laminas/.github/blob/a1debce9a0842e3000ae9e01fbc60e32db62901d/renovate-config.json).

To use this configuration, create a [`renovate.json`](./.github/renovate.json) file in the repository where it should be active:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "local>InteractionDesignFoundation/.github:renovate-config"
  ]
}
```

## Security Updates Only Configuration

Extending the default, the file `renovate-config-security-updates-only.json` is made specifically for use in InteractionDesignFoundation repositories that have been marked as receiving "security-updates-only".

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "local>InteractionDesignFoundation/.github:renovate-config-security-updates-only"
  ]
}
```

Please note that this file will change according to **OUR OWN** needs: no BC guaranteed.
I endorse that you copy the configuration and adapt it yourself, should you need it.