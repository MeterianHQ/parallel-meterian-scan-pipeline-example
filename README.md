# parallel-meterian-scan-pipeline-example

On this branch, upon trigger the [**example pipeline**](.github/workflows/main.yaml) runs a primary job named `primary_routine` in parallel with the `meterian_scan` job.
Failure of the `meterian_scan` job won't affect the execution of the `primary_routine` job.
Should the `meterian_scan` job fail an automated email from GitHub will alert the maintainers of the repository.

```
name: Sample workflow

on: push:

jobs:
  primary_routine:
    name: Primary routine
    runs-on: ubuntu-latest
    steps:
      - run: |
             echo "Running primary routine..."
             sleep 30
             echo "Done"
  
  meterian_scan:
    name: Meterian Scan
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v3

      - name: Run Meterian scan
        uses: MeterianHQ/meterian-github-action@v1.0.13
        env:
          METERIAN_API_TOKEN: ${{ secrets.METERIAN_API_TOKEN }}
   ```
