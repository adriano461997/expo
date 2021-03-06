---
title: Advanced Release Channels
---

## Introduction

For a quick introduction to release channels, read [this](../release-channels/).

When you publish your app by running `expo publish --release-channel staging`, it creates:

- a release, identified by a `publicationId` for Android and iOS platforms. A release refers to your bundled source code and assets at the time of publication.
- a link to the release in the `staging` channel, identified by a `channelId`. This is like a commit on a git branch.

For simplicity, the rest of this article will refer to just the `ios` releases, but you could swap out ios for android at any point and everything would still be true.

## See past publishes
You can see everything that you’ve published with `expo publish:history`.

#### Example command and output
`expo publish:history --platform ios`

| publishedTime  | appVersion  | sdkVersion  | platform  | channel  | publicationId  |
|---|---|---|---|---|---|---|
| 2018-01-05T23:55:04.603Z  |  1.0.0 | 24.0.0 |  ios | staging  | 80b1ffd7-4e05-4851-95f9-697e122033c3 |

To see more details about this particular release, you can run `expo publish:details`

#### Example command and output
`expo publish:details --publish-id 80b1ffd7-4e05-4851-95f9-697e122033c3  `

![Publish Details](/static/images/release-channels-pub-details-1.png)


## What version of the app will my users get?

Your users will get the most recent compatible release that was pushed to a release channel. Factors that affect compatibility:

- sdkVersion (standalone apps are built to support only a single SDK version)
- platform
- releaseChannel

The following flowchart shows how we determine which release to return to a user:

![Serving Flowchart](/static/images/release-channels-flowchart.png)

## Promoting a release to a new channel

Example use case: you previously published a release to `staging` and everything went well in your testing. Now you want this release to be active in another channel (ie) production

We run `expo publish:set` to push our release to the `production` channel.
`expo publish:set --publish-id 80b1ffd7-4e05-4851-95f9-697e122033c3 --release-channel production`

Continuing from the previous section, we can see that our release is available in both the `staging` and the `production` channels.

`expo publish:history --platform ios`

| publishedTime  | appVersion  | sdkVersion  | platform  | channel  | publicationId  |
|---|---|---|---|---|---|---|
| 2018-01-05T23:55:04.603Z  |  1.0.0 | 36.0.0 |  ios | staging | 80b1ffd7-4e05-4851-95f9-697e122033c3  |
| 2018-01-05T23:55:04.603Z  |  1.0.0 | 36.0.0 |  ios | production | 80b1ffd7-4e05-4851-95f9-697e122033c3  |
| 2018-01-04T22:43:19.302Z  |  1.0.0 | 36.0.0 |  ios | production | d6b61741-a8dc-11e9-852a-3b0715b88238 |

## Rollback a channel entry

Example use case: you published a release to your `production` channel, only to realize that it includes a major regression for some of your users, so you want to revert back to the previous version.

Continuing from the previous section, we rollback our `production` channel entry for all platforms with `expo publish:set`

`expo publish:set --release-channel production --publish-id d6b61741-a8dc-11e9-852a-3b0715b88238`
`expo publish:set --release-channel production --publish-id 43d89652-a8dc-11e9-b565-4586c71668b9`

We could accomplish the same thing more succinctly with publish:rollback
`expo publish:rollback --release-channel production --sdk-version 36.0.0`

Now we can see that our releases are available on the production channel.

`expo publish:history --platform ios`

| publishedTime  | appVersion  | sdkVersion  | platform  | channel  | publicationId  |
|---|---|---|---|---|---|---|
| 2018-01-04T22:43:19.302Z  |  1.0.0 | 36.0.0 |  ios | production | d6b61741-a8dc-11e9-852a-3b0715b88238 |
| 2018-01-04T22:43:46.714Z  |  1.0.0 | 36.0.0 |  android | production | 43d89652-a8dc-11e9-b565-4586c71668b9 |


## Release channels CLI tools
### Publish history

```
  Usage: expo publish:history [--release-channel <channel-name>] [--count <number-of-logs>]

  View a log of your published releases.

  Options:
    -c, --release-channel <channel-name>  Filter by release channel. If this flag is not included, the most recent publications will be shown.
    -count, --count <number-of-logs>      Number of logs to view, maximum 100, default 5.
    -r, --raw                             Produce some raw output.
    -p, --platform <ios|android>          Filter by platform, android or ios.
```

### Publish details
```
  Usage: expo publish:details --publish-id <publish-id>
  View the details of a published release.

  Options:
    --publish-id <publish-id>  Publication id. (Required)
    -r, --raw                             Produce some raw output.
```

### Publish rollback
```
Usage: expo publish:rollback

  Rollback an update to a channel. Equivalent to running `expo publish:set` with publish-id set to the most recent release to the specified sdk version

  Options:
    -c, --release-channel <channel-name>  The name of the release channel to roll back (Required)
    -s, --sdk-version <version>           The SDK version to roll back (e.g. 37.0.0) (Required)
    -p, --platform <ios|android>          The platform to roll back (roll back both unless specified)
```

### Publish set
```
 Usage: expo publish:set --release-channel <channel-name> --publish-id <publish-id>

  Set a published release to be served from a specified channel.

  Options:
    -c, --release-channel <channel-name>  The channel to set the published release. (Required)
    -p, --publish-id <publish-id>         The id of the published release to serve from the channel. (Required)
```
