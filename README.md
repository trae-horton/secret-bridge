secret-bridge
==============

A bridge to help increase your ability to detect secrets shared on Github.

More details on the methodology can be found in our [blog post](https://duo.com/labs/research/how-to-monitor-github-for-secrets).

How It Works
============

There are two ways this can operate:

-	**Event Polling** - In this mode, the script polls the [Github Events API](https://developer.github.com/v3/activity/events/) for an organization, developer, or set or repositories. This is useful when you may not have control over the thing you're watching.
-	**Webhook** - In this mode, the script sets up a server which can receive webhook events. This is useful when you have control over the thing you're watching, since it provides data in near real-time.

Once events are received indicating that new code has been pushed, this script executes configured *detectors* to detect secrets in the changes. At this time, the following detectors are supported:

-	[`detect-secrets`](https://github.com/Yelp/detect-secrets)
-	[`git-secrets`](https://github.com/awslabs/git-secrets)

A more general listing of tools which can be used to detect secrets in Git repositories can be found in [TOOLS.md](TOOLS.md)

If a secret is found, it is sent upstream to a *notifier*. At this time, we support notifying both via stdout as well as Slack.

Installation
============

Via Docker
----------

The easiest way to get started is by using our Docker image. You can see how to run the Docker image in the Usage section below.

Installation from Source
------------------------

First, you need to clone the repository:

```
git clone https://github.com/duo-labs/secret-bridge.git
```

Then, install the required dependencies:

```
pip install -r requirements.txt
```

Configuration
=============

Configuration is done through `config.toml`. In this file, you set your `access_token`, the organizations, developers, and repositories you want to monitor for secrets, and more.

Setting Up the Access Token
---------------------------

You may wish to avoid having the access token in a file. Instead, you can set this value to `env`, and put the access token in the `GITHUB_WATCHER_TOKEN` environment variable.

Setting Up the Monitors
-----------------------

If you're monitoring via event polling (as opposed to using the webhook server), then you can configure what to monitor via the `monitors` configuration value.

You have the option of configuring one or more Github organization, user, or repository.

Setting Up the Detectors
------------------------

This tool doesn't actually implement secret detection for Git repositories, since we consider that largely a solved problem. Instead, we handle running various secret detection tools for you in near real-time.

Detectors are configured via the `detectors` configuration value. Right now, the following values are accepted:

-	`detect-secrets`
-	`git-secrets`

> Note: It's expected that the detector you use is installed and available on your `$PATH`. If you are running this via the Docker image, all the required tools are pre-installed.

Setting Up the Notifiers
------------------------

If a secret is detected, we will notify you using your configured `notifiers`. Currently, the two notification methods are printing to the console, and notifying via Slack.

To configure Slack notifications, create the following configuration option with the `webhook_url` provided by Slack:

```toml
[notifiers.slack_webhook]
    webhook_url='your_webhook_url'
```

Usage
=====

```
python main.py --help
Usage: main.py [OPTIONS] COMMAND [ARGS]...

Options:
  -f, --config PATH  [default: config.toml]
  -v, --verbose      Print verbose debug information
  --help             Show this message and exit.

Commands:
  poll
  webhook
```

Running Via Docker
------------------

```
docker run -ti --rm -e GITHUB_WATCHER_TOKEN=your_access_token duolabs/secret-bridge poll
sudo docker run -ti -v $(pwd)/config.toml:/usr/src/app/config.toml duolabs/secret-bridge poll
```
