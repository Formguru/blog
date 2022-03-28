---
layout: default
title: Getting Started with Guru
---

# Get Started with Guru in 10 Minutes

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
Performing this exchange ensures you don't need to send your secret credentials on every call, and can
instead use temporary tokens that are easier to revoke if compromised.
The token exchange is performed using a common flow called the [OAuth Client-Credential Flow](https://auth0.com/docs/get-started/authentication-and-authorization-flow/client-credentials-flow).
Here's how to perform the exchange (we use `curl` to demonstrate the calls in this article, but any HTTP request library will work):
```bash
curl -XPOST -H "Content-Type: application/x-www-form-urlencoded" \
 -d "client_id=CLIENT_ID&client_secret=CLIENT_SECRET&audience=https://api.formguru.fitness/&grant_type=client_credentials" \
 https://guru-prod.us.auth0.com/oauth/token
```
The `CLIENT_ID` and `CLIENT_SECRET` are the credentials that you have received from Guru.

This call will respond with a JSON document that looks like this:
```json
{
  "access_token": "abc123",
  "expires_in": 86400,
  "token_type": "Bearer"
}
```
You will need to save the `access_token`. This is the value you will pass in the Authorization header to make calls to Guru.
It is recommended to re-use a token for multiple calls to ensure your integration is fast and available.
Also note the `expires_in` field: this tells you how long (in seconds) the token is valid for.
Once the token expires, you can make the same call to retrieve a new token.

## Uploading a Video
Armed with your `access_token`, you are now ready to upload your first video.
Use your phone to record yourself (or an understanding co-worker) doing a few bodyweight squats.
Guru works best with `mp4`, but can also handle `webm` and `mov` videos if needed.
Once you have the video, you first need to tell Guru about your video:
```bash
curl -XPOST -H "Content-Type: application/json" -H "Authorization: Bearer ACCESS_TOKEN" \
 -d '{"filename": "squat.mp4", "size": 1234, "domain": "weightlifting", "activity": "squat", "repCount": 5, "source": "my-service"}' \
 https://api.getguru.fitness/videos
```
The `ACCESS_TOKEN` is the value you received back from the token exchange.
The body of the request describes your video.
At the very least you will want to update the value of `filename` and `size`, the latter to be the size
of your video in bytes. The remaining fields are optional, but if provided will assist in the quality
of the recognition results.

The response back from this call is a JSON document that will look like:
```json
{
  "fields": {
    "AWSAccessKeyId": "xxxxxxxxxxxxxxxxxxxx",
    "Content-Type": "video/mp4",
    "key": "dev/abc-123.mp4",
    "policy": "xxxxxxxxxxxxxxxxxxxx",
    "signature": "xxxxxxxxxxxxxxxxxxxx",
    "x-amz-security-token": "xxxxxxxxxxxxxxxxxxxx"
  },
  "id": "abc-123",
  "url": "https://upload-url.getguru.fitness"
}
```
Keep the `id` saved for later, this is how you will refer to this video when communicating with Guru's API.

With this information, you're now ready to actually upload the video and begin the analysis process.
You will upload the video to the URL returned in the previous response.
You will post the content as `multipart/form-data`.
Each of the key/value pairs returned in the `fields` object will need to be passed as form fields to ensure
the call is authenticated. You will also specify an extra `file` form field, which is the content of your video.
Tieing that all together, your call will look something like:
```bash
curl -XPOST -H "Content-Type: multipart/form-data" \
 -F 'AWSAccessKeyId=xxxxxxxxxxxxxxxxxxxx' \
 -F 'Content-Type=video/mp4' \
 -F 'key=dev/abc-123.mp4' \
 -F 'policy=xxxxxxxxxxxxxxxxxxxx' \
 -F 'signature=xxxxxxxxxxxxxxxxxxxx' \
 -F 'x-amz-security-token=xxxxxxxxxxxxxxxxxxxx' \
 -F 'file=@/path/to/squat.mp4' \
 https://upload-url.getguru.fitness
```
Congratulations, your video is now being analysed by Guru!

## Fetching the Analysis
Depending on its length the analysis process can take up to a minute. You can poll the status of the analysis using:
```bash
curl -H "Authorization: Bearer xxxxxxxxxxxxxxxxxxxx" https://api.getguru.fitness/videos/abc-123
```
This call will return a response like:
```json
{
  "status": "Success",
  "uri": "https://https://upload-url.getguru.fitness/abc-123.mp4"
}
```
If the returned `status` is `Success` then analysis has completed.
If it is `Pending` then analysis is still running.
The returned `uri` can be used to download the original video if required.

Once analysis has completed you can fetch a high-level summary of the analysis using:
```bash
curl -H "Authorization: Bearer xxxxxxxxxxxxxxxxxxxx" https://api.getguru.fitness/videos/abc-123/analysis
```
This will return a response like:
```json
{
  "status": "Complete",
  "liftType": "SQUAT",
  "reps": [{
    "startTimestampMs": 20,
    "midTimestampMs": 2255,
    "endTimestampMs": 3756,
    "analyses": [{
      "analysisType": "BAR_PATH_SINUOSITY_PERCENTAGE",
      "analysisScalar": 97.15
    }, {
      "analysisType": "HIP_KNEE_ANGLE_DEGREES",
      "analysisScalar": -1.9
    }, {
      "analysisType": "ELBOW_WRIST_ANGLE_DEGREES",
      "analysisScalar": 39.75606106179941
    }]
  }]
}
```
This includes the `liftType` that was detected by our platform, and information on each rep
found in the video. The information for each rep contains  timestamps indicating the start, middle, and
end of the rep within the video. It also contains `analyses`, each one specific to the type of activity
being performed within the video. Play around with different types of videos to see what the platform can do.
We are adding more analysis types all the time!

The platform can provide much deeper information about your video, including time-series 2D positional data for
each joint using the `/videos/<id>/j2p` endpoint, but that's a topic for another blog post. Enjoying playing around with the API and let us know any
queries or comments at support@getguru.fitness!
