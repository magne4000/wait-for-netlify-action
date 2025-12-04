# Wait for Netlify Deployment

Do you have other Github actions (Lighthouse, Cypress, etc) that depend on the Netlify deploy URL? This action will wait until the deploy URL is available before running the next task.

This action uses the Netlify API to always retrieve the correct deployment being built. You will need to generate a [Personal Access Token](https://app.netlify.com/user/applications/personal) to use and pass it as the `NETLIFY_TOKEN` environment variable.

## Env

### `NETLIFY_TOKEN`

**Required.** Your Netlify [Personal Access Token](https://app.netlify.com/user/applications/personal) to use for API access. This should be set as a GitHub secret, see example.

## Inputs

### `site_id`

**Required** The API ID of your site. See Settings > Site Details > General in the Netlify UI

### `max_timeout`

Optional — The amount of time to spend waiting on the Netlify deployment to respond with a success HTTP code after reaching "ready" status. Defaults to 60 seconds.

### `context`

Optional — The Netlify deploy context. Can be `branch-deploy`, `production` or `deploy-preview`. Defaults to all of them.

## Outputs

### `url`

The Netlify deploy preview url that was deployed.

### `deploy_id`

The Netlify deployment ID that was deployed.

## Example usage

Basic Usage

```yaml
steps:
  - name: Hold for Netlify
    uses: magne4000/wait-for-netlify-action@v4.0.1
    id: waitForDeployment
    with:
      site_id: 'YOUR_SITE_ID' # See Settings > Site Details > General in the Netlify UI
    env:
      NETLIFY_TOKEN: ${{ secrets.NETLIFY_TOKEN }}

# Then use it in a later step like:
# ${{ steps.waitForDeployment.outputs.url }}
```

<details>
<summary>Complete example with Cypress</summary>
<br />

```yaml
name: Cypress
on: pull_request
jobs:
  integration:
    runs-on: ubuntu-latest

    jobs:
    cypress:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v2

        - name: Install modules
          run: npm ci

        - name: Hold for Netlify
          uses: magne4000/wait-for-netlify-action@v4.0.1
          id: waitForDeployment
          with:
            site_id: '[your site ID here]'
          env:
            NETLIFY_TOKEN: ${{ secrets.NETLIFY_TOKEN }}

        - name: Run Cypress
          uses: cypress-io/github-action@v2
          with:
            record: true
            config: baseUrl=${{ steps.waitForDeployment.outputs.url }}
          env:
            # pass the Dashboard record key as an environment variable
            CYPRESS_RECORD_KEY: ${{ secrets.CYPRESS_RECORD_KEY }}
            # this is automatically set by GitHub
            GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

</details>

<details>
<summary>Complete example with Lighthouse</summary>
<br />

```yaml
name: Lighthouse

on: pull_request

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v1
      - name: Use Node.js
        uses: actions/setup-node@v1
        with:
          node-version: 16
      - name: Install
        run: |
          npm ci
      - name: Build
        run: |
          npm run build
      - name: Hold for Netlify
        uses: magne4000/wait-for-netlify-action@v4.0.1
        id: waitForNetlifyDeploy
        with:
          site_id: 'YOUR_SITE_ID' # See Settings > Site Details > General in the Netlify UI
        env:
          NETLIFY_TOKEN: ${{ secrets.NETLIFY_TOKEN }}
      - name: Lighthouse CI
        run: |
          npm install -g @lhci/cli@0.3.x
          lhci autorun --upload.target=temporary-public-storage --collect.url=${{ steps.waitForNetlifyDeploy.outputs.url }} || echo "LHCI failed!"
        env:
          LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_GITHUB_APP_TOKEN }}
```

</details>

<small>
This is a heavily-modified fork of [kamranayub/wait-for-netlify-action](https://github.com/kamranayub/wait-for-netlify-action).
</small>
