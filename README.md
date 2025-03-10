<p align="center">
<br/>
<br/>
<br/>
   <img src="media/ks.png" width="420"/>
<br/>
<br/>
</p>
<p align="center">
<b>:white_check_mark: Automate your key and secret validation workflows</b>
<br/>
<b>:cowboy_hat_face: Over 30 different providers</b>
<br/>
<b>:robot: Export to JSON, audit via CSV</b>
<br/>
<hr/>
</p>


<p align="center">
<img src="media/screen.png" width="1019"/>
</p>

# :key: Keyscope <img src="https://github.com/spectralops/keyscope/actions/workflows/build.yml/badge.svg"/>

Keyscope is a key and secret workflow (validation, invalidation, etc.) tool built in Rust, powered by [`service_policy_kit`](https://github.com/spectralops/service-policy-kit).

Current workflows supported:

* Validation

# 🦀 Why Rust?

* With Rust, _"If it compiles, it works."_ and also, it compiles to many platforms.
* Rust is _fast_, has no _VM_, or unnecessary cruft (typically 5-8mb binaries with LOTS of code and libraries).
* Multi purpose, safe, and generalistic - makes for healthy and expressive [mission critical code](https://www.youtube.com/watch?v=ylOpCXI2EMM). Adding code or abstraction doesn't increase bloat, doesn't hurt performance, doesn't increase chance for bugs in a radical way (less edge cases).
* Amazing package manager: `Cargo`. Productive installing and running of tasks and examples.
* Rust is getting headlines in the security community as the go-to language for security tools. Equally interesting is offensive security + Rust [here](https://github.com/trickster0/OffensiveRust) and [here](https://github.com/skerkour/black-hat-rust).

# :rocket: Quick Start

Grab a release from [releases](https://github.com/spectralops/keyscope/releases), or install via Homebrew:

```
brew tap spectralops/tap && brew install keyscope
```

## Using keyscope

You can try out validating a key for a provider, say, Github (assuming the key  is in the `GITHUB_TOKEN` environment variable):

```
$ keyscope validate github $GITHUB_TOKEN
```

You can see which other providers are supported by running:

```
$ keyscope validate --providers

  .
  :
  .

twilio:validation
keyscope validate twilio -p p1 p2

twitter:validation
keyscope validate twitter -p p1

zendesk:validation
keyscope validate zendesk -p p1 p1

Total 33 providers available.
$
```

And what parameters are required for a certain provider by running (say, `stripe`):

```
$ keyscope validate stripe --requirements

provider stripe requires:
 - param: p1
   desc: stripe key
$
```

Finally the general structure of the `validate` command is:

```
$ keyscope validate PROVIDER -p PARAM1 PARAM2 .. PARAM_N
```

# :white_check_mark: Validation: key should be active


You can validate a specific provider like so:

```
$ keyscope validate twilio -p $TWILIO_KEY
```

With the general pattern of:

```
$ keyscope validate PROVIDER -p PARAM1 PARAM2 ...
```

The number of keys/params would change based on authentication type:

* `Bearer` - usually just a single key (token)
* `Basic Auth` - usually 2 keys: user, password
* `OAuth` - usually 2 keys: client_id, client_secret
* And others.

Each provider in Keyscope will tell you what it requires using `requirements`:

```
$ keyscope validate twilio --requirements
```

You'll get a report:

```
$ keyscope --verbose validate stripe -p $STRIPE_KEY

✔ stripe:validation: ok 766ms

Ran 1 interactions with 1 checks in 766ms

Success: 1
Failure: 0
  Error: 0
Skipped: 0
```

And an executable exit code that reflects success/failure.

You can use the `--verbose` flag to see API responses:

```
$ keyscope --verbose validate stripe -p $STRIPE_KEY

✗ stripe:validation: failed 413ms
      status_code: 200 != 401 Unauthorized

Ran 1 interactions with 1 checks in 413ms

Success: 0
Failure: 1
  Error: 0
Skipped: 0
```

In this case the exit code is `1` (failure).

# :x: Validation: key should be inactive

When you are validating keys that are supposed to be inactive, you can use the `flip` flag. In this mode, a failed API access is a good thing, and the exit code will reflect that.

```
$ keyscope --flip validate stripe -p $STRIPE_KEY

✔ stripe:validation: ok 766ms

Ran 1 interactions with 1 checks in 766ms
```

In this case, the key is active - _which is bad for us_. Using `--flip`, the exit code will be `1` (failure).

# :runner: Setting up a validation job

## Audit active keys

You can set up a CI job (or other form of scheduled job you like) to perform an audit, by reading all parameters from a dedicated CSV file like so:

```
$ keyscope validate --csv-in report.csv
```

The format of the CSV that you need to prepare should include a header line and look like this:

```
provider,key1,key2,key3
twilio,tw-key1,,,
```

You can specify as many key columns as you like, as long as you provide an _empty_ value for providers which don't have that many keys, and all rows contain the same amount of cells.




## Audit inactive keys

If you have a dump of keys from your vault that are stale have expiry and should have been rotated, you want to test that they are all stale:

```
$ keyscope --flip validate --csv-in my-key-audit.csv
```


# :link: Supported providers

We're always adding [new providers](src/defs.yaml), keep a link to this list or watch this repo to get updated.

We use our [`service_policy_kit`](https://github.com/spectralops/service-policy-kit) library to specify interactions with services and their policies, if you find a service [not in this list](src/defs.yaml) feel free to open an issue or contribute back.


<!-- providers -->
| provider | actions | params |
|---|---|---|
|**tester**<br/>Tester: valid key|validation|`tester_1` - hookbin ID (https://hookb.in)<br/>`tester_2` - fake key to put as a query param|
|**covalenthq**<br/>Covalent: valid key|validation|`covalenthq_1` - covalent token|
|**asana**<br/>Asana: valid token|validation|`asana_1` - asana token|
|**bitly**<br/>Bit.ly: valid access token|validation|`bitly_1` - bit.ly token|
|**localytics**<br/>Localytics: valid API credentials|validation|`localytics_1` - localytics user<br/>`localytics_2` - localytics key|
|**algolia**<br/>Algolia: valid API credentials|validation|`algolia_1` - algolia application ID<br/>`algolia_2` - algolia index<br/>`algolia_3` - algolia API key|
|**branchio**<br/>branch.io: valid API credentials|validation|`branchio_1` - branch.io key<br/>`branchio_2` - branch.io secret|
|**browserstack**<br/>browserstack: valid API credentials|validation|`browserstack_1` - browserstack key<br/>`browserstack_2` - browserstack secret|
|**buildkite**<br/>Buildkite: valid token|validation|`buildkite_1` - buildkite token|
|**datadog**<br/>datadog: valid API credentials|validation|`datadog_1` - datadog API key|
|**github**<br/>github: valid API credentials|validation|`github_1` - github token|
|**dropbox**<br/>dropbox: valid API credentials|validation|`dropbox_1` - dropbox token|
|**gitlab**<br/>gitlab: valid API credentials|validation|`gitlab_1` - gitlab token|
|**heroku**<br/>heroku: valid API credentials|validation|`heroku_1` - heroku token|
|**mailchimp**<br/>mailchimp: valid API credentials|validation|`mailchimp_1` - mailchimp datacenter ID<br/>`mailchimp_2` - mailchimp key|
|**mailgun**<br/>mailgun: valid API credentials|validation|`mailgun_1` - mailgun key|
|**pagerduty**<br/>pagerduty: valid API credentials|validation|`pagerduty_1` - pagerduty token|
|**circleci**<br/>circleci: valid API credentials|validation|`circleci_1` - circleci key|
|**facebook-access-token**<br/>facebook: valid API token|validation|`facebook-access-token_1` - facebook token|
|**salesforce**<br/>salesforce: valid API credentials|validation|`salesforce_1` - salesforce instance name<br/>`salesforce_2` - salesforce token|
|**jumpcloud**<br/>jumpcloud: valid API credentials|validation|`jumpcloud_1` - jumpcloud key|
|**saucelabs-us**<br/>saucelabs-us: valid API credentials|validation|`saucelabs-us_1` - saucelabs user<br/>`saucelabs-us_2` - saucelabs key|
|**saucelabs-eu**<br/>saucelabs-eu: valid API credentials|validation|`saucelabs-eu_1` - saucelabs user<br/>`saucelabs-eu_2` - saucelabs key|
|**sendgrid**<br/>sendgrid: valid API credentials|validation|`sendgrid_1` - sendgrid key|
|**slack**<br/>slack: valid API credentials|validation|`slack_1` - slack key|
|**slack-webhook**<br/>slack-webook: valid API credentials|validation|`slack-webhook_1` - slack webhook|
|**stripe**<br/>stripe: valid API credentials|validation|`stripe_1` - stripe key|
|**travisci**<br/>travisci: valid API credentials|validation|`travisci_1` - travisci domain, choose 'org' or 'com'<br/>`travisci_2` - travisci key|
|**twilio**<br/>twilio: valid API credentials|validation|`twilio_1` - twilio account sid<br/>`twilio_2` - twilio token|
|**twitter**<br/>twitter: valid API credentials|validation|`twitter_1` - twitter API token|
|**zendesk**<br/>zendesk: valid API credentials|validation|`zendesk_1` - zendesk domain<br/>`zendesk_2` - zendesk key|
|**firebase**<br/>firebase: valid API credentials|validation|`firebase_1` - firebase API key<br/>`firebase_2` - firebase ID token|
|**aws**<br/>aws: valid API credentials|validation|`aws_1` - AWS ID<br/>`aws_2` - AWS secret|
|**elastic-apm-secret**<br/>Elastic APM: secret key validation|validation|`elastic-apm-secret_1` - Elastic APM host address and port, including 'http/s' part<br/>`elastic-apm-secret_2` - Elastic APM secret|
|**artifactory**<br/>Artifactory: token validation|validation|`artifactory_1` - Artifactory host (including http(s) part)<br/>`artifactory_2` - Artifactory token|
|**ibm-cos**<br/>IBM: cloud object storage key validation (HMAC)|validation|`ibm-cos_1` - IBM HMAC ID<br/>`ibm-cos_2` - IBM HMAC secret|
<!-- /providers -->







# :cake: Adding your own providers

You can specify a custom definitions file (here [is an example](custom-defs.example.yaml)):

```
$ keyscope -f your-definitions.yaml validate --list
```

Which is suitable for adding your own internal services, key issuing policies, and infrastructure.

You can also use custom definitions to test out new providers that you'd like to contribute back to keyscope :smile:

## Basics of a definition

All definitions represent an interaction. A request being made, and a policy that's being checked against it.

```yaml
providers:
  hookbin:
    validation:
      #
      # the request part
      #
      request:
        params:
        - name: hookbin_1
          desc: hookbin ID (https://hookb.in)
        - name: hookbin_2
          desc: fake key to put as a query param
        id: "postbin:validation"
        desc: "Postbin: valid key"
        # variable interpolation is good for all fields
        uri: "https://hookb.in/{{hookbin_1}}?q={{hookbin_2}}"
        method: post
        # you can also use headers, body, form, basic_auth and more (see defs.yaml)

      # 
      # the policy part: all values are actually regular expressions and are matched against service response
      #
      response:
        status_code: "200"
        body: ok
```

When in doubt, you can check keyscope's own [defs.yaml](src/defs.yaml) for real examples of how to use this infrastructure.

## Tutorial: adding Dropbox

To validate if a dropbox API key works, we first need to learn about the canonical way to authenticate against that API.

First stop, API docs:

* Dropbox has an [API Explorer](https://dropbox.github.io/dropbox-api-v2-explorer) which is super useful

Next stop, we want to find an API call that is a representative for:

* Has to be authenticated
* Has to indicate that when accessed successfully with our candidate key, the key has some authoritative value. Which means, that if exposed, contains significant risk.

For this example, getting our current account sounds like something that only when we identify who we are - we're able to do.

We'll select [get_current_account](https://www.dropbox.com/developers/documentation/http/documentation#users-get_current_account).

Let's start forming our interaction. First the needed skeleton: containing the name of the provider (`dropbox`), its ID and description below, as well as parameters required and their name and description:

```yaml
  dropbox:
    validation:
      request:
        id: "dropbox:validation"
        desc: "dropbox: valid API credentials"
        params:
        - name: dropbox_1
          desc: dropbox token
```

We keep the name of the parameter with a special convention that helps when feeding keyscope automatically:

```
PROVIDER_N
Where 'N' starts in 1 e.g.:
dropbox_1
dropbox_2
aws_1
...
```

Then, details about actually making an HTTP call, as required by Dropbox (Bearer token authentication).

```yaml
        uri: https://api.dropboxapi.com/2/users/get_current_account
        method: post
        headers:
          Authorization:
          - Bearer {{dropbox_1}}
```

Note that per standard, all HTTP header fields are actually _arrays_. It's OK to always make an array of size _one_ if you only have one value (most common case).

We also see _variable interpolation_ here. Where `{{dropbox_1}}` will get replaced by keyscope in time before making the actual call.


Finally, we want to make sure we answer the question:

* What does it mean to have a successful call?

In our case, the Dropbox API call returns `HTTP OK` on success, which means a `200` status code.

And the final, complete result is this:

```yaml
dropbox:
  validation:
    request:
      id: "dropbox:validation"
      desc: "dropbox: valid API credentials"
      params:
      - name: dropbox_1
        desc: dropbox token
      uri: https://api.dropboxapi.com/2/users/get_current_account
      method: post
      headers:
        Authorization:
        - Bearer {{dropbox_1}}
    response:
      status_code: "200"
```

Meanwhile, you can drop this provider in your own `providers.yaml` file and run keyscope:

```
$ keyscope -f providers.yaml validate dropbox -p MY_KEY
```

Now you can keep this in your private `providers.yaml` file or contribute it back to keyscope if you think other people might enjoy using it - we're happy to accept pull requests.



# Thanks

To all [Contributors](https://github.com/spectralops/keyscope/graphs/contributors) - you make this happen, thanks!


# Copyright

Copyright (c) 2021 [@jondot](http://twitter.com/jondot). See [LICENSE](LICENSE.txt) for further details.
