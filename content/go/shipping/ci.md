+++
title = 'Continuous integration (CI)'
date = '2026-01-19T10:55:26-05:00'
weight = 10
draft = false
+++

CI is the practice of merging code changes into a shared repo and testing the results. It helps you catch errors early, and it completes all tasks automatically.

## GitHub Actions syntax

The following table describes the syntax used in GitHub action files.

| Item           | What it is        | What it does                                                 | Example                     | Why it matters                                 |
| :------------- | :---------------- | :----------------------------------------------------------- | :-------------------------- | :--------------------------------------------- |
| `uses`         | Action reference  | Runs a **prebuilt GitHub Action** instead of a shell command | `uses: actions/checkout@v3` | Reuses tested logic instead of writing scripts |
| `with`         | Action inputs     | Supplies **configuration parameters** to an action           | `with: go-version: "^1.24"` | Customizes action behavior                     |
| `key`          | Cache identifier  | Unique identifier for a cache entry                          | `key: ubuntu-go-abc123`     | Determines when cache is reused vs recreated   |
| `restore-keys` | Cache fallback    | Prefixes used to find **closest matching cache**             | `restore-keys: ubuntu-go-`  | Speeds builds even if exact cache is missing   |
| `${{ }}`       | Expression syntax | Evaluates an expression at runtime                           | `${{ runner.os }}`          | Injects dynamic values into YAML               |


{{< admonition "In other words" tip >}}
`uses` runs code, `with` configures it, `key` names cached data, `restore-keys` loosens matching rules, and `${{ }}` injects runtime values.
{{< /admonition>}}

## Caching

You can cache your Go dependencies in two methods:
- Go module cache, in `~/go/pkg/mod`
- CI systems, such as GitHub Actions, let you cache files or directories between workflow runs

To demonstrate, here is a GH Actions workflow that runs tests every time you push a commit:

1. Human-readable name
2. Run this workflow on every push to any branch, including direct pushes and merged pull requests (merges are pushes).
3. These lines define a job named `test-and-dependencies`.
4. Tells GitHub to provision a new Ubuntu VM to run the tests. This VM is deleted after the tests complete.
5. Defines the steps in the job.
6. Checks out (clones) the repos code into the runner. `actions/checkout@v3` is GitHub's official checkout action.
7. Sets up Go in the runner. This installs Go 1.24.x, adds Go to the PATH, configures GOPATH and the module cache paths.
8. Steps to cache modules and speed up builds.
9. Defines the cache path. This is Go's module download cache.
10. Uniquely identifies the cache. It includes a hash of all `go.sum` files. This means that the cache is reused unless the dependencies change.
11. A fallback to any Go cache in case the exact hash key is not found.
12. This step downloads all project dependencies listed in `go.mod`. Always run this before tests so that dependency issues are caught before tests run.
13. Runs the tests. `./...` means "run verbose tests for all packages recursively"

```yaml
name: Go CI on Commit                                               # 1

on:                                                                 # 2
  push:

jobs:                                                               # 3
  test-and-dependencies:
    runs-on: ubuntu-latest                                          # 4
    steps:                                                          # 5
      - uses: actions/checkout@v3                                   # 6
      - name: Set up Go                                             # 7
        uses: actions/setup-go@v3
        with:
          go-version: "^1.24"

      - name: Cache Go modules                                      # 8
        uses: actions/cache@v3                  
        with:
          path: ~/go/pkg/mod                                        # 9
          key: ${{ runner.os }}-go-${{ hashFiles('**/go.sum') }}    # 10
          restore-keys: |                                           # 11
            ${{ runner.os }}-go-

      - name: Get dependencies                                      # 12
        run: go mod download
      - name: Run tests                                             # 13
        run: go test -v ./...
```

## Static analysis

