# sample_output/

These files now contain **real outputs** captured from the workshop account in
region `us-west-2` on `2026-05-22`.

Files:

- `list_ec2_example.txt` — output of `./costctl.py --region us-west-2 list ec2`
- `list_ec2_missing_app_example.txt` — output of `./costctl.py --region us-west-2 list ec2 --missing-tag Application`
- `cost_example.txt` — output of `./costctl.py --region us-west-2 cost --tag Application=FoodieDash --days 7`

These values can drift over time as resources and costs change in the account.
