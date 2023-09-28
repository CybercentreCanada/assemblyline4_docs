# *ServiceBase* class

There are few exceptions that a service can raise that have unique handling in Assemblyline.

??? info "Always raise an exception if something goes wrong in your service that submits files to external services"
    If you write a service that submits a file to an external service and it fails silently if the external service fails, this result will be cached and then will be returned the next time that the same file is seen in the system.

## NonRecoverableError

This exception is raised to indicate that the analysis of the file is never going to succeed, so *it is not* worth resubmitting the file to try again.

Situations when you should use this exception in your service:

- A file is received by a service that handles submitting that file to an external server like CAPE, Intezer, VirusTotal, etc. The external server fails to analyze the file in a way that provides high confidence that the external server will fail consistently for this file, or any file during a given time frame such as when the REST API is down.

## RecoverableError

This exception is raised to indicate that the analysis of the file is could succeed, so **it is** worth resubmitting the file to try again.

Situations when you should use this exception in your service:

- A file is received by a service that handles submitting that file to an external server like CAPE, Intezer, VirusTotal, Suricata, etc. The external server fails to analyze the file but you have high confidence that the external server won't fail again.

## When to fail silently

Sometimes the above exceptions are not what you are looking for.

Situations when you should catch an error and return no result:

- A file is received by a service which uses a library for analysis. The library crashes when it attempts to analyze this file, and it will always crash for this file.
