# GitHub Action to Install and Configure PostgreSQL

[![⚖️ PostgreSQL]][pg] [![🎬 Action]][action] [![🧪 Test]][ci]

This action sets up a PostgreSQL server on the GitHub runner VM to enable the
automated testing of PGXN extensions against multiple versions of PostgreSQL.
It currently supports:

*   Ubuntu Runners, PostgreSQL 8.2-19
*   macOS Runners, PostreSQL 14-18

Example workflow:

``` yaml
name: 🧪 Test
on:
  push:
jobs:
  test:
    strategy:
      fail-fast: false
      matrix:
        include:
          - { img: 🐧, os: Ubuntu, vm: latest,    arch: amd64, pg: 19  }
          - { img: 🐧, os: Ubuntu, vm: latest,    arch: amd64, pg: 18  }
          - { img: 🐧, os: Ubuntu, vm: latest,    arch: amd64, pg: 17  }
          - { img: 🐧, os: Ubuntu, vm: 26.04-arm, arch: arm64, pg: 19  }
          - { img: 🐧, os: Ubuntu, vm: 26.04-arm, arch: arm64, pg: 18  }
          - { img: 🐧, os: Ubuntu, vm: 26.04-arm, arch: arm64, pg: 17  }
          - { img: 🍎, os: macOS,  vm: latest,    arch: arm64,  pg: 18  }
          - { img: 🍎, os: macOS,  vm: latest,    arch: arm64,  pg: 17  }
          - { img: 🍎, os: macOS,  vm: 26-intel,  arch: amd64,  pg: 18  }
          - { img: 🍎, os: macOS,  vm: 26-intel,  arch: amd64,  pg: 17  }
    name: ${{ matrix.img }} ${{ matrix.arch }} 🐘 ${{ matrix.pg }}
    runs-on: ${{ matrix.os }}-${{ matrix.vm }}
    steps:
      - name: Check out the repo
        uses: actions/checkout@v7
      - name: Start Postgres ${{ matrix.pg }}
        id: pg
        uses: pgxn/pg-setup@v0
        with: { version: "${{ matrix.pg }}" }
      - name: Build
        run: make
      - name: Install
        run: sudo make
      - name: Test
        id: test
        run: make installcheck
      - name: Show Diffs
        if: failure() && steps.test.outcome == 'failure'
        run: find . -name regression.diffs -exec cat {} +
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

  [⚖️ PostgreSQL]: https://img.shields.io/badge/License-PostgreSQL-blue.svg "⚖️ PostgreSQL License"
  [pg]: https://opensource.org/license/postgresql "⚖️ PostgreSQL License"
  [🧪 Test]: https://github.com/pgxn/pg-setup/actions/workflows/test.yml/badge.svg "🧪 Test Status"
  [ci]: https://github.com/pgxn/pg-setup/actions/workflows/test.yml "🧪 Test Status"
  [🎬 Action]: https://img.shields.io/badge/Marketplace-Action-orange.svg "[🎬 Marketplace Action]"
  [action]: https://github.com/marketplace/actions/pg-setup "[🎬 Marketplace Action]"
