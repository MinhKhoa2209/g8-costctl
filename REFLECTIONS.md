# Reflections

## Multi-account

To run `costctl` across 100 AWS accounts, I would stop assuming one set of credentials and add a profile or role loop. The clean approach is to assume a read-only or limited-write cross-account role in each target account, run the same command per account, and aggregate the results into a machine-readable output such as JSON or CSV.

For commands like `cost` and `list`, that aggregation layer matters more than the CLI itself. I would also include account id, account alias, and region in every output row so results stay auditable when multiple accounts are queried in one run.

## `idle` vs Trusted Advisor

An `idle` command based on a 24-hour CPU window is useful for fast feedback during labs or short-lived environments because it reacts quickly to recent usage changes. It is better when I want a near-real-time signal before I shut down practice resources at the end of the day.

Trusted Advisor is stronger when I care about safer long-horizon decisions because its longer observation window reduces false positives. I would trust Trusted Advisor more before deleting or rightsizing production resources, and trust `idle` more for temporary practice infrastructure where speed matters more than historical confidence.

## `clean --apply` blast radius

If `clean --tag Environment=dev --apply` were run in a shared account, the first safeguard I would want is tighter scoping than a generic environment tag. A safer design is to require an ownership tag such as `Owner`, `Team`, or `purpose=practice`, and to default to dry-run unless an explicit apply flag is present.

I would also want IAM boundaries and SCPs that prevent this tool from touching resources outside the intended sandbox accounts. That way, even if the tag filter is too broad, the permissions model still limits the damage.
