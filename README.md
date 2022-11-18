# Adding an _optional_ boolean input to a GitHub workflow breaks the `/dispatches` API clients

The GitHub REST API allows users to create [workflow dispatch events](https://docs.github.com/en/rest/actions/workflows#create-a-workflow-dispatch-event) programmatically. For example, the `test.yaml` workflow in this repo can be dispatched with

```bash
curl \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  https://api.github.com/repos/vlukashov/boolean-workflow-input-issue/actions/workflows/test.yaml/dispatches \
  -d '{"ref":"main"}'
```

The `test.yaml` workflow has an optional string input `name`:

```yaml
inputs:
  name:
    description: An optional string workflow input
    type: string
```

The `/dispatches` REST API works regardless of whether or not the `name` input is provided.

```bash
curl \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  https://api.github.com/repos/vlukashov/boolean-workflow-input-issue/actions/workflows/test.yaml/dispatches \
  -d '{"ref":"main","inputs":{"name":"Mona the Octocat"}}'
```

## The issue: adding an _optional_ boolean workflow input causes these API calls to fail

The `with-boolean-input` branch of this repo adds an optional boolean input to the `test.yaml` workflow:

```yaml
has_admin_role:
  description: An optional boolean workflow input
  type: boolean
```

With that, the API calls that used to work now fail:

```bash
curl \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  https://api.github.com/repos/vlukashov/boolean-workflow-input-issue/actions/workflows/test.yaml/dispatches \
  -d '{"ref":"with-boolean-input","inputs":{"name":"Mona the Octocat"}}'

{
  "message": "Provided value '' for input 'has_admin_role' not in the list of allowed values",
  "documentation_url": "https://docs.github.com/rest/reference/actions#create-a-workflow-dispatch-event"
}
```

The API call fails in the same way regardless whether or not it includes the `inputs` parameter at all.

```bash
curl \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  https://api.github.com/repos/vlukashov/boolean-workflow-input-issue/actions/workflows/test.yaml/dispatches \
  -d '{"ref":"with-boolean-input"}'

{
  "message": "Provided value '' for input 'has_admin_role' not in the list of allowed values",
  "documentation_url": "https://docs.github.com/rest/reference/actions#create-a-workflow-dispatch-event"
}
```

## Expected behaviour

Adding any optional workflow inputs should not cause working API calls to fail.

## Workaround

Do not use the `boolean` input type.
Instead, use the `choice` type with `true` and `false` options, and `false` as the default value.