Static analysis checks your code for errors, deviations from best practices, and style inconsistencies. This example adds [Staticcheck](https://staticcheck.dev/) in the workflow using a pre-built GitHub action:

1. This line runs the workflow on every push for every branch. This command is equivalent to the `push` command in [Caching](#caching).
2. Defines a job named `build-test-staticcheck`.
3. Tells GitHub to provision a new Ubuntu VM to run the tests. This VM is deleted after the tests complete.
4. Checks out (clones) the repos code into the runner. `actions/checkout@v3` is GitHub's official checkout action.
5. Runs Staticcheck. This is more strict and comprehensive than `go vet`.
6. Shows how you could specify the version. When this is commented out, it uses the default version for the action.

```yaml
name: Go CI

on: [push]                                              # 1

jobs:                                                   
  build-test-staticcheck:                               # 2
    runs-on: ubuntu-latest                              # 3

    steps:
      - uses: actions/checkout@v3                       # 4
      - uses: dominikh/staticcheck-action@v1.2.0        # 5
        with:
          # Optionally specify Staticcheck version      
          # version: latest                             # 6
```

## Releasing an application

### GoReleaser configuration

[GoReleaser](https://goreleaser.com/) lets you describe how you want your app packaged and distributed and releases it. For example, you might want pre-built binaries for every OS and a Docker image pushed to a registry.

1. `builds` describes how binaries are built. 
2. Options passed to the Go linker to control how the final binary is produced.
   1. Strip the symbol table (`-s`) and debug info (`-w`)
   2. Force static linking
3. Environment variables for the build. This disables CGO to avoid libc dependencies, creating a truly static binary.
4. Target operating systems.
5. Target CPU architecture.
6. Sets the modification time of the files inside the binary.
7. Determines how the binaries are packaged.
8. Controls the artifact file names. For example, `myapp_1.2.3_linux_amd64`.
9. Wraps the binary in a folder before archiving. For example, `myapp_1.2.3_linux_amd64/myapp`
10. Produces raw binaries, not tarballs.
11. Defines OS-specific packaging rules. Here, Windows binaries are zipped.
12. Build and publish Docker images. The major and minor versions are derived by GoReleaser.

```yaml
builds:                                             # 1
  - ldflags:                                        # 2
      - -s -w                                       # 2.1
      - -extldflags "-static"                       # 2.2
    env:                                            # 3
      - CGO_ENABLED=0
    goos:                                           # 4
      - linux
      - windows
      - darwin
    goarch:                                         # 5
      - amd64
    mod_timestamp: "{{ .CommitTimestamp }}"         # 6
archives:                                           # 7
  - name_template: "{{ .ProjectName }}_{{ .Version }}_{{ .Os }}_{{ .Arch }}"    # 8
    wrap_in_directory: true                         # 9
    format: binary                                  # 10
    format_overrides:                               # 11
      - goos: windows
        format: zip

dockers:                                            # 12
  - image_templates:                                
      - "ghcr.io/alexrios/endpoints:{{ .Tag }}"
      - "ghcr.io/alexrios/endpoints:v{{ .Major }}"
      - "ghcr.io/alexrios/endpoints:v{{ .Major }}.{{ .Minor }}"
      - "ghcr.io/alexrios/endpoints:latest"
```

### GitHub action

This is the GitHub action that corresponds to the [GoReleaser configuration](#goreleaser-configuration). It runs GoReleaser when you push a Git tag.

Here is a quick explanation of git tag commands:
1. Tag the current commit. This creates a full git object with metadata.
2. Push a single tag.
3. List local tags.
4. Show tag details.
5. Delete a local tag.
6. Delete a remote tag.

```bash
git tag -a v1.2.3 -m "Release v1.2.3"       # 1
git push origin v1.2.3                      # 2
git tag                                     # 3
git show v1.2.3                             # 4
git tag -d v1.2.3                           # 5
git push origin :refs/tags/v1.2.3           # 6
```

Here is the GitHub Action file:
1. Run this when you push any tag. GoReleaser will use the tag as the version.
2. Define a job named `goreleaser`.
3. Tells GitHub to provision a new Ubuntu VM to run the tests. This VM is deleted after the tests complete.
4. GitHub token permissions for this workflow.
   1. Create GitHub releases and upload assets.
   2. Push Docker images to GitHub Container Registry (GHCR).
5. Checks out (clones) the repos code into the runner. `actions/checkout@v3` is GitHub’s official checkout action.
6. Fetches the full git history for changelogs and all tags. GitReleaser needs this information, or it will fail.
7. Sets up Go in the runner. This installs Go 1.21.x, adds Go to the PATH, configures GOPATH and the module cache paths.
8. Runs GoReleaser as an action.
9. GoReleaser options:
   - `version: latest`: Install the latest version of GoReleaser.
   - `release`: Perform a release (not a dry run)
   - `--rm-dist`: Delete the `dist/` directory before building.
   - `--clean`: Clean up temp files after building.
10. Provide GitHub's built in token. This lets GoReleaser create GitHub releases, upload binaries, and push images to the GHCR.

```yaml
name: GoReleaser

on:                                                     # 1
  push:
    tags:
      - '*'

jobs: 
  goreleaser:                                           # 2
    runs-on: ubuntu-latest                              # 3
    permissions:                                        # 4
      packages: write                                   # 4.1
      contents: write                                   # 4.2
    steps:
      - name: Checkout                                  # 5
        uses: actions/checkout@v2
        with:
          fetch-depth: 0                                # 6
      - name: Set up Go                                 # 7
        uses: actions/setup-go@v2
        with:
          go-version: 1.21
      - name: Run GoReleaser                        
        uses: goreleaser/goreleaser-action@v2           # 8
        with:                                           # 9
          version: latest
          args: release --rm-dist --clean
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}     # 10
```
