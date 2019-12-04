# magnet/payments.stripe
[![Build Status](https://api.travis-ci.com/magnetcoop/payments.stripe.svg?branch=master)](https://travis-ci.com/magnetcoop/payments.stripe)
[![Clojars Project](https://img.shields.io/clojars/v/magnet/payments.stripe.svg)](https://clojars.org/magnet/payments.stripe)

A [Duct](https://github.com/duct-framework/duct) library that provides an [Integrant](https://github.com/weavejester/integrant) key for interacting with the Stripe API.

## Table of contents
* [Installation](#installation)
* [Usage](#usage)
  * [Configuration](#configuration)
  * [Obtaining a Stripe record](#obtaining-a-stripe-record)
  * [Available methods](#available-methods)

## Installation

[![Clojars Project](https://clojars.org/magnet/payments.stripe/latest-version.svg)](https://clojars.org/magnet/payments.stripe)

## Usage

### Configuration
To use this library add the following key to your configuration:

`:magnet.paymets/stripe`

This key expects a configuration map with one unique mandatory key, plus another three optional ones.
These are the mandatory keys:

* `:api-key` : [API key](https://stripe.com/docs/keys) to authenticate requests

These are the optional keys:
* `:timeout`: Timeout value (in milli-seconds) for an connection attempt with Stripe.
* `:max-retries`: If the connection attempt fails, how many retries we want to attempt before giving up.
* `:backoff-ms`: This is a vector in the form [initial-delay-ms max-delay-ms multiplier] to control the delay between each retry. The delay for nth retry will be (max (* initial-delay-ms n multiplier) max-delay-ms). If multiplier is not specified (or if it is nil), a multiplier of 2 is used. All times are in milli-seconds.

Key initialization returns a `Stripe` record that can be used to perform the Stripe operations described below.

#### Configuration example
Basic configuration:
```edn
  :magnet.payments/stripe
   {:api-key #duct/env ["STRIPE_API_KEY" Str :or "pk_test_TYooMQauvdEDq54NiTphI7jx"]}
```

Configuration with custom request retry policy:
```edn
  :magnet.payments/stripe
   {:api-key #duct/env ["STRIPE_API_KEY" Str :or "pk_test_TYooMQauvdEDq54NiTphI7jx"]
    :timeout 1000
    :max-retries 3
    :backoff-ms [10 500]}
```

### Obtaining a `Stripe` record

If you are using the library as part of a [Duct](https://github.com/duct-framework/duct)-based project, adding any of the previous configurations to your `config.edn` file will perform all the steps necessary to initialize the key and return a `Stripe` record for the associated configuration. In order to show a few interactive usages of the library, we will do all the steps manually in the REPL.

First we require the relevant namespaces:

```clj
user> (require '[integrant.core :as ig]
               '[magnet.payments.core :as core])
nil
user>
```

Next we create the configuration var holding the Stripe integration configuration details:

```clj
user> (def config {:api-key #duct/env ["STRIPE_API_KEY" Str :or "pk_test_TYooMQauvdEDq54NiTphI7jx"]})
#'user/config
user>
```

Now that we have all pieces in place, we can initialize the `:magnet.paymets/stripe` Integrant key to get a `Stripe` record. As we are doing all this from the REPL, we have to manually require `magnet.payments.stripe` namespace, where the `init-key` multimethod for that key is defined (this is not needed when Duct takes care of initializing the key as part of the application start up):

``` clj
user> (require '[magnet.payments.stripe :as stripe])
nil
user>
```

And we finally initialize the key with the configuration defined above, to get our `Stripe` record:

``` clj
user> (def stripe-record (->
                       config
                       (->> (ig/init-key :magnet.paymets/stripe))))
#'user/stripe-record
user> stripe-record
#magnet.payments.stripe.Stripe{:api-key #duct/env ["STRIPE_API_KEY" Str :or "pk_test_TYooMQauvdEDq54NiTphI7jx"]
                               :timeout 2000,
                               :max-retries 10,
                               :backoff-ms [500 1000 2.0]}
user>
```
Now that we have our `Stripe` record, we are ready to use the methods defined by the protocols defined in `magnet.payments.core` namespace.

### Available methods

This are the methods available to interact with the Stripe API. The mapping for the methods is one to one, so refer to the Stripe  official documentation for details.

  * [Balance](https://stripe.com/docs/api/balance)
    * [(get-balance stripe-record)](https://stripe.com/docs/api/balance/balance_retrieve)
    * [(get-balance-transaction stripe-record bt-id)](https://stripe.com/docs/api/balance_transactions/retrieve)
    * [(get-balance-transactions stripe-record opt-args)](https://stripe.com/docs/api/balance_transactions/list)
  * [Charge](https://stripe.com/docs/api/charges)
    * [(create-charge stripe-record charge)](https://stripe.com/docs/api/charges/create)
    * [(get-charge stripe-record charge-id)](https://stripe.com/docs/api/charges/retrieve)
    * [(get-all-charges stripe-record opt-args)](https://stripe.com/docs/api/charges/list)
    * [(update-charge stripe-record charge-id charge)](https://stripe.com/docs/api/charges/update)
  * [Customer](https://stripe.com/docs/api/customers)
    * [(create-customer stripe-record customer)](https://stripe.com/docs/api/customers/create)
    * [(get-customer stripe-record customer-id)](https://stripe.com/docs/api/customers/retrieve)
    * [(get-all-customers stripe-record opt-args)](https://stripe.com/docs/api/customers/list)
    * [(update-customer stripe-record customer-id customer)](https://stripe.com/docs/api/customers/update)
    * [(delete-customer stripe-record customer-id)](https://stripe.com/docs/api/customers/delete)
  * [Plan](https://stripe.com/docs/api/plans)
    * [(create-plan stripe-record plan)](https://stripe.com/docs/api/plans/create)
    * [(get-plan stripe-record plan-id)](https://stripe.com/docs/api/plans/retrieve)
    * [(get-all-plans stripe-record opt-args)](https://stripe.com/docs/api/plans/list)
    * [(update-plan stripe-record plan-id plan)](https://stripe.com/docs/api/plans/update)
    * [(delete-plan stripe-record plan-id)](https://stripe.com/docs/api/plans/delete)
  * [Subscription](https://stripe.com/docs/api/subscriptions)
    * [(create-subscription stripe-record subscription)](https://stripe.com/docs/api/subscriptions/create)
    * [(get-subscription stripe-record subscription-id)](https://stripe.com/docs/api/subscriptions/retrieve)
    * [(get-all-subscriptions stripe-record opt-args)](https://stripe.com/docs/api/subscriptions/list)
    * [(update-subscription stripe-record subscription-id subscription)](https://stripe.com/docs/api/subscriptions/update)
    * [(cancel-subscription stripe-record subscription-id)](https://stripe.com/docs/api/subscriptions/cancel)


All the responses will include a `:success?` key. When `:success?` is `false`, `:reason` and `error-details` keys will be also included. The possible reasons are: `:bad-request`, `not-found`, `access-denied` and `error`. The `error-details` will include a map with the error information provided by the Stripe API.

## License

Copyright (c) 2019 Magnet S Coop.

This Source Code Form is subject to the terms of the Mozilla Public License,
v. 2.0. If a copy of the MPL was not distributed with this file, You can obtain
one at https://mozilla.org/MPL/2.0/
