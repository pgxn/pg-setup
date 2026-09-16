# GitHub Action to Install and Configure PostgreSQL

[![⚖️ PostgreSQL]][pg] [![🎬 Action]][action] [![🧪 Test]][ci]

This action sets up a PostgreSQL server on the GitHub runner VM to enable the
automated testing of PGXN extensions against multiple versions of PostgreSQL.
It currently supports:

*   Ubuntu Runners with PGDG-installed PostgreSQL 8.2-19
*   macOS Runners with Homebrew-installed PostreSQL 14-18
*   Windows Runners with Chocolatey-installed PostreSQL 10-18[^win-pgxs]

In addition, each provies the [pgxn client] to simplify installing additional
extension from [PGXN]. The Windows images also adds `sudo` to minimize
differences in the commands required to install extension on each OS.

Example workflow:

``` yaml
name: 🧪 Test
on:
  push:
defaults:
  run: { shell: bash }
jobs:
  test:
    strategy:
      fail-fast: false
      matrix:
        include:
          - { img: 🐧, os: Ubuntu,  vm: latest,    arch: amd64, pg: 19 }
          - { img: 🐧, os: Ubuntu,  vm: latest,    arch: amd64, pg: 18 }
          - { img: 🐧, os: Ubuntu,  vm: latest,    arch: amd64, pg: 17 }
          - { img: 🐧, os: Ubuntu,  vm: 26.04-arm, arch: arm64, pg: 19 }
          - { img: 🐧, os: Ubuntu,  vm: 26.04-arm, arch: arm64, pg: 18 }
          - { img: 🐧, os: Ubuntu,  vm: 26.04-arm, arch: arm64, pg: 17 }
          - { img: 🍎, os: macOS,   vm: latest,    arch: arm64, pg: 18 }
          - { img: 🍎, os: macOS,   vm: latest,    arch: arm64, pg: 17 }
          - { img: 🍎, os: macOS,   vm: 26-intel,  arch: amd64, pg: 18 }
          - { img: 🍎, os: macOS,   vm: 26-intel,  arch: amd64, pg: 17 }
          - { img: 🪟, os: Windows, vm: latest,    arch: amd64, pg: 18 }
          - { img: 🪟, os: Windows, vm: latest,    arch: amd64, pg: 17 }
          - { img: 🪟, os: Windows, vm: 11-arm,    arch: arm64, pg: 18 }
          - { img: 🪟, os: Windows, vm: 11-arm,    arch: arm64, pg: 17 }
    name: ${{ matrix.img }} ${{ matrix.arch }} 🐘 ${{ matrix.pg }}
    runs-on: ${{ matrix.os }}-${{ matrix.vm }}
    steps:
      - name: Check out the repo
        uses: actions/checkout@v7
      - name: Start Postgres ${{ matrix.pg }}
        uses: pgxn/pg-setup@v0
        with: { version: "${{ matrix.pg }}" }
      - name: Build
        run:  make
      - name: Install
        run:  sudo make
      - name: Test
        id:   test
        run:  make installcheck
      - name: Show Diffs
        if:   failure() && steps.test.outcome == 'failure'
        run:  find . -name regression.diffs -exec cat {} +
```

## Input Parameters

This action takes the following parameters:

| Key                | Type    | Default   | Description                                          |
| ------------------ | ------- | --------- |----------------------------------------------------- |
| `version`          | string  | ""        | PostgreSQL major version to install                  |
| `port`             | integer | 55432     | Port on which PostreSQL should listen for cnnections |
| `packages`         | string  | ""        | List of additional OS-specific packages to install   |
| `start`            | boolean | true      | Start the PostgreSQL server after installing         |
| `encoding`         | string  | ""        | The encoding to use for databases in the cluster     |
| `locale`           | string  | ""        | The locale to use for databases in the cluster       |

For the `packages` input, use package names specific to the OS packaging system:

*   Linux: [Debian Packages]
*   macOS: [Homebrew Formulae]
*   Windows: [Chocolatey Packages]

## Environment Variables

On completion, this action sets the following environment variables:

| Variable    | Value        | Description                                     |
| ----------- | ------------ | ----------------------------------------------- |
| `PGUSER`    | "postgres"   | The name of the PostgreSQL super user           |
| `PGPORT`    | `input.port` | The port on which the PostgreSQL server listens |
| `PG_CONFIG` | varies       | The path `pg_config`, used to build extensions  |

## Path

On completion, this action adds the path to the PostgreSQLl executables to the
`PATH` environment varaible, so they can be called without needing to know the
full, often version-specific path.

## Prior Art/Inspirations

*   [pgxn-tools]: Old PGXN Linux/amd64-only OCI image for testing extensions
*   [petere/pguint]: Commit converting to GitHub actions using `apt.postgresql.org`
*   [ikalnytskyi/action-setup-postgres]: Setup PostgreSQL for Linux, macOS and
    Windows runner machines

  [^win-pgxs]: Although currently the standard `include $(PGXS)` pattern in
    `Makefiles` appears to work only on Postgres 17 and later, because
    `pg_config --pgxs` returns a path with spaces in it on earlier versions.

  [⚖️ PostgreSQL]: https://img.shields.io/badge/License-PostgreSQL-blue.svg "⚖️ PostgreSQL License"
  [pg]: https://opensource.org/license/postgresql "⚖️ PostgreSQL License"
  [🧪 Test]: https://github.com/pgxn/pg-setup/actions/workflows/test.yml/badge.svg "🧪 Test Status"
  [ci]: https://github.com/pgxn/pg-setup/actions/workflows/test.yml "🧪 Test Status"
  [🎬 Action]: https://img.shields.io/badge/Marketplace-Action-orange.svg "[🎬 Marketplace Action]"
  [action]: https://github.com/marketplace/actions/pg-setup "[🎬 Marketplace Action]"
  [Debian Packages]: https://packages.debian.org/index
  [Homebrew Formulae]: https://formulae.brew.sh
  [Chocolatey Packages]: https://community.chocolatey.org/packages/
  [pgxn client]: https://pgxn.github.io/pgxnclient/
  [pgxn-tools]: https://github.com/pgxn/docker-pgxn-tools/ "Test image for PostgreSQL & PGXN extensions"
  [petere/pguint]: https://github.com/petere/pguint/commit/bcc3335
    "petere/pguint@bcc3335 Convert CI from Cirrus to GitHub Actions"
  [ikalnytskyi/action-setup-postgres]: https://github.com/ikalnytskyi/action-setup-postgres/
    "Setup a PostgreSQL for Linux, macOS and Windows runner machines"
