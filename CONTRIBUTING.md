# Contributing to DATUM Gateway

Bug fixes, documentation, tests, reviews, and improvements to Bitcoin mining are welcome.

## Project scope

**[OCEAN-xyz/datum_gateway](https://github.com/OCEAN-xyz/datum_gateway) is the original and official upstream repository. DATUM Gateway officially supports Bitcoin mining only.** Contributions here should serve that purpose and preserve compatibility with Bitcoin's consensus rules and proof of work.

Other forks may support non-Bitcoin mining, and everyone is welcome to contribute to whichever projects they choose. This repository is not a platform for promoting non-Bitcoin cryptocurrencies, mining algorithms, pools, or forks. Promotional references to those projects do not belong here, including in code, documentation, issues, pull requests, commit messages, etc. Maintainers may request their removal or decline contributions that fall outside the project's scope.

## Before starting

- Read the [README](README.md) and search existing issues and pull requests to avoid duplicating work.
- For substantial features or changes to protocol behavior, reward handling, or architecture, open an issue to discuss the approach before investing significant effort. Small fixes can go directly to a pull request.
- Report ordinary bugs through [GitHub Issues](https://github.com/OCEAN-xyz/datum_gateway/issues), with reproduction steps, versions, and relevant logs. Remove credentials and private information.
- Report suspected vulnerabilities privately under [SECURITY.md](SECURITY.md), including security fixes that would reveal an undisclosed vulnerability.

## Pull requests

Create a branch in your fork and submit a pull request to the official upstream repository. Keep changes focused, follow the surrounding code's style, and avoid unrelated formatting or refactoring.

Explain the problem, the resulting behavior, and how you verified the change. Link related issues and call out configuration, compatibility, or operational effects. Use clear commit messages that explain the change and its purpose. Update affected documentation and examples.

For code changes, follow the [build instructions](doc/INSTALL.md#building-from-source), then run from the repository directory:

```bash
cmake .
cmake --build .
./datum_gateway --test
```

Add or update meaningful tests for changed behavior where practical. Mining changes may also need checks with a node, pool, or ASIC; describe the environment and any coverage you could not perform. For documentation-only changes, check links, commands, examples, and rendering. Address relevant CI failures and review feedback before requesting another review.

## Working together and licensing

Keep discussions respectful and technical. Explain disagreements with evidence and avoid personal attacks. Maintainers make the final decision on whether a contribution fits the project and is ready to merge.

Only submit work you have the right to contribute under the project's [MIT license](LICENSE). Preserve required copyright, license notices, and attribution for existing or third-party material. These contribution guidelines govern participation in this repository; they do not change the software license.
