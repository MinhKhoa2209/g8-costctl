# costctl — XBrain W6 side challenge starter

A starter scaffold for a small AWS-resource-management CLI. **The CLI structure
is built; you implement the command logic.** Fork this repo, fill in the
stubs, make the tests pass, customize for your group, then submit.

> **Side challenge is OPTIONAL and does NOT count toward W6 score or bonus cap.**
> Recognition is separate (Slack callout / Phase 2 selection / portfolio).
> See `outputs/W6/W6_downtime_exercises.md` in the XBrain program repo for
> the full brief.

---

## Current Status

- Group: `G8`
- Repository: `https://github.com/MinhKhoa2209/g8-costctl`
- Test result: `25/25 passing`
- Implemented commands: `list`, `cost`, `terminate`, `tag`, `clean`

---

## What's given vs what you build

| Provided (don't reinvent) | Your job |
|---------------------------|----------|
| `costctl.py` — argparse entrypoint, dispatch table | Implement each command's `run(args)` |
| `commands/_common.py` — `parse_kv`, `tags_to_dict`, `tags_match`, `confirm` | Use these helpers, don't rebuild them |
| `tests/test_common.py` — 10 unit tests for the helpers (all green) | Don't modify — they verify the helpers still work |
| `tests/test_list.py`, `tests/test_terminate.py`, `tests/test_clean.py` — failing tests that define each command's behavior | Make them green |
| Module docstrings in every `commands/*_cmd.py` — full spec, hints, AWS APIs to use | Read them carefully before coding |
| `Makefile`, `requirements*.txt`, `.gitignore`, `LICENSE` | Untouched |
| `sample_output/*_example.txt` | Replace with REAL outputs once your impl works |

**Initial state of `make test`:** 10 passed (helpers), 15 failed (commands).
You're done when all 25 pass.

---

## Quickstart (5 minutes)

```bash
# 1. Fork / clone
git clone <your-fork-url> g8-costctl && cd g8-costctl

# 2. Install
make install-dev                   # or: pip install -r requirements-dev.txt

# 3. Confirm baseline — 10 passed, 15 failed
make test

# 4. Confirm --help works (CLI scaffolding is already wired up)
./costctl.py --help

# 5. Open commands/list_cmd.py and start implementing.
#    The module docstring tells you what to build. Make test_list.py green.
```

Configure AWS credentials when you're ready to run against your account:

```bash
aws configure                      # or set AWS_* env vars
./costctl.py list ec2              # will throw NotImplementedError until you finish step 5
```

---

## Implementation roadmap

Recommended order. You need `list` + at least 2 of the next 3.

### Required

| # | File | Make pass | Time |
|---|------|-----------|------|
| 1 | `commands/list_cmd.py` | `pytest tests/test_list.py` (7 tests) | ~45 min |
| 2 | Pick **at least 2** of: | | |
|   | • `commands/cost_cmd.py` | (no test file — verify manually with `./costctl.py cost --tag X=Y --days 7` and compare to AWS Console) | ~30 min |
|   | • `commands/terminate_cmd.py` | `pytest tests/test_terminate.py` (4 tests) | ~40 min |
|   | • `commands/tag_cmd.py` | (no test — verify with `tag` + `list` roundtrip) | ~30 min |

### Stretch (optional — extra portfolio value)

| File | Make pass | Time |
|------|-----------|------|
| `commands/clean_cmd.py` | `pytest tests/test_clean.py` (4 tests) | ~30 min |
| `commands/idle_cmd.py` | (no test — verify manually) | ~45 min |
| `commands/migrate_gp3_cmd.py` | (no test — verify manually, then run `--apply` once for real) | ~30 min |

### How to read a stub

Every `commands/*_cmd.py` starts with a module docstring that includes:

- **WHAT YOU MUST BUILD** — high-level behavior
- **HELPERS YOU CAN USE** — point to `commands/_common.py`
- **AWS APIS YOU'LL NEED** — exact boto3 calls
- **EXPECTED OUTPUT FORMAT** — copy this exactly when you `print(...)`
- **VERIFY** — pytest command or manual recipe

Don't skip the docstring and jump to `raise NotImplementedError`. The docstring
is your spec.

---

## Commands (final shape after you implement)

| Command | What it does | Tier |
|---------|--------------|------|
| `list <type>` | List EC2/RDS/S3/Volume, filter by tag or missing-tag | core |
| `cost --tag k=v` | Sum cost over N days for resources matching a tag | core |
| `terminate <type> --id` | Terminate/delete one resource (asks confirmation) | core |
| `tag <type> --id --set` | Add/update tags on one resource | core |
| `clean --tag k=v` | Bulk terminate resources by tag (dry-run by default) | stretch |
| `idle` | Find idle EC2 by 24h CPU avg | stretch |
| `migrate-gp3` | Plan or apply gp2 → gp3 EBS migration | stretch |

Resource types: `ec2`, `rds`, `s3`, `volume`.

### Example invocations (after implementing)

```bash
# List
./costctl.py list ec2 --tag Environment=dev
./costctl.py list ec2 --missing-tag Application
./costctl.py list s3

# Cost (data lags 8–24h; if "no cost data", try larger --days)
./costctl.py cost --tag Application=HealthBot --days 7

# Terminate (asks y/N)
./costctl.py terminate ec2 --id i-0abc123
./costctl.py terminate ec2 --id i-0abc123 --force

# Tag
./costctl.py tag ec2 --id i-0abc --set Owner=alice --set Application=HealthBot

# One-liner: fix one missing-tag resource
./costctl.py tag ec2 \
  --id $(./costctl.py list ec2 --missing-tag Application | awk 'NR==4{print $1}') \
  --set Application=HealthBot

# Stretch
./costctl.py clean --tag purpose=practice          # dry-run
./costctl.py clean --tag purpose=practice --apply
./costctl.py idle --threshold 5 --hours 24
./costctl.py migrate-gp3
./costctl.py migrate-gp3 --apply --volume-id vol-0xyz
```

---

## Requirements

- Python 3.11+
- `boto3` (via `make install`)
- AWS credentials with:
  - **Read**: EC2, RDS, S3, CloudWatch, Cost Explorer
  - **Write** (only for `terminate`/`tag`/`clean`/`migrate-gp3`): EC2, RDS, S3

For tests:
- `moto`, `pytest`, `pytest-cov` (via `make install-dev`)

---

## Project structure

```
costctl-starter/
├── costctl.py                # argparse entrypoint (provided)
├── commands/
│   ├── _common.py            # helpers — IMPLEMENTED, leave alone
│   ├── list_cmd.py           # STUB → implement
│   ├── cost_cmd.py           # STUB → implement
│   ├── terminate_cmd.py      # STUB → implement
│   ├── tag_cmd.py            # STUB → implement
│   ├── clean_cmd.py          # STUB → stretch
│   ├── idle_cmd.py           # STUB → stretch
│   └── migrate_gp3_cmd.py    # STUB → stretch
├── tests/                    # ALL provided; some pass, some fail until you implement
│   ├── conftest.py
│   ├── test_common.py        # 10 tests, green from day 1
│   ├── test_list.py          # 7 tests — implement list_cmd to green these
│   ├── test_terminate.py     # 4 tests
│   └── test_clean.py         # 4 tests (stretch)
├── sample_output/            # example outputs — replace with yours
├── Makefile
├── requirements.txt
├── requirements-dev.txt
├── LICENSE
└── README.md (this file)
```

---

## TDD loop

```bash
# 1. Pick a failing test
pytest tests/test_list.py::test_list_ec2_empty -v

# 2. Open commands/list_cmd.py, find the function it references (_list_ec2)
# 3. Implement just enough to make THAT test pass
# 4. Re-run — green?
pytest tests/test_list.py::test_list_ec2_empty -v

# 5. Move to the next test in the file
pytest tests/test_list.py::test_list_ec2_no_filter_returns_all -v

# Repeat. When all tests in the file pass, that command is done.
```

The provided tests use [moto](https://github.com/getmoto/moto) — no real AWS
calls, no charges, runs in seconds.

---

## How to extend — add a new command

Say you want a `snapshot` command that creates an EBS snapshot.

**1.** Create `commands/snapshot_cmd.py`:

```python
"""snapshot — create an EBS snapshot of one volume."""
import boto3

def run(args):
    ec2 = boto3.client("ec2")
    resp = ec2.create_snapshot(
        VolumeId=args.volume_id,
        Description=f"costctl backup of {args.volume_id}",
    )
    print(f"Created snapshot {resp['SnapshotId']} (state: {resp['State']})")
```

**2.** Add parser block in `costctl.py` `build_parser()`:

```python
sn = sub.add_parser("snapshot", help="snapshot an EBS volume")
sn.add_argument("--volume-id", required=True)
```

**3.** Register in `CMD_MODULE`:

```python
CMD_MODULE = {
    ...,
    "snapshot": "snapshot_cmd",
}
```

Run:

```bash
./costctl.py snapshot --volume-id vol-0xyz
```

Add `tests/test_snapshot.py` mirroring `test_list.py`. Moto supports
`create_snapshot` out of the box.

---

## Reflections (paste 2+ before submission)

Add a `REFLECTIONS.md` to your repo. Sample prompts:

1. **Multi-account**: To run `costctl` against 100 AWS accounts (not just yours),
   what changes? Cross-account roles? Profile loop? Aggregated CSV per account?
2. **`idle` vs Trusted Advisor**: `idle` uses a 24h CPU window. Trusted Advisor
   uses 14 days. When do you trust `idle` more, when do you trust TA more?
3. **`clean --apply` blast radius**: If you accidentally ran
   `clean --tag Environment=dev --apply` in an account shared with another
   team, what would you have wanted in place to limit damage?
4. **AI assistance**: What fraction of code came from AI tools (Claude /
   Cursor / Copilot) unmodified? Which parts did you actively modify, why?
5. **W7 carry-over**: Which commands will you keep going into W7
   (production-style multi-account)? Which would you drop and why?

---

## Submission checklist (W6 side challenge)

- [x] Fork → rename to `g8-costctl` → clone locally
- [x] `make install-dev && make test` shows 10 passed at start
- [x] Implement `list` → `pytest tests/test_list.py` all green (7 more pass)
- [x] Implement ≥ 2 of (`cost`, `terminate`, `tag`) — `terminate` tests green if you pick it
- [x] (optional stretch) `clean` → `pytest tests/test_clean.py` green; or `idle` / `migrate-gp3`
- [x] `make test` final score reported in README (e.g. "21/25 passing")
- [x] Replace `sample_output/*_example.txt` with real outputs from your account
- [x] `REFLECTIONS.md` with 2+ answers
- [x] At least 3 meaningful commits (init → first command working → final polish)
- [x] Replace `g8` placeholders throughout README with your real group number
- [x] Add Team section with member names
- [x] Tag: `git tag w6-sidechallenge-v1 && git push --tags`
- [x] Post link in Slack `#w6-sidechallenge` thread:
      `G8 — https://github.com/MinhKhoa2209/g8-costctl — 25/25 tests passing — implemented: list, cost, terminate, tag, clean`

Reminder: **OPTIONAL and does NOT count toward W6 score.** Recognition is
separate (Slack callout / Phase 2 selection / portfolio).

---

## License

MIT — see `LICENSE`.

---

## Team

- Dinh Minh Khoa - SID: XB-DN26-125


---

*Starter scaffold from the XBrain W6 side challenge —
`outputs/W6/costctl-starter/` in the program repo.*
