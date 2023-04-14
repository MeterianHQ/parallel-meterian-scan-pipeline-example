# parallel-meterian-scan-pipeline-example

On this branch, upon trigger the [**example pipeline**](.github/workflows/main.yaml) runs a primary job named `primary_routine` and a test and deployment job `test_n_deployment` in parallel with the `meterian_scan` job.
The `test_n_deployment` job depends on the successful run of the `primary_routine` job in order to run.
Failure of the Meterian scan won't cause the job to fail and won't affect the execution of the either of the other jobs.
Instead the exit code of it's execution is captured and is used to send a custom email to a pre-configured email address.
With this setup the `meterian_scan` job will never report a failure.

```
name: Sample workflow

on: push

jobs:
  primary_routine:
    name: Primary routine
    runs-on: ubuntu-latest
    steps:
      - run: |
             echo "Running primary routine..."
             sleep 30
             echo "Done"
  test_n_deployment:
    needs: [primary_routine]
    name: Test and deployment
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v3
 
      - name: "Test"
        run: mvn test

      - name: "Deploy"
        run: |
             echo "Deploying..."
             sleep 10
             echo "Done"             
  
  meterian_scan:
    name: Meterian Scan
    runs-on: ubuntu-latest
    container:
      image: meterian/cli:latest-gha
    steps:
      - name: Checkout repo
        uses: actions/checkout@v3

      - name: Run Meterian scan
        run: meterian_github_action.sh v1.0.13 || echo "meterian_scan_result=$?" >> $GITHUB_ENV
        env:
          METERIAN_API_TOKEN: ${{ secrets.METERIAN_API_TOKEN }}

      - name: Non-blocking scan failure email alert
        if: ${{ env.meterian_scan_result != 0 }}
        uses: cinotify/github-action@main
        with:
          to: ${{ secrets.NOTIFICATION_EMAIL }}
          subject: 'Meterian scan failure on ${{github.repository}}#${{github.ref_name}}'
          body: 'The Meterian scan on branch ${{github.ref_name}} of repo ${{github.repository}} has failed. Please review it.'
          
```

**Note:** for simplicity this example uses the action `cinotify/github-action@main` to send email without requiring configuration for an SMTP server.
Should you wish to use one instead there are pleanty of actions that can be used instead on the GitHub marketplace.
