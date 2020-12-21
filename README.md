# Slack for PHP

[![Total Downloads](https://poser.pugx.org/jeremykenedy/slack/d/total.svg)](https://packagist.org/packages/jeremykenedy/slack)
[![Latest Stable Version](https://poser.pugx.org/jeremykenedy/slack/v/stable.svg)](https://packagist.org/packages/jeremykenedy/slack)
[![StyleCI](https://github.styleci.io/repos/97971152/shield?branch=master)](https://github.styleci.io/repos/97971152?branch=master)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A simple PHP package for sending messages to [Slack](https://slack.com) with
[incoming webhooks](https://my.slack.com/services/new/incoming-webhook).

* Laravel integration: [Slack for Laravel](https://github.com/jeremykenedy/slack-laravel)

- [Requirements](#requirements)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
	- [Instantiate the client](#instantiate-the-client)
	- [Settings](#settings)
	- [Sending Messages](#sending-messages)
		- [Sending a basic message](#sending-a-basic-message)
		- [Sending a message to a non-default channel](#sending-a-message-to-a-non-default-channel)
		- [Sending a message to a user](#sending-a-message-to-a-user)
		- [Sending a message to a channel as a different bot name](#sending-a-message-to-a-channel-as-a-different-bot-name)
		- [Sending a message with a different icon](#sending-a-message-with-a-different-icon)
		- [Send an attachment](#send-an-attachment)
		- [Send an attachment with fields](#send-an-attachment-with-fields)
		- [Send an attachment with an author](#send-an-attachment-with-an-author)
- [Advanced Usage](#advanced-usage)
	- [Markdown](#markdown)
	- [Send a message enabling or disabling Markdown](#send-a-message-enabling-or-disabling-Markdown)
	- [Send an attachment specifying which fields should have Markdown enabled](#send-an-attachment-specifying-which-fields-should-have-Markdown-enabled)
	- [Explicit message creation](#explicit-message-creation)
	- [Attachments](#attachments)
- [Contributing](#contributing)
- [Credits](#credits)
- [License](#license)

### Requirements

* PHP 7.3, 7.4+

### Installation

##### 1. From your projects root folder in terminal run:

```bash
    composer require jeremykenedy/slack
```

##### 2. Create Webhook
	* [Create an incoming webhook](https://my.slack.com/services/new/incoming-webhook) on your Slack account for the package to use. You'll need the webhook URL to instantiate the client (or for the configuration file if using Laravel).

### Basic Usage

#### Instantiate the client

```php
// Instantiate without defaults
$client = new jeremykenedy\Slack\Client('https://hooks.slack.com/...');

// Instantiate with defaults, so all messages created
// will be sent from 'Cyril' and to the #accounting channel
// by default. Any names like @jeremykenedy or #channel will also be linked.
$settings = [
	'username' => 'Cyril',
	'channel' => '#accounting',
	'link_names' => true
];

$client = new jeremykenedy\Slack\Client('https://hooks.slack.com/...', $settings);
```

##### Settings

The default settings are pretty good, but you may wish to set up default behaviour for your client to be used for all messages sent. **All settings are optional and you don't need to provide any**. Where not provided, we'll fallback to what is configured on the webhook integration, which are [managed at Slack](https://my.slack.com/apps/manage/custom-integrations), or our sensible defaults.

Field | Type | Description
----- | ---- | -----------
`channel` | string | The default channel that messages will be sent to
`username` | string | The default username for your bot
`icon` | string | The default icon that messages will be sent with, either `:emoji:` or a URL to an image
`link_names` | bool | Whether names like `@jeremykenedy` or `#accounting` should be linked in the message (defaults to false)
`unfurl_links` | bool | Whether Slack should unfurl text-based URLs (defaults to false)
`unfurl_media` | bool | Whether Slack should unfurl media-based URLs, like tweets or Youtube videos (defaults to true)
`allow_markdown` | bool | Whether markdown should be parsed in messages, or left as plain text (defaults to true)
`markdown_in_attachments` | array | Which attachment fields should have markdown parsed (defaults to none)

#### Sending Messages

##### Sending a basic message
```php
$client->send('Hello world!');
```

##### Sending a message to a non-default channel
```php
$client->to('#accounting')->send('Are we rich yet?');
```

##### Sending a message to a user
```php
$client->to('@jeremykenedy')->send('Yo!');
```

##### Sending a message to a channel as a different bot name
```php
$client->from('Jake the Dog')->to('@FinnTheHuman')->send('Adventure time!');
```

##### Sending a message with a different icon
```php
// Either with a Slack emoji
$client->to('@jeremykenedy')->withIcon(':ghost:')->send('Boo!');

// or a URL
$client->to('#accounting')->withIcon('http://example.com/accounting.png')->send('Some accounting notification');
```

#### Send an attachment

```php
$client->to('#operations')->attach([
	'fallback' => 'Server health: good',
	'text' => 'Server health: good',
	'color' => 'danger',
])->send('New alert from the monitoring system'); // no message, but can be provided if you'd like
```

#### Send an attachment with fields

```php
$client->to('#operations')->attach([
	'fallback' => 'Current server stats',
	'text' => 'Current server stats',
	'color' => 'danger',
	'fields' => [
		[
			'title' => 'CPU usage',
			'value' => '90%',
			'short' => true // whether the field is short enough to sit side-by-side other fields, defaults to false
		],
		[
			'title' => 'RAM usage',
			'value' => '2.5GB of 4GB',
			'short' => true
		]
	]
])->send('New alert from the monitoring system'); // no message, but can be provided if you'd like
```

#### Send an attachment with an author

```php
$client->to('@jeremykenedy')->attach([
	'fallback' => 'Keep up the great work! I really love how the app works.',
	'text' => 'Keep up the great work! I really love how the app works.',
	'author_name' => 'Jane Appleseed',
	'author_link' => 'https://yourapp.com/feedback/5874601',
	'author_icon' => 'https://static.pexels.com/photos/61120/pexels-photo-61120-large.jpeg'
])->send('New user feedback');
```

### Advanced usage

#### Markdown

By default, Markdown is enabled for message text, but disabled for attachment fields. This behaviour can be configured in settings, or on the fly:

##### Send a message enabling or disabling Markdown

```php
$client->to('#weird')->disableMarkdown()->send('Disable *markdown* just for this message');

$client->to('#general')->enableMarkdown()->send('Enable _markdown_ just for this message');
```

##### Send an attachment specifying which fields should have Markdown enabled

```php
$client->to('#operations')->attach([
	'fallback' => 'It is all broken, man',
	'text' => 'It is _all_ broken, man',
	'pretext' => 'From user: *JimBob*',
	'color' => 'danger',
	'mrkdwn_in' => ['pretext', 'text']
])->send('New alert from the monitoring system');
```

#### Explicit message creation

For convenience, message objects are created implicitly by calling message methods on the client. We can however do this explicitly to avoid hitting the magic method.

```php
// Implicitly
$client->to('@jeremykenedy')->send('I am sending this implicitly');

// Explicitly
$message = $client->createMessage();

$message->to('@jeremykenedy')->setText('I am sending this explicitly');

$message->send();
```

#### Attachments

When using attachments, the easiest way is to provide an array of data as shown in the examples, which is actually converted to an Attachment object under the hood. You can also attach an Attachment object to the message:

```php
$attachment = new Attachment([
	'fallback' => 'Some fallback text',
	'text' => 'The attachment text'
]);

// Explicitly create a message from the client
// rather than using the magic passthrough methods
$message = $client->createMessage();

$message->attach($attachment);

// Explicitly set the message text rather than
// implicitly through the send method
$message->setText('Hello world')->send();
```

Each attachment field is also an object, an AttachmentField. They can be used as well instead of their data in array form:

```php
$attachment = new Attachment([
	'fallback' => 'Some fallback text',
	'text' => 'The attachment text',
	'fields' => [
		new AttachmentField([
			'title' => 'A title',
			'value' => 'A value',
			'short' => true
		])
	]
]);
```

You can also set the attachments and fields directly if you have a whole lot of them:

```php
// implicitly create a message and set the attachments
$client->setAttachments($bigArrayOfAttachments);

// or explicitly
$client->createMessage()->setAttachments($bigArrayOfAttachments);
```

```php
$attachment = new Attachment([]);

$attachment->setFields($bigArrayOfFields);
```

### Contributing

If you're having problems, spot a bug, or have a feature suggestion, please log and issue on Github.
If you'd like to have a crack yourself, fork the package and make a pull request.
Please include tests for any added or changed functionality. If it's a bug, include a regression test.

### Credits
* Full development credit must go to [maknz](https://github.com/maknz/slack).
* This package was forked and improved. The original package states that it was no longer maintained.
* This package was forked and modified to be compliant with [MIT](https://opensource.org/licenses/MIT) licencing standards for production use.

### License
Slack for PHP is licensed under the MIT license for both personal and commercial products. Enjoy!
