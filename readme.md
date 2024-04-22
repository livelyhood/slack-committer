
[<img src="https://github-ads.s3.eu-central-1.amazonaws.com/support-ukraine.svg?t=1" />](https://supportukrainenow.org)

# Slack committer action

This action transforms the github committer of the current commit into a slack [userID](https://www.workast.com/help/article/how-to-find-a-slack-user-id/) or [channelID](https://help.socialintents.com/article/148-how-to-find-your-slack-team-id-and-slack-channel-id).

The mapping provided as JSON. The "fallback" is used when no user can be determined. The github username look up ignores cases.

Based on [spatie/slack-committer](https://github.com/spatie/slack-committer)

## Example usage with JSON provided as string

```yaml
- name: Resolve slack committer with JSON provided as string
  id: slack-committer
  uses: penchef/slack-committer@v1.2
    with:
    # JSON mapping from Github user to slack userID or channelID. "fallback" is used when no user was found.
    user-mapping: >
      {
        "Penchef": "UUSAQBVDZ",
        "fallback": "XYZXYZXYZ"
      }
```

## Example usage with JSON provided as users.json file

Note: this won't work with reusable workflows.

```yaml
- name: Resolve slack committer with JSON provided as users.json file
  id: read-users
  run:  echo users=$(jq . users.json) >> $GITHUB_OUTPUT
- name: Resolve slack committer
  id: slack-committer
  uses: penchef/slack-committer@v1.2
  with:
    user-mapping: ${{ steps.read-users.outputs.users }}
```

## Later in your workflow:

```yml
- name: Notify
  id: slack
  uses: slackapi/slack-github-action@v1.25.0
  env:
    SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
  with:
    # Slack channel id, channel name, or user id to post message.
    # See also: https://api.slack.com/methods/chat.postMessage#channels
    channel-id: ${{ inputs.notifyee || steps.slack-committer.outputs.username }}
    # see slack's attachment api: https://api.slack.com/reference/messaging/attachments
    payload: |
      {
         "attachments": [
              {
                  "mrkdwn_in": ["fields", "title"],
                  "color": "${{ ( (inputs.failure || contains(inputs.status, 'fail')) && '#a63647' ) || '#36a67d' }}",
                  "title": "${{ env.TITLE }}",
                  "title_link": "${{ env.TITLE_LINK }}",
                  "fields": [
                    {
                      "title": "Repo",
                      "value": "<${{ github.event.repository.html_url }}|${{ github.event.repository.name }}>",
                      "short": true
                    },
                    {
                      "title": "Workflow",
                      "value": "<${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}|${{ github.workflow }}>",
                      "short": true
                    },
                    {
                      "title": "Status",
                      "value": "${{ inputs.status || (inputs.failure && 'FAILED') || (!inputs.failure && 'SUCCESS') }}",
                      "short": true
                    },
                    {
                      "title": "Ref",
                      "value": "<${{ github.event.pull_request.html_url || github.event.head_commit.url }}|${{ github.head_ref || github.ref_name }}>",
                      "short": true
                    },
                    {
                      "title": "Event",
                      "value": "${{ github.event_name }}",
                      "short": true
                    }
                  ]
              }
            ]
      }

```

## Advance usage:

In order to avoid repetition, one can combine the three steps

1. determine slack username
2. notify committer

within one reusable workflow

1. create a new re-usable workflow by copying [notify.yml](./.github/workflows/notify.yml), either within the repo you want to use it or within a public repository. See also [re-useable workflow limitation](https://docs.github.com/en/actions/using-workflows/reusing-workflows#limitations)
2. create a github user mapping JSON, e.g. [users.json](users.json).
3. create a new job within the workflow you want to report on, e.g. see [caller.yml](.github/workflows/caller.yml)


