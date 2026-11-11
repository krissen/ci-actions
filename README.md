# ci-actions

Shared GitHub Actions for my repositories.

## apt-cached

Installs apt packages from a cache of their `.deb` files instead of going to
the apt mirrors on every run. The mirrors fail and hang: an uncached
`apt-get install` can sit until the job's time limit.

- The cache key is the runner image (`ImageOS`, `ImageVersion`), the
  architecture and the sorted package list. A new runner image is a miss.
- On a hit the packages are installed with `dpkg -i`, without the network.
- On a miss the download runs against the runner's existing package lists
  first. `apt-get update` runs only if that download fails. Each apt call
  has its own timeout.
- An incomplete set of packages is never saved to the cache.

```yaml
- uses: krissen/ci-actions/apt-cached@<sha>
  timeout-minutes: 6
  with:
    packages: pandoc poppler-utils
    commands: pandoc pdftotext
    # required: "false"  # warn instead of failing when the tests skip
```

| input | default | |
|---|---|---|
| `packages` | — | space- or newline-separated |
| `commands` | `""` | must be on `PATH` afterwards |
| `required` | `"true"` | `"false"` turns a failed install into a warning |
| `download-timeout` | `240` | seconds per download attempt |
| `update-timeout` | `120` | seconds for `apt-get update` |

Outputs: `cache-hit`, `installed`.

Pin the action to a commit SHA. Put `timeout-minutes` on the step that
uses it as a hard ceiling.
