# Slack Github Action

Post a rich Slack Block Kit notification for GitHub Actions runs.

It standardizes success/failure formatting, links to your workflow run and commit, and shows release notes when triggered by a release event.

- Color mapping: success → #2FB886, failure → #B22222
- Emojis: ✅ for success, ❌ for failure
- Works with both GitHub-hosted and self-hosted runners

## Features

- Consistent Slack Block Kit layout across repositories and workflows
- Auto links to Workflow Run and Commit
- Short SHA badge, timestamp, repo/actor/event context
- Release-aware: shows tag and “Release notes” when `on: release`
- Simple boolean input `is_success` to drive color and status prefix

## Inputs

- webhook (required): Slack Incoming Webhook URL (pass from caller repository via secrets)
- is_success (required): 'true' or 'false' string to indicate success/failure
- header_text (required): Title text shown as Slack header
- version_number (required): Semantic version (e.g., 1.2.3) when not running on a release event
- build_number (required): Build number (e.g., 42) when not running on a release event
- app_icon_url (optional): Accessory image URL shown on the right in the fields section

Notes:
- GitHub Actions passes composite action inputs as strings. This action compares `is_success` to the literal string 'true'.
- When event is “release”, the tag (`github.ref_name`) is displayed and the second line shows “Release notes: …”.
- When not “release”, it displays `v<version_number>+<build_number>` and “Triggered manually”.

## Usage

### Quick Start (Marketplace)

```yaml
name: Slack Notify (Demo)

on:
  workflow_dispatch:
    inputs:
      is_success:
        description: Whether the run is successful
        type: choice
        options: ["true", "false"]
        default: "true"
        required: true
      version_number:
        description: Version number (e.g., 1.2.3)
        default: "1.0.0"
        required: true
      build_number:
        description: Build number
        default: "1"
        required: true
      header_text:
        description: Header title
        default: Android Release - Play Store
        required: true
      app_icon_url:
        description: Optional icon URL
        required: false

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - name: Notify Slack
        uses: ssiswent/slack-github-action@v1
        with:
          webhook: ${{ secrets.SLACK_LOGS_WEBHOOK }}
          is_success: ${{ inputs.is_success }}          # or ${{ success() }}
          header_text: ${{ inputs.header_text }}
          version_number: ${{ inputs.version_number }}
          build_number: ${{ inputs.build_number }}
          app_icon_url: ${{ inputs.app_icon_url }}
```

### Release Event Example (auto release notes)

```yaml
name: Notify on Release

on:
  release:
    types: [published]

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - name: Notify Slack
        uses: ssiswent/slack-github-action@v1
        with:
          webhook: ${{ secrets.SLACK_LOGS_WEBHOOK }}
          is_success: 'true'                          # or decide based on prior jobs
          header_text: 'Android Release - Play Store'
          version_number: '0.0.0'                     # ignored for release event
          build_number: '0'                           # ignored for release event
```

### Using a Build Job Result

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "build something"

  notify:
    needs: [build]
    runs-on: ubuntu-latest
    steps:
      - name: Notify Slack with build result
        uses: ssiswent/slack-github-action@v1
        with:
          webhook: ${{ secrets.SLACK_LOGS_WEBHOOK }}
          is_success: ${{ needs.build.result == 'success' }}
          header_text: 'Android Release - Play Store'
          version_number: '1.0.0'
          build_number: '42'
```

## Security and Secrets

- This action cannot read caller repository secrets directly.
- Always pass the webhook from the caller workflow: `with.webhook: ${{ secrets.SLACK_LOGS_WEBHOOK }}`.

## Versioning

- Use floating major tags for stability and easy upgrades:
  - `@v1` → latest v1.x.y
- Semantic version tags available (e.g., `@v1.0.0`).

## Troubleshooting

- Missing “shell” in composite steps: Every step with `run:` must specify `shell: bash`.
- Webhook not set: Ensure your repo has `SLACK_LOGS_WEBHOOK` secret and you pass it to `with.webhook`.
- `is_success` truthiness: This action checks string `'true'` explicitly; pass `'true'` or `'false'` as strings or use expressions that resolve to those.
- Timezone: The timestamp uses “Asia/Shanghai”. Fork and adjust if needed.

## License

MIT
