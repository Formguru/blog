---
layout: post
title: Getting Started with Guru
---

The Guru API makes it simple to extract data and insights from video. You can upload someone performing a barbell squat
and have time-series 2D positional joint data and meaningful form feedback analytics returned within seconds.
Once you have an authentication token, you can be up and running with the API in under 10 minutes. This guide will show
you how, let's go!

If you'd rather dive straight into the API docs, you can find them [here](https://docs.getguru.fitness).

## Authentication
Before you can make a call you will need to authenticate with the API.
Begin by contacting Guru to [request access](mailto:support@getguru.fitness?subject=Access+Request).
This process is quick and painless and ensures we can meet your needs.

Once you receive your client credentials you will want to store them in a safe place, accessible by your code.
We use [AWS Secrets Manager](https://aws.amazon.com/secrets-manager), but there are many other alternatives
available.

With the credentials stored, it's time to exchange the credentials for an access token.
Performing this exchange ensures you don't need to send your secret credentials on every call, and
instead use temporary tokens that are easier to revoke if compromised.
The token exchange is performed using a very common flow called the [OAuth Client-Credential Flow](https://auth0.com/docs/get-started/authentication-and-authorization-flow/client-credentials-flow).
Here's how to perform the exchange (we use `curl` to demonstrate the calls in this article, but any HTTP request library will work):
```bash
curl -XPOST -H "Content-Type: application/x-www-form-urlencoded" \
 -d "client_id=CLIENT_ID&client_secret=CLIENT_SECRET&audience=https://api.formguru.fitness/&grant_type=client_credentials" \
 https://guru-prod.us.auth0.com/oauth/token
```
The `CLIENT_ID` and `CLIENT_SECRET` are the credentials that you will have received from Guru.

This call will respond with a JSON document that looks like this:
```json
{
  "access_token": "abc123",
  "expires_in": 86400,
  "token_type": "Bearer"
}
```
You will need to save the `access_token`, this is the value you will pass in the Authorization header to make calls to Guru.
It is recommended to re-use a token for multiple calls to ensure your integration is fast and available.
Also note the `expires_in` field: this tells you how long (in seconds) the token is valid for.
Once the token expires, you can make the same call to retrieve a new token.
