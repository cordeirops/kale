# Exercising the release workflow in test mode

The `Publish release artifacts` workflow supports two entry points:

- **Tag-driven releases**: publish to PyPI and npm automatically when a `v*` release
  is published on GitHub.
- **Manual runs**: run the same workflow on demand from the Actions tab in either
  real release mode or a non-destructive test mode.

## Running a manual rehearsal

1. Open **Actions → Publish release artifacts → Run workflow**.
2. Enter the git ref to validate in the **tag** field. This can be any
   existing tag or branch – the workflow checks out that ref before building.
3. Choose **test** for the **mode** input so the jobs target staging services.

In test mode the workflow will:

- Build the backend wheels and upload them to TestPyPI using the
  `TEST_PYPI_API_TOKEN` secret. No artefacts touch the production index.
- Build the labextension, run `npm publish --dry-run`, and produce a tarball via
  `npm pack`. This exercises the packaging logic end-to-end without creating a
  release on npm. You can optionally provide a `TEST_NPM_TOKEN` secret if you want
  to validate authentication as well, but it is not required for the dry run.

After the rehearsal, re-run the workflow with **mode** set
to `release` (or create/publish a GitHub release tagged `v*`) to publish to the
production registries using the existing `PYPI_API_TOKEN` and `NPM_TOKEN`
secrets.

## Required secrets

| Secret name            | Used in mode | Purpose                                           |
| ---------------------- | ------------ | ------------------------------------------------- |
| `PYPI_API_TOKEN`       | release      | Publish backend artefacts to PyPI.               |
| `NPM_TOKEN`            | release      | Publish the labextension to npm.                 |
| `TEST_PYPI_API_TOKEN`  | test         | Push backend artefacts to https://test.pypi.org. |
| `TEST_NPM_TOKEN` (opt) | test         | Auth token for registry checks during dry runs.  |

## Troubleshooting

- **Missing TestPyPI token**: the TestPyPI upload step fails fast if the token is
  absent. Provide `TEST_PYPI_API_TOKEN` in the repository secrets to enable test
  runs.
- **Dry-run validation**: `npm publish --dry-run` performs the same validations as
  a real publish (including checking for missing files and metadata) but stops
  short of creating the release. Inspect the generated tarball in the workflow
  artefacts to review the package contents.
