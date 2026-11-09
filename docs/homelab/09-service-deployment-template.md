# Service Deployment Template — Copy for New Services

> Replace the bracketed placeholders. Do not publish passwords, private keys, access tokens or unique device IDs.

## Context

- **Service name:** [name]
- **Deployment date:** [YYYY-MM-DD]
- **Goal / problem:** [why this service was needed]
- **Current status:** [running / testing / planned / retired]
- **Where it runs:** [host / LXC / VM, guest ID in private notes only]
- **Constraints:** [CPU, RAM, storage, connectivity]

## Architecture and decisions

- **Alternatives considered:** [options and trade-offs]
- **Network path:** [client → gateway → host → service]
- **Dependencies:** [DNS, storage, authentication, upstream providers]
- **Trust boundary:** [who can access it and how]

## Deployment record

1. [Observed setup step 1]
2. [Observed setup step 2]
3. [Observed setup step 3]

> Record the real commands used in a private log before publishing a sanitized version. For commands not actually executed, label them “illustrative example.”

## Validation / evidence

| Test | Command or method | Expected | Observed | Status |
|---|---|---|---|---|
| Reachability | [example] | [result] | [result] | [pass/fail] |
| Service | [example] | [result] | [result] | [pass/fail] |
| Security / access | [example] | [result] | [result] | [pass/fail] |

## Troubleshooting

- **Symptom:** [what was visible]
- **Hypotheses:** [possible causes]
- **Test:** [how the causes were separated]
- **Confirmed cause:** [only if proven]
- **Action / result:** [what changed and what happened]

## Explain this in an interview

- **What it is:** [2 sentences]
- **Why I chose it:** [real trade-off]
- **How it works:** [network/data path]
- **How I tested it:** [proof]
- **What I would improve:** [specific next step]

## Next actions

- [ ] Patch/version audit
- [ ] Backup + restore test
- [ ] Monitoring and alerts
- [ ] Publishing safety review
