# VuePress on Netlify

How to deploy a VuePress site to Netlify using a GitHub Action.

## Requirements

- A working VuePress site in a GitHub repository
- Netlify account

## Preparation

- Run `yarn docs:build`

This will create a *.dist* directory in the GitHib repository under *docs/.vuepress*

## Netlify

1. Login to Netlify
2. Click on "Create new site from Git"
3. In the bottom of the page, drop the **dist** folder into the upload field
4. Create a new token and use that token for creating a new "secret" on GitHub (see below)

## GitHub

### Settings

1. Create new "secret" (name: NETLIFY_AUTH_TOKEN) with the newly created Netlify token
2. Create a second secret (name: NETLIFY_SITE_ID) with the API ID of your new site in Netlify

### GitHub Action

```yaml
name: Deploy

on:
  # Trigger the workflow on pull request,
  # but only for the main branch for changes in /docs
  pull_request:
    types: [ closed ]
    branches:
      - main
    paths:
      - 'docs/**'

jobs:
  build:
    # This job will only run if the PR has been merged
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    name: Checkout and build
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '14'

      - name: Cache Node Modules
        id: cache-node-modules
        uses: actions/cache@v2
        with:
          path: node_modules
          key: node-modules-${{ hashFiles('yarn.lock')}}

      - name: Install
        working-directory: .
        if: steps.cache.outputs.cache-hit != 'true'
        run: yarn install

      - name: Build
        working-directory: .
        run: yarn docs:build

      - name: Deploy to Netlify
        uses: nwtgck/actions-netlify@v1.2
        with:
          publish-dir: 'docs/.vuepress/dist'
          production-branch: main
          production-deploy: true
          github-token: ${{ secrets.GITHUB_TOKEN }}
          deploy-message: "Deploy from GitHub Actions"
          enable-pull-request-comment: false
          enable-commit-comment: true
          overwrites-pull-request-comment: true
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
```