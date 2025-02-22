# joelpet/mailbox-org-cli

**Unoffical** command line "client" for managing [mailbox.org](https://mailbox.org) [disposable addresses](https://mailbox.org/en/post/more-privacy-with-anonymous-disposable-e-mail-addresses).

TBH it's hard to call it client - it **does not** use API for achieve its goals, as there is no API for that feature.

What it does it pretends to be browser and interacts with management panel in good-old form-based way.

## Installation

If you have Go installed, you can either:

* install it from source `go install github.com/joelpet/mailbox-org-cli@latest`
* clone repository and build it by yourself: `go build .` (requires Go 1.17)

## Usage

```text
Command line "client" for mailbox.org disposable addresses feature
mailbox-org-cli 0.1.0
Usage: mailbox-org-cli --username USERNAME [--password PASSWORD] [--password-on-stdin] <command> [<args>]

Options:
  --username USERNAME    mailbox.org username [env: MAILBOX_ORG_USERNAME]
  --password PASSWORD    mailbox.org password [env: MAILBOX_ORG_PASSWORD]
  --password-on-stdin    read password from stdin
  --help, -h             display this help and exit
  --version              display version and exit

Commands:
  list                   list disposable addresses
  renew                  renew disposable address
  delete                 delete disposable address
  set-memo               set-memo on existing disposable address
  create                 create new disposable address with optional memo
```

Here is an example how you can use this command with password manager:

```text
$ pass Email/mailbox.org | mailbox-org-cli --username you@example.com --password-on-stdin list

[
  {
    "email": "kajsdlkj230@temp.mailbox.org",
    "memo": "foo bar",
    "expires": "2022-02-28"
  },
  {
    "email": "aks92jasl943@temp.mailbox.org",
    "memo": "", # there's no memo set
    "expires": "2022-03-31"
  }
]
```

All output is JSON, so you will probably need something like [`jq`](https://github.com/stedolan/jq) to extract specific data. Using example output above this command will copy first item's email into clipboard (`wl-copy` on Wayland):

```text
mailbox-org-cli ... list | jq --raw '.[0].email' | wl-copy
```

### Possible use cases

* mailbox.org's disposable addresses have expiry date. But can be extended as many times as required. So if you want to have "permanent" address, just set cron every, lets say, 2 weeks with `mailbox-org-cli renew`.
* as this is CLI tool, you can easily integrate it with some launcher like [Alfred](https://www.alfredapp.com/)
* ...your idea :)

### Design decisions

1. Why it's in Go instead of JS/Python/PHP/other-scripting-language ?

The original author based the first version on JS, but then they realized that they wanted single binary which can be run on `scratch`, without any JS, Python, PHP, Ruby, etc interpreter installed.

2. Why there are no tests?

Because of how `surf` works, it's hard to "feed" it with stubbed HTML content. Other solutions include using some [HTTP mocking library](https://github.com/h2non/gock) or setuping some [local mock server](https://mockoon.com/). Manual testing will suffice for now.

3. Why does it use `username`/`password` instead of token?

Becasue the original author didn't find a way to generate application token in Mailbox.org interface. Official [API](https://api.mailbox.org/v1/doc/welcome/Grundlegende-Informationen.html) also requires username/password and gives you token which is valid only for 20 minutes.

One could probably use `PHPSESSID` but extracting this requires you either to dig into browser's Dev Tools or CLI tool should store it somewhere after first login.

Due to the fact that this tool isn't making dozens of API requests in one call and how fast login process is, ATM username/password is the way to go.
