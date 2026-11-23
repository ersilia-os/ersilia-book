# Model Hub maintenance

Models that worked when they were incorporated can stop working later: a dependency changes, an upstream repository moves, or a Docker image breaks on a new architecture. The [ersilia-maintenance](https://github.com/ersilia-os/ersilia-maintenance) repository checks the whole Hub regularly, so problems are found before users find them. Its workflows test models, keep their metadata in sync, and publish reports that the team reviews every week.

## Scheduled workflows

These run automatically. All times are UTC.

<table><thead><tr><th width="186.328125">Workflow</th><th width="173.87890625">Runs</th><th>What it does</th></tr></thead><tbody><tr><td>Test Models (<code>test_models.yml</code>)</td><td>Mondays at 10:00</td><td>Picks 10 models and tests them with <code>ersilia test --shallow --from_github</code>. Models never tested come first, then those tested longest ago. Models with open issues are skipped. For each model that fails, it opens a "Failed weekly maintenance tests" issue in the model's repository, or comments on the existing one, and sets the model's status to <code>In maintenance</code>.</td></tr><tr><td>Generate Failing Models Report (<code>generate_failing_models_report.yml</code>)</td><td>Mondays at 06:00, and after every test run</td><td>Lists every model whose last test failed.</td></tr><tr><td>Repo Data Sync (<code>repo_data_sync.yml</code>)</td><td>Every 10 days</td><td>Collects the metadata, open issues, and last test, release, and packaging dates of every model, and updates the model report.</td></tr><tr><td>Monthly Health Report (<code>hub_health_status.yml</code>)</td><td>The 1st of every month at 01:00</td><td>Calculates Hub-wide health metrics and trends, with plots.</td></tr><tr><td>Models scan (<code>models_scan.yml</code>)</td><td>The 1st of every month at 02:00</td><td>Compares the model repositories on GitHub with the models in Airtable, and opens an issue for each one missing from either.</td></tr><tr><td>Check Repo Updates (<code>check_repo_updates.yml</code>)</td><td>The 1st of every month at 08:00</td><td>Finds models whose original source code, on GitHub or PyPI, has changed since the model was last packaged.</td></tr></tbody></table>

## Workflows run by hand

Maintainers start these from the repository's Actions tab when they need them:

<table><thead><tr><th width="266.890625">Workflow</th><th>What it does</th></tr></thead><tbody><tr><td>Test Single Model (<code>single_model_test.yml</code>)</td><td>Tests one model with <code>ersilia test --shallow --from_github</code>. If it passes, it sets the model's status back to <code>Ready</code> and closes its maintenance issue. Run this after fixing a model.</td></tr><tr><td>Test model from source (<code>test-model-source.yml</code>)</td><td>Tests a model from a branch of its repository, before the fix is merged.</td></tr><tr><td>Test model image (<code>test-model-image.yml</code>, <code>test-model-image-arm64.yml</code>)</td><td>Tests a model's Docker image with <code>ersilia test --deep</code>, on AMD64 or ARM64, with Ersilia installed from any branch.</td></tr><tr><td>Update_metadata (<code>update_metadata.yml</code>, <code>update_metadata_batch.yml</code>)</td><td>Syncs the metadata of one model, or a list of models, to Airtable and the catalog's S3 JSON file.</td></tr><tr><td>Update Publication Links (<code>update_publication_links.yml</code>)</td><td>Updates the publication links of models from a list of DOIs. Runs as a dry run by default.</td></tr><tr><td>Check Missing Contributor (<code>check_missing_contributor.yml</code>)</td><td>Lists models without a <code>Contributor</code> field in their metadata.</td></tr></tbody></table>

The repository also has workflows for debugging specific problems, such as running an ARM64 image directly or testing a fix in an Ersilia branch. They're meant for one-off investigations, not regular maintenance.

## Reports

The workflows publish these reports in the repository's [reports folder](https://github.com/ersilia-os/ersilia-maintenance/tree/main/reports):

| Report                                                                                                                | Updated                            | Shows                                                                                             |
| --------------------------------------------------------------------------------------------------------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------- |
| [Weekly model testing](https://github.com/ersilia-os/ersilia-maintenance/blob/main/reports/weekly_model_testing.md)   | Every Monday                       | The results of the week's tests.                                                                  |
| [Failing models](https://github.com/ersilia-os/ersilia-maintenance/blob/main/reports/failing_models.md)               | Every Monday, and after every test | Every model whose last test failed.                                                               |
| [Model report](https://github.com/ersilia-os/ersilia-maintenance/blob/main/reports/model_report.md)                   | Every 10 days                      | All models, with their metadata, status, and open issues.                                         |
| [Monthly health report](https://github.com/ersilia-os/ersilia-maintenance/blob/main/reports/monthly_health_report.md) | Every month                        | Hub-wide health metrics and how they change over time.                                            |
| [Updated source code](https://github.com/ersilia-os/ersilia-maintenance/blob/main/reports/updated_models.md)          | Every month                        | Models whose original source code changed after they were last packaged, and might need updating. |

## Maintained inputs

[ersilia-model-hub-maintained-inputs](https://github.com/ersilia-os/ersilia-model-hub-maintained-inputs) holds the reference set of molecules that Ersilia uses for its precalculations, so every model's stored predictions cover the same inputs. The [CLI testing playground](../testing-and-quality/cli-testing.md) also takes its example input from it. Its large files are stored with [eosvc](../data-storage/ersilia-version-control.md).
