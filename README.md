# Scintillator Action

The official GitHub Action for Scintillator.
Seamlessly offloads VRT artifacts to Cloudflare R2, removing the need for heavy databases or complex storage orchestration.

## Usage

Insert this step into your workflow.
The action handles baseline restoration, artifact uploading, and baseline updates automatically based on the branch context.

```yaml
name: VRT
on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # Setup environment (Node, Browsers, etc.)
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: pnpm install
      - run: npx playwright install chromium --with-deps

      # Run Scintillator
      - name: Visual Regression Test
        uses: cubiream/scintillator-action@v1
        with:
          cf_account_id: ${{ secrets.CF_ACCOUNT_ID }}
          r2_access_key_id: ${{ secrets.R2_ACCESS_KEY_ID }}
          r2_secret_access_key: ${{ secrets.R2_SECRET_ACCESS_KEY }}
          test_command: 'pnpm test:vrt' # Your actual test command
          bucket: 'my-vrt-bucket' # Optional
```

## Inputs

| Input | Required | Description | Default |
| :--- | :---: | :--- | :--- |
| `cf_account_id` | **Yes** | Cloudflare Account ID. | - |
| `r2_access_key_id` | **Yes** | R2 Token Access Key ID. | - |
| `r2_secret_access_key` | **Yes** | R2 Token Secret Access Key. | - |
| `test_command` | **Yes** | Command to execute the VRT process (e.g. `lost-pixel`). | `pnpm test:vrt` |
| `bucket` | No | Target R2 bucket name. | `visual-regression-test` |

## Behavior

1. **Restore**: Downloads the latest baseline images from `references/main` (or your default branch) before the test runs.
2. **Execute**: Runs the user-provided `test_command`.
3. **Upload**: Pushes new snapshots and diff images to `runs/<commit-sha>`. This runs even if the test fails.
4. **Update**: If the workflow is running on the default branch (e.g., `main`), it promotes the current snapshots to be the new baseline.

## License

MIT
