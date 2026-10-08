# Postman collection

Requests for the model query endpoints in [`openapi.yaml`](../openapi.yaml), used to retest the API Gateway (HP2-2165).

## Setup

1. Import `HighPeak-API.postman_collection.json` and the three `HighPeak-*.postman_environment.json` files.
2. Select an environment and set `apiKey` to the gateway key for that stage (`HighPeakEnterprise-key-<env>`).
3. Set `bearerToken` to a Cognito ID or access token for a user in the `Admins` or `EnterpriseManagers` group. The model Lambdas check this token themselves through `highpeak-auth-layer`, even behind the API-key gateway. The collection adds `Authorization: Bearer <token>` to every request except the Auth tests.

`apiKey` and `bearerToken` are secret variables. Put their values in Postman's *current value* only and never commit them.

Use **Import** to load these files. Don't open this folder as a Postman local workspace: Postman will convert the files to YAML in place and save the environment values, including keys and tokens, into the repo. Keep the local workspace outside the repo, for example in `~/postman/highpeak`.

## Hosts

| Host | Auth | Unauthenticated response |
| --- | --- | --- |
| `api.key.<env>.highpeak.ai` | API key (`X-API-Key`) plus a Cognito bearer token, which the Lambda checks | 403 without a key; 401 with a key but no token |
| `api.<env>.highpeak.ai` | Cognito bearer token | 401 |

The environments point at the `api.key.*` hosts. To test the Cognito hosts, change `baseUrl` and set `bearerToken`.

## Folders

- **Happy path**: each endpoint returns 200 with the response shape from the spec.
- **Auth**: requests with no key or an invalid key are rejected with 401 or 403. A request with a valid key but no token gets a 401 from the Lambda.
- **Validation**: requests with missing required fields, a bad `overall_rating` value or malformed JSON are rejected. The query Lambdas currently return 500 for invalid input, not 400, so these tests accept either.

The gateway returns the same generic error message for every error, and an unknown path also returns 403. So a passing Auth test only proves the request was rejected, not why. Run **Happy path** with a valid key first to confirm the routes exist.

## Running from the CLI

```sh
npx newman run HighPeak-API.postman_collection.json \
  -e HighPeak-QA.postman_environment.json \
  --env-var apiKey="$API_QA_GATEWAY_KEY"
```
