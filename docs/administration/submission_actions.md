# Submission Actions

Assemblyline can be configured to take a number of actions with completed submissions. This can be because a submission has been processed, or because an ingested submission is completed as a cache hit. The following actions can be taken:

 - creating an alert
 - resubmitting the submission to more services
 - calling a webhook

The configuration of actions is under "Post-process actions" of the administration navigation menu of Assemblyline. The actions configuration is a yaml formatted dictionary with arbitrary keys and values formatted as [PostprocessingAction](../../odm/models/actions/#postprocessaction) objects.


## Selecting submissions for action

Depending on the context where the submission is being tested for an action different terms are available for the Lucene query in the `filter` field. If only `run_on_completed` is set, the `filter` field may use any part of the [Submission](../../odm/models/submission/#submission) or [Tagging](../../odm/models/tagging/#tagging) objects where tag fields are prefixed with "tags.". When `run_on_cache` is set, irrespective of the value of `run_on_completed`, the search in `filter` may _only_ use the following fields:

 - `sid`
 - `max_score`
 - `files.*`
 - `metadata.*`
 - `params.*`

## Action configuration

### Alert

There are no sub-configuration fields of the `raise_alert` field, it is either true or false.

### Resubmit

The `resubmit` action, when not null, must be a [ResubmitOptions](../../odm/models/actions/#resubmitoptions) object. The `additional_services` field is a list of services to resubmit to _in addition to_ the services given by a submission's `params.services.resubmit` parameter provided at submission or ingestion. The `random_below` parameter lets you further filter selected submissions by their max score, only randomly accepting submissions with score between 0 and the given value. The distribution is exponential with low scoring submissions being ignored more often.

### Webhook

The `webhook` action will call a webhook url with a body holding a json object with the fields:

 - `is_cache`: True if this action is triggered by a submission cache hit in ingester.
 - `score`: The score of the submission.
 - `submission`: A [Submission](../../odm/models/submission/#submission) object for processed submissions, or a different [Submission](../../odm/messages/submission/#submission) object for cache hit actions.

The `webhook` field must be a [Webhook](../../odm/models/actions/#webhook) object.

## Default actions

By default Assemblyline comes with two actions defined:

```yaml
default_alerts:
  enabled: true
  filter: 'max_score: >=500'
  raise_alert: true
  resubmit: null
  run_on_cache: true
  run_on_completed: true
  webhook: null
default_resubmit:
  enabled: true
  filter: 'max_score: >=0'
  raise_alert: false
  resubmit:
    additional_services: []
    random_below: 500
  run_on_cache: false
  run_on_completed: true
  webhook: null
```

The action named `default_alerts` applies to all submissions (both cache and non-cache) where the score is 500 or more; both `resubmit` and `webhook` are disabled on this action, and `raise_alert` is active.

The action named `default_resubmit` applies on completion to submissions that are processed which scores zero or more. `webhook` and `raise_alert` are disabled, and `resubmit` is enabled with the following settings: `additional_services` is set to an empty list, so only the submission's own resubmit service list is used; `random_below` is set to 500, so submissions with a `max_score` between 0 and 500 will only be randomly resubmitted.
