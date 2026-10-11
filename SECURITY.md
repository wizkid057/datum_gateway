# Security policy

This policy applies to the official Bitcoin-only DATUM Gateway project at [OCEAN-xyz/datum_gateway](https://github.com/OCEAN-xyz/datum_gateway).

Jason Hughes is the original creator of DATUM and is an active maintainer of the official repository.

## Reporting a vulnerability

**Report suspected security vulnerabilities privately. Do not disclose vulnerability details in public issues, discussions, or pull requests.** If you are unsure whether a problem is security-related, use a private reporting channel.

You can report through either of these channels:

- **GitHub private vulnerability reporting:** Open the official repository's [Security tab](https://github.com/OCEAN-xyz/datum_gateway/security) and select **Report a vulnerability**, or use the [private report form](https://github.com/OCEAN-xyz/datum_gateway/security/advisories/new).
- **Directly to Jason Hughes:** Email [jason+datum@ocean.xyz](mailto:jason+datum@ocean.xyz). Encrypt sensitive details and attachments using the OpenPGP key below. If GitHub's private reporting form is unavailable, use this direct contact.

### Jason's OpenPGP key

Download his [public key from Ubuntu's keyserver](https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x84B7D5BE8EBB91692FA7092513E0B617CF1A97A1). Verify the complete fingerprint before encrypting your report:

```text
84B7 D5BE 8EBB 9169 2FA7 0925 13E0 B617 CF1A 97A1
```

Unspaced fingerprint: `84B7D5BE8EBB91692FA7092513E0B617CF1A97A1`.

Keep email subjects free of sensitive exploit details. Include a way to contact you privately and, if desired, your public key for an encrypted reply.

## What to include

- A summary, potential impact, and any evidence of active exploitation or immediate risk to miners.
- The affected DATUM version or commit, installation method, operating system, and relevant Bitcoin Core or ASIC firmware versions.
- Required access, configuration, and network exposure; identify whether the issue affects the ASIC, pool, node RPC, or dashboard/API connection.
- Reproduction steps and a minimal proof of concept, crash trace, or relevant logs where available. A proposed fix is welcome but not required.

Use test data and remove unrelated credentials and personal information. Never send wallet seed phrases or private keys. An initial report is welcome even if you have not yet determined the full impact or all affected versions.

## Scope and versions

Report issues affecting official DATUM Gateway releases or current upstream code, including vulnerabilities that could interrupt mining, compromise a host, expose credentials, alter work, or redirect rewards. Include the exact affected version; if practical, check whether the latest official release is also affected without delaying an urgent report.

Third-party forks and their infrastructure are outside this project's maintenance scope. If a finding in a fork also affects official DATUM, please report the shared vulnerability here. Published advisories will identify affected versions, available fixes, and any mitigations; do not assume older releases will receive backports.

## Testing and coordinated disclosure

Use systems you own or have permission to test. Avoid disrupting production mining, accessing other users' data, or redirecting real rewards. Provide the minimum evidence needed to demonstrate the issue.

We will review reports privately, request clarification where needed, and coordinate fixes, mitigations, and disclosure with the reporter. Please allow time for assessment and remediation before publishing exploit details, and tell us about any planned disclosure date. If you have not received an acknowledgement after seven days, follow up through the same channel or try the other private contact method.

Disclosure timing depends on severity, exploitation risk, and the time miners need to deploy a fix. When an advisory is published, we will credit the reporter unless they prefer to remain anonymous.

For ordinary bugs and support questions, use [GitHub Issues](https://github.com/OCEAN-xyz/datum_gateway/issues) as described in the [README](README.md#reporting-problems).
