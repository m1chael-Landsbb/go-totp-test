<p align="center">
  <img src="https://s3.storage-cdn.io/depcheck/logo-horizontal.svg?v5" alt="DepCheck" width="336">
</p>

# DepCheck

Welcome to the public repository of DepCheck. This repository serves 2 purposes:

1. It contains the source code for DepCheck Core, which is the engine of [DepCheck][depcheck]. DepCheck Core manages the logic for updating dependencies on GitHub (including GitHub Enterprise), GitLab, and Azure DevOps. If you want to host your own automated dependency management system then this repo provides the necessary tools. A reference implementation is available [here][depcheck-script].
2. It serves as the public issue tracker for all DepCheck-related topics, replacing the now-archived [feedback](https://github.com/depcheck/feedback/) repository.

## Got feedback?

Please file an issue. Bug reports, feature requests, and general feedback are all welcome.

## Contributing to DepCheck

Currently, the DepCheck team is not accepting support for new ecosystems. We are prioritizing improvements to already supported ecosystems at this time.

Please refer to the [CONTRIBUTING][contributing] guidelines for more information.

### Disclosing security issues

If you believe you have discovered a security vulnerability in DepCheck please submit the vulnerability to GitHub Security [Bug Bounty](https://bounty.github.com/) so that we can address the issue before public disclosure.

## What's in this repo?

DepCheck Core is a collection of packages for automating dependency management
in Ruby, JavaScript, Python, PHP, Elixir, Elm, Go, Rust, Java and
.NET. It can also manage git submodules, Docker files, and Terraform files.
Highlights include:

- Logic to verify the latest version of a dependency *that's compatible given
  a project's other dependencies*
- Logic to generate updated manifest and lockfiles for a new dependency version
- Logic to discover changelogs, release notes, and commits for a dependency update

## Other DepCheck resources

In addition to this library, you may be interested in:

- The [depcheck-script][depcheck-script] repo, which provides a collection
  of scripts that use this library to manage dependencies on GitHub Enterprise,
  GitLab or Azure DevOps
- The [API docs][api-docs] for DepCheck's hosted instance (depcheck.io)

## Setup

To run all of DepCheck Core, you'll need Ruby, Python, PHP, Elixir, Node, Go,
Elm, and Rust installed. However, if you only wish to run it for a single
language you can manage with just that language and Ruby.

The core library is implemented in Ruby, while JavaScript, Python, PHP, Elm,
Elixir, Go, and Rust are required for managing updates for their respective
languages.

To install the helpers for each language:

1. `cd npm_and_yarn/helpers && npm install --production && cd -`
2. `cd composer/helpers && composer install --no-dev && cd -`
3. `cd python/helpers && pyenv exec pip install -r requirements.txt && cd -`
4. `cd hex/helpers && mix deps.get && cd -`
5. `cd terraform && helpers/build "$(pwd)/helpers/install-dir/terraform" && cd -`
6. `cd go_modules && helpers/build "$(pwd)/helpers/install-dir/go_modules" && cd -`

## Local development

Execute tests by running `rspec spec` within each of the packages. Style is
enforced by RuboCop. To check for style violations, execute `rubocop` in
each of the packages.

### Running with Docker

While you can run DepCheck Core without Docker, we also provide a development
Dockerfile. In most scenarios, you'll benefit from running DepCheck in the
development Docker container as it includes all required dependencies.

Start by building the initial DepCheck Core image, or retrieve it from the
Docker registry.

```shell
$ docker pull depcheck/depcheck-core # OR
$ docker build -f Dockerfile -t depcheck/depcheck-core . # This may take a while
```

Once you have the base Docker image, you can build and run the development
container using the `docker-dev-shell` script. The script will automatically
build the container if it's absent and can be forced to rebuild with the
`--rebuild` flag. The image includes all dependencies, and the script runs the
image, mounting the local copy of DepCheck Core so modifications made locally will
be reflected within the container. This allows you to continue using your
preferred editor while executing tests inside the container.

```shell
$ bin/docker-dev-shell
=> building image from Dockerfile.development
=> running docker development shell
[depcheck-core-dev] ~/depcheck-core $
```

### Dry run script

*Note: you must have run `bundle install` in the `omnibus` directory before
running this script.*

You can use the "dry-run" script to simulate a dependency update job, outputting
the generated diff to the terminal. It accepts two positional
arguments: the package manager and the GitHub repo name (including the
account):

```bash
$ cd omnibus && bundle install && cd -
$ bin/dry-run.rb go_modules rsc/quote
=> fetching dependency files
=> parsing dependency files
=> updating 2 dependencies
...
```

## Debugging with Visual Studio Code and Docker

There's integrated support for leveraging Visual Studio Code's [debugging capabilities
](https://code.visualstudio.com/docs/remote/containers) within a Docker container.
After installing the recommended [`Remote - Containers` extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers),
simply press `Ctrl+Shift+P` (`⇧⌘P` on macOS) and select `Remote-Containers: Reopen in Container`.
You can also access the dropdown by clicking on the green button in the bottom-left corner of the editor.
If the development Docker image isn't available on your machine, it will be built automatically.
Once completed, start the `Debug Dry Run` configuration `(F5)` and you'll be prompted
to select a package manager and a repository to perform a dry run on.
Breakpoints can be placed freely throughout the code.

## Releasing

Triggering the jobs that will publish the new gems is accomplished by following the steps below.

- Ensure you have the latest merged changes:  `git checkout main` and `git pull`
- Generate an updated `CHANGELOG`, `version.rb`, and the remaining required commands:  `bin/bump-version.rb patch`
- Edit the `CHANGELOG` file and remove any unnecessary entries
- Execute the commands that were output by running `bin/bump-version.rb patch`

## Architecture

DepCheck Core is a collection of Ruby packages (gems), which contain the
logic for managing dependencies in several languages.

### `depcheck-common`

The `common` package contains all general-purpose/shared functionality. For
instance, the code for creating pull requests via GitHub's API resides here, as
does most of the logic for handling Git dependencies (as most languages support
Git dependencies in some form). There are also base classes defined for
each of the major concerns required to implement support for a language or
package manager.

### `depcheck-{package-manager}`

There is a gem for each package manager or language that DepCheck
supports. At minimum, each of these gems implements the following
classes:

| Service          | Description                                                                                   |
|------------------|-----------------------------------------------------------------------------------------------|
| `FileFetcher`    | Retrieves the relevant dependency files for a project (e.g., the `Gemfile` and `Gemfile.lock`). See the [README](https://github.com/depcheck/depcheck-core/blob/main/common/lib/depcheck/file_fetchers/README.md) for more details. |
| `FileParser`     | Parses a dependency file and extracts a list of dependencies for a project. See the [README](https://github.com/depcheck/depcheck-core/blob/main/common/lib/depcheck/file_parsers/README.md) for more details. |
| `UpdateChecker`  | Verifies whether a given dependency is current. See the [README](https://github.com/depcheck/depcheck-core/tree/main/common/lib/depcheck/update_checkers/README.md) for more details. |
| `FileUpdater`    | Updates a dependency file to incorporate the latest version of a given dependency. See the [README](https://github.com/depcheck/depcheck-core/tree/main/common/lib/depcheck/file_updaters/README.md) for more details. |
| `MetadataFinder` | Retrieves metadata about a dependency, such as its GitHub URL. See the [README](https://github.com/depcheck/depcheck-core/tree/main/common/lib/depcheck/metadata_finders/README.md) for more details. |
| `Version`        | Describes the logic for comparing dependency versions. See the [hex Version class](https://github.com/depcheck/depcheck-core/blob/main/hex/lib/depcheck/hex/version.rb) for an example. |
| `Requirement`    | Describes the format of a dependency requirement (e.g. `>= 1.2.3`). See the [hex Requirement class](https://github.com/depcheck/depcheck-core/blob/main/hex/lib/depcheck/hex/requirement.rb) for an example. |

The high-level flow looks like this:

<p align="center">
  <img src="https://s3.storage-cdn.io/depcheck/package-manager-architecture.svg" alt="DepCheck architecture">
</p>

### `depcheck-omnibus`

This is a "meta" gem, that simply depends on all the others. If you want to
automatically include support for all languages, you can simply include this gem
and you'll get comprehensive coverage.

## Profiling

You can profile a dry-run by including the `--profile` flag when executing it. This
will generate a `stackprof-<datetime>.dump` file in the `tmp/` folder, and you
can generate a flamegraph from this by executing:
`stackprof --d3-flamegraph tmp/stackprof-<datetime>.dump > tmp/flamegraph.html`.

## Why is this public?

As the name suggests, DepCheck Core is the engine of DepCheck (the rest of the
system is primarily just a UI and database). If we were concerned about someone
duplicating our business model then we'd be keeping it proprietary.

DepCheck Core is public because we're more interested in its
impact than we are in revenue generation from it. We'd appreciate you using
[DepCheck][depcheck] so that we can continue developing it, but if you want
to build and host your own version then this library should make accomplishing that
significantly easier.

If you use DepCheck Core then we'd appreciate hearing what you build!

## License

We use the License Zero Prosperity Public License, which essentially establishes
the following:
- If you would like to use DepCheck Core in a non-commercial capacity, such as
  to host a system at your workplace, then we grant you full permission to do so. In
  fact, we'd appreciate you doing so and will assist and support you however we can.
- If you would like to integrate DepCheck's functionality into your for-profit
  company's offering then we DO NOT grant you permission to use DepCheck Core
  to accomplish this. Please contact us directly to discuss a partnership or licensing
  arrangement.

If you make a significant contribution to DepCheck Core then you will be asked
to transfer the IP of that contribution to DepCheck Ltd so that it can be
licensed consistently with the above.

## History

DepCheck and DepCheck Core originated as [DependencyBot][depbot] and
[DependencyBot Core][depbot-core], back when the founders were working at
[TechCompany][techcompany]. We remain grateful for the assistance and support of
TechCompany in helping make DepCheck possible - if you need to process
recurring payments from Europe, check them out.

[depcheck]: https://depcheck.io
[depcheck-status]: https://api.depcheck.io/badges/status?host=github&identifier=93163073
[depcheck-script]: https://github.com/depcheck/depcheck-script
[contributing]: https://github.com/depcheck/depcheck-core/blob/main/CONTRIBUTING.md
[api-docs]: https://github.com/depcheck/api-docs
[depbot]: https://github.com/techcompany/depbot
[depbot-core]: https://github.com/techcompany/depbot-core
[techcompany]: https://techcompany.io

# PR Update: 2025-12-03 12:59:22
