# Sending Heroku logs to AWS Cloudwatch

## Rationale

While Heroku do provide some basic logging functionality that you can access through the GUI or CLI (`heroku logs -a <appname>`), it's limited to a buffer of only 1500 lines. As the log buffer contains logs for both the Heroku networking layer (prefixed with `heroku router`) and the actual application logs, this can result in a very short log horizon if your logging is in any way verbose. Typically our oldest available logs in the Heroku buffer are less than six hours old.

[Heroku's stated reason](https://devcenter.heroku.com/articles/log-drains) for providing such a small log buffer is that they expect users to "drain" the log data into a separate, dedicated log storage system that will retain the logs long-term. Heroku do not provide anything like this off-the-shelf, and although there are a number of paid third-party addons that claim to do the job we do not want to overcomplicate our Heroku deployment.

Given that we already use AWS for a number of functions, if we want long-term log storage it makes sense to write a Lambda function that will ingest the logs into AWS Cloudwatch, which is Amazon's logging and metrics service.

### Why does this document exist?

Because while there are a few pieces of code in public Github repositories that provide a basic example of how shipping logs from Heroku to Cloudwatch via Lambda works (see [here](https://github.com/rwilcox/heroku_cloudwatch_sync/blob/master/src/heroku_sync_to_cloudwatch.py), there are others available with some basic googling), attempting to adapt this example code into an actual working Lambda function that didn't scream blue murder in our alerting channel took ~~four~~ five rounds of refactoring. The learnings are too numerous to be captured in comments in the function code, so I'm writing them down here.

## Basic operation

In theory, very simple:

- Write a Lambda function handler in Python capable of parsing Heroku log data and forwarding it on to Cloudwatch
- Connect Heroku up to the Lambda function by using the Heroku CLI to add the Lambda function URL as a log drain: `heroku drains:add <function url> -a <appname>`.

In practice, very difficult.

**Note**: After a lot of trial and error on our side to get this to work in December 2022, Amazon pushed an update to their Cloudwatch `PutLogEvents` API and the matching `boto3` method on 2023-01-04 that removed the requirement for the sequence token, which made the whole thing a lot easier. I'm still publishing this document as a record of the learnings even if some of them are now out-of-date.

### Problem 1: Cloudwatch log groups and log streams

AWS Cloudwatch logging relies on the concept of log *streams*, which contain streams of individual log messages, which are all part of a single log *group*. Log streams are the things that actually get written to by the Lambda function, and the log group allows filtering and searching to be run across multiple log streams at once.

Most of the example code for pushing logs to Cloudwatch only pushed the logs to a single static log stream. This unfortunately does not work in practice, because of...

### Problem 2: Parallel Lambda function execution

If you push five separate unbatched events up to a Lambda function, AWS will not invoke one instance of the function that processes those five events one after another. Instead it will spawn five function instances that each process one event in parallel. 

This means that we potentially have five function instances all simultaneously attempting to write to the same Cloudwatch log stream. This was the cause of most of the initial errors to do with our implementation, thanks to...

### Problem 3: Cloudwatch log stream sequence tokens

When you do a `put_log_event` operation against a Cloudwatch log stream, one of the things you were supposed to pass was a sequence token. There is only one valid sequence token at any one time per log stream. Attempting to write to a log stream with an incorrect or out-of-date sequence token results in `InvalidSequenceTokenException`. And every *successful* write to a log stream updates the sequence token to a new value.

Therefore if we have five Lambda functions all trying to write to the same log stream with the same sequence token, the one that writes first will win and the other four will fail with `InvalidSequenceTokenException`. The exception does come with the updated token in the response (under the `expectedSequenceToken` property), so we can attempt to retry if we handle the exception; however the best case scenario is that one of the remaining four Lambda functions successfully writes and invalidates the new token and the other three fail again. The *worst* case scenario we saw was a function still failing out after 50 retries thanks to the sequence token being repeatedly invalidated.

### Solution part 1: Don't bother passing the sequence token

At the time that this code was being written the [boto3 documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/logs.html#CloudWatchLogs.Client.put_log_events) claimed we need to pass sequence token to `put_log_event` in order for our log event to be accepted. This is potentially awkward as there are only three ways of getting the sequence token:

- Via a successful `put_log_events` call, which returns the new sequence token on the response under `nextSequenceToken`. Of course, this is somewhat chicken-and-egg: how do you make a successful call if you don't have the right sequence token in the first place?
- Via a `describe_log_streams` call. This does not work as there's a limit of five `describe_log_streams` calls per second per log group. If you have many Lambda functions all trying to write to log streams inside the same log group at the same time some of them will fail out with `ThrottlingException`.
- By making an intentionally unsuccessful `put_log_events` call and then getting the sequence token from the `expectedSequenceToken` property on the `InvalidSequenceTokenException` response.

However the actual underlying `PutLogEvents` API has since been updated to no longer require the sequence token (see [PutLogEvents documentation](https://docs.aws.amazon.com/AmazonCloudWatchLogs/latest/APIReference/API_PutLogEvents.html)). `boto3` should match this behaviour as of version `1.26.43`.

For versions of `boto3` older than `1.26.43` (we were on `1.26.16`) this approach still works, and is the one we arrived at via trial and error. If you attempt to pass any sequence token as part of the `put_log_events` call that sequence token must still be correct, otherwise you still get `InvalidSequenceTokenException`. This will happen even if you try and pass `None` - whatever you pass as the sequence token must be the right type and value.

What we **are** allowed to do here, though, is **not pass a sequence token argument at all**. The `put_log_events` payload will be accepted without one.

### Solution part 2: Avoiding transaction limits

We're creating one log stream per log group (or app) per day. It would also be nice to dynamically create containing log groups as new applications are added. Both approaches are potentially risky, though, as both `create_log_stream` and `create_log_group` have transaction limits:

- `create_log_stream` is limited to 50 requests per second.
- `create_log_group` is limited to 5 requests per second.

We clearly don't want to be doing these operations unless we *need* to, but we also can't check to see if the log stream/group exists via a `describe_log_groups` or `describe_log_streams` call because these also have limits of 5 requests per second, which will be frequently breached by many Lambda function instances running in parallel. 

The somewhat-imperfect solution we arrived at was:

- Assume that both the log group and log stream already exist and attempt the `put_log_events` call first.
- If the `put` fails due to the log group or log stream not existing, it will return a `ResourceNotFoundException`.
- We check the error message returned by the `ResourceNotFoundException` to figure out whether it's the group or the stream that's missing.
- If the log stream is missing, we do `create_log_stream`. If the log group is missing, we do `create_log_group` and then `create_log_stream` (as the stream must also need to be created inside the new log group). 
  - Both `create` calls have an exception handler for `ResourceAlreadyExistsException`, as two simultaneously-executing Lambda function instances may attempt to create the same resource at the same time. If the resource has already been created, we pass.
- Then we retry `put_log_events`.
