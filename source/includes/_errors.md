# Errors

The UNLOC API uses the following error codes:

Error Code | Meaning
---------- | -------
400 | Bad Request -- Your request is malformed
401 | Authentication failure -- You do not have permissions to act on the resource
403 | Forbidden -- Several requests with invalid credentials within a short time span has been detected: Authentication attempts (even with valid credentials) will be temporarily rejected.
404 | Not Found -- The lock/key was not found
405 | Method Not Allowed -- You tried to access a resource with an invalid method
406 | Not Acceptable -- You requested a format that isn't json
409 | Conflict -- For example, a key that has already been deleted was attempted deleted again
422 | Unprocessable Entity -- A parameter was malformed, for example an invalid phone number
429 | Too Many Requests -- Please slow down
500 | Internal Server Error -- We are having problems
503 | Service Unavailable -- We're temporarially offline for maintanance. Please try again later.
