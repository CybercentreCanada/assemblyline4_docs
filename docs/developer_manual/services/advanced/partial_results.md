# Partial Results

Producing a partial result is an optional feature that services can use to delay processing while waiting for other services to provide additional information.

This additional information can come from other files within the same submission, even after the file being processed has already been otherwise completed.

## Service Development

When a service produces a result it may optionally flag it as 'partial'. This signals to assemblyline that the service may be able to do further processing. A hypothetical service that expects a password to be provided might have sections that looks like this:

```python
class IncompleteService(ServiceBase):
    
    def execute(self, request: ServiceRequest):
        # prepare a result
        result = Result()
        request.result = result

        try:
            passwords = self.get_passwords(request)
            self.process_password_protected_data(request, passwords)
        except PasswordNotFoundError:
            request.partial()
            section = ResultTextSection(
                "Failed to extract password protected file.", 
                heuristic=Heuristic(1), 
                parent=request.result
            )
            section.add_tag("file.behavior", "Archive Unknown Password")
            return


    def get_passwords(self, request: ServiceRequest):
        # Start with default passwords from the service config
        passwords = list(self.config.get("default_password_list", []))

        # Get passwords users may have added to this submission
        user_supplied = request.get_param("password")
        if user_supplied:
            passwords.append(user_supplied)

        # Get passwords from temp data, other services might be providing some
        if "passwords" in request.temp_submission_data:
            passwords.extend(request.temp_submission_data["passwords"])

        return passwords
```

Further processing will be triggered by services producing changes in monitored keys in the temporary submission data. What keys can be monitored is configured per system, which keys a service monitors is configured per service, both must be set for this feature to function. Our example service that wants to wait for other services to update the list of possible passwords related to the submission would add this to its manifest:

```yaml
uses_temp_submission_data: true
monitored_keys:
  - passwords
```

## System Configuration

The set of keys that can be used by services for monitoring must be configured at the system level. New fields can be added by setting them under the `submission.temporary_keys` configuration field.

What aggrigation will be used to combine temporary data from different services must be set for each key. Available options are:
 - `union` - Keep this key as a submission wide list merging equal items
 - `overwrite` - Keep this key submission wide on a "last write wins" basis

To add the temporary submission data keys 'sample_key_a' and 'sample_key_b' with the union and overwrite aggrigations respectively the following could be used:

```yaml
submission:
    temporary_keys:
        sample_key_a: union
        sample_key_b: overwrite
```

### Defaults

By default all systems are configured with some keys active:
 - `passwords`: union, a list of possible passwords related to this submission extracted from different parts of the submitted file.
 - `email_body`: union, a list of words found in email bodies embedded into this submission.

Adding more values via `submission.temporary_keys` won't overwrite these defaults or prevent future defaults from being added.

If you wish to remove these defaults, or block future changes to them, you can overwrite the `submission.default_temporary_keys` field.

```yaml
submission:
    default_temporary_keys: {}
```

