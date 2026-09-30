# CI/CD Concepts Reference

A reference for the core mechanics behind a CI/CD pipeline, why each piece exists, not just what it does.

## CI vs CD

- **CI (Continuous Integration):** every push is automatically built and tested. Answers: *does the code still work?*
- **Continuous Delivery:** code is automatically packaged and staged, ready to deploy, but a human approves the release to production.
- **Continuous Deployment:** no approval step, if tests pass, it ships automatically.

The deciding factor between Delivery and Deployment usually comes down to two questions: *how much do we trust automated checks to catch what actually matters*, and *what's the cost if something slips through anyway*. A typo fix might auto-deploy; a change touching payment data usually gets a human check, because that's the kind of risk a test suite isn't built to catch (e.g. a data leak, no test exists for a bug nobody anticipated).

## Trigger Configuration

```yaml
on:
  push:
    branches:
      - main
```

Restricting triggers to `main` isn't a Git formality, it's a checkpoint. Without it, any branch (including unreviewed, in-progress work) can trigger a deploy if its tests happen to pass. Restricting to `main` means code only reaches deploy after going through whatever review process happens before merge.

## Job Dependencies (`needs`)

By default, GitHub Actions runs jobs in a workflow **in parallel**. Without `needs: <job>`, a `deploy` job has no awareness of whether a `test` job passed or failed, it would start immediately, at the same time as `test`, and finish its own steps regardless of the test result.

```yaml
deploy:
  needs: test
```

This isn't just about ordering, it's the only thing preventing a failing test and a successful deploy from happening at the same time.

## Matrix Strategy

```yaml
strategy:
  matrix:
    python-version: ["3.9", "3.10", "3.11"]
```

Runs the same job multiple times, once per listed value, in this case, running tests across three Python versions in parallel. Catches version-specific incompatibilities (code that only works on one version) in CI, before they surface as a failure in a different environment or for a different user.

## OIDC Authentication

```yaml
permissions:
  id-token: write

- uses: aws-actions/configure-aws-credentials@v6
  with:
    role-to-assume: arn:aws:iam::<account>:role/<role-name>
```

No long-lived AWS access key is stored anywhere. Instead, GitHub generates a short-lived, signed OpenID Connect (OIDC) token at the moment the workflow runs. AWS has a pre-configured trust relationship on the IAM role accepting tokens from GitHub, and issues temporary credentials (roughly an hour) in response.

**Why it matters:** a stored access key is a persistent secret that exists until someone manually rotates it, if it leaks, it works immediately and keeps working. OIDC removes that risk at the source: there's no permanent secret to steal in the first place.

## Semantic Versioning (SemVer)

`MAJOR.MINOR.PATCH`, each number signals a different level of risk to someone upgrading:

| Bump | Meaning | Safe to upgrade? |
|---|---|---|
| PATCH | Bug/security fix, no new features, no breaking changes | Yes |
| MINOR | New feature, backwards compatible | Yes |
| MAJOR | Breaking change | No, read the changelog first |

## Conventional Commits

| Prefix | Version impact | Why |
|---|---|---|
| `fix:` | PATCH | Corrects a bug, no new behavior |
| `feat:` | MINOR | Adds new behavior, stays backwards compatible |
| `feat!:` / `fix!:` / `BREAKING CHANGE:` footer | MAJOR | Overrides the type's normal bump, existing usage may break |
| `chore:` | No bump | Doesn't fix a bug, add a feature, or change behavior for the user (e.g. linter config, dependency bumps), nothing for a version number to signal |

The breaking-change flag is independent of the `feat`/`fix` label, either type can be marked breaking, and doing so always forces a MAJOR bump regardless of what the type alone would normally trigger.

## Security Groups: Load Balancer vs Instance

- **Load balancer SG:** controls what's allowed in from the public internet (e.g. HTTPS from anywhere).
- **Instance SG:** only trusts traffic from the load balancer's SG, not the internet directly.

Forcing all traffic through one controlled entry point reduces attack surface (an instance's IP being directly reachable gives more surface area to probe or attack) and centralizes security configuration in one managed, monitored place instead of scattering it across every instance.
