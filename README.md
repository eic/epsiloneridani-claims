# epsiloneridani-claims

This repository holds no code. It exists so that [EpsilonEridani](https://github.com/eic/EpsilonEridani)
workers run by different operators can avoid doing the same job twice.

A worker takes a lease before it starts a task by pushing a custom git ref
`refs/epsiloneridani-claims/<key>` here, pointing at an orphan commit whose message is a small JSON
document `{owner, expires_at, ...}`. Acquire, renew, takeover and release are all one atomic
GitHub operation, `git push --force-with-lease`, so a race has exactly one winner. Leases carry a
TTL, so a worker that dies never blocks anyone. The mechanism is
[`scripts/claim.sh`](https://github.com/eic/EpsilonEridaniWorker/blob/main/scripts/claim.sh) in
EpsilonEridaniWorker, and the namespace it picks is described in
[the claim namespace](https://github.com/eic/EpsilonEridaniWorker/blob/main/docs/reference.md#the-claim-namespace).

Claims are cooperative. Honouring one only avoids duplicate work; nothing here protects a branch.
That guarantee comes from the `--force-with-lease` compare-and-swap each worker does on its own
push, which does not depend on claims at all.

Push access is granted automatically to anyone who has had a pull request merged into EpsilonEridani, by
the `claims-access` workflow in that repository. It is deliberately a separate repository from
EpsilonEridani: coordinating workers should never require write access to the library itself.

Two housekeeping notes. Expired leases are swept periodically, so a stray ref is not a problem.
Actions are disabled here, and must stay that way: everyone who coordinates through this repository
can push to it, and a repository with both write access and CI is a repository where anyone can run
anything.
