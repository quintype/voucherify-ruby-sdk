## Voucherify Ruby SDK

[Voucherify](http://voucherify.io?utm_source=inbound&utm_medium=github&utm_campaign=voucherify-ruby-sdk) has a new platform that will help your team automate voucher campaigns. It does this by providing composable API and the marketer-friendly interface that increases teams' productivity:

- **roll-out thousands** of vouchers **in minutes** instead of weeks,
- **check status** or disable **every single** promo code in real time,
- **track redemption** history and build reports on the fly.

Here you can find a library that makes it easier to integrate Voucherify with your Ruby server.

Full documentation is located at [voucherify.readme.io](https://voucherify.readme.io).

### Usage

#### Installation

Add this line to your application's Gemfile:

```ruby
gem 'voucherify'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install voucherify

#### Authentication

[Log-in](http://app.voucherify.io/#/login) to Voucherify web interface and obtain your Application Keys from [Configuration](https://app.voucherify.io/#/app/configuration):

![](https://www.filepicker.io/api/file/WKYkl2bSAWKHccEN9tEG)

```ruby
require "voucherify"

voucherify = Voucherify.new({
  "applicationId" => "YOUR-APPLICATION-ID-OBTAINED-FROM-CONFIGURATION",
  "clientSecretKey" => "YOUR-CLIENT-SECRET-KEY-OBTAINED-FROM-CONFIGURATION"
})
```

#### Error handling

This voucherify gem uses [rest-client](https://github.com/rest-client/rest-client) under the hood for handling HTTP calls. In case of an HTTP 4xx or 5xx error, a RestClient exception is thrown. The server's specific response is available in `.response` of the exception object.

```ruby
begin
  voucherify.publish("test")
rescue => e
  JSON.parse(e.response)
end
```

```json
{
    "code": 400,
    "message": "Couldn't find any voucher suitable for publication."
}
```

Please refer to [rest-client documentation](https://github.com/rest-client/rest-client#exceptions-see-httpwwww3orgprotocolsrfc2616rfc2616-sec10html) for more detailed information.

#### Listing vouchers

```ruby
voucherify.list({ limit: 10, skip: 20, category: "API Test" })
```

Result:
```json
[{
     "code": "9mYBpIk",
     "campaign": null,
     "category": "API Test",
     "discount": {
       "type": "AMOUNT",
       "amount_off": 400
     },
     "start_date": "2016-03-01T12:00:00Z",
     "expiration_date": null,
     "redemption": {
       "quantity": 1,
       "redeemed_quantity": 0,
       "redemption_entries": []
     },
     "active": true,
     "additional_info": null,
     "metadata": null
   },
   {
       "code": "AzTsIH",
       "campaign": null,
       "category": "API Test",
       "discount": {
        "type": "AMOUNT",
        "amount_off": 400
       },
       "start_date": "2016-03-01T10:00:00Z",
       "expiration_date": null,
       "redemption": {
        "quantity": 1,
        "redeemed_quantity": 0,
        "redemption_entries": []
       },
       "active": true,
       "additional_info": null,
       "metadata": null
   },
   ...
]  
```

#### Getting voucher details

```ruby
voucherify.get("v1GiJYuuS")
```

Result:
```json
{
    "code": "v1GiJYuuS",
    "campaign": "vip",
    "discount": {
        "percent_off": 10.0,
        "type": "PERCENT"
    },
    "expiration_date": "2015-12-31T23:59:59Z",
    "redemption": {
        "quantity": 3,
        "redeemed_quantity": 1,
        "redemption_entries": [
            {
                "date": "2015-09-24T06:03:35Z",
                "tracking_id": "GENERATED-OR-PROVIDED-TRACKING-ID"
            }
        ]
    },
    "additional_info": ""
}
```

#### Getting voucher redemption

```ruby
voucherify.redemption("v1GiJYuuS")
```

Result:
```json
{
    "quantity": 3,
    "redeemed_quantity": 1,
    "redemption_entries": [
        {
            "date": "2015-09-24T06:03:35Z",
            "tracking_id": "GENERATED-OR-PROVIDED-TRACKING-ID"
        }
    ]
}
```

#### Publishing voucher

This method selects active, unpublished voucher from the specific campaign and returns it to client. 
In effect, this voucher is marked as published and it will not be announced once again to customer. 

Example:

```ruby
voucherify.publish("First Ride")
```

Positive result:

```json
{
   "code": "FR-zT-u9I7zG",
   "campaign": "First Ride",
   "category": null,
   "discount": {
      "type": "PERCENT",
      "amount_off": 50
   },
   "start_date": "2015-01-01T00:00:00Z",
   "expiration_date": "2016-12-31T23:59:59Z",
   "redemption": {
      "quantity": 1,
      "redeemed_quantity": 0,
      "redemption_entries": []
   },
   "active": true,
   "additional_info": null,
   "metadata": {
      "published": "2016-01-22T09:25:07Z"
   }
}
```

Possible error:

```json
{
  "code": 400,
  "message": "Couldn't find any voucher suitable for publication."
}
```

#### Redeeming voucher

##### 1. Just by code

```ruby
voucherify.redeem("v1GiJYuuS")
```

Result (voucher details after redemption):

```json
{
    "code": "v1GiJYuuS",
    "campaign": "vip",
    "discount": {
        "percent_off": 10.0,
        "type": "PERCENT"
    },
    "expiration_date": "2015-12-31T23:59:59Z",
    "redemption": {
        "quantity": 3,
        "redeemed_quantity": 2,
        "redemption_entries": [
            {
                "date": "2015-09-24T06:03:35Z",
                "tracking_id": "(tracking_id not set)"
            },
            {
                "date": "2015-09-25T10:34:57Z",
                "tracking_id": "(tracking_id not set)"
            },
        ]
    },
    "additional_info": ""
}
```

Error:
```json
{
  "code": 400,
  "message": "voucher expired or quantity exceeded",
  "details": "v1GiJYuuS"
}
```

##### 2. With tracking id

You can provide a tracking id (e.g. your customer's login or a generated id) to the voucher redemption request.

```ruby
voucherify.redeem("v1GiJYuuS", "alice.morgan")
```

Result:
```json
{
    "code": "v1GiJYuuS",
    "campaign": "vip",
    "discount": {
        "percent_off": 10.0,
        "type": "PERCENT"
    },
    "expiration_date": "2015-12-31T23:59:59Z",
    "redemption": {
        "quantity": 3,
        "redeemed_quantity": 3,
        "redemption_entries": [
            {
                "date": "2015-09-24T06:03:35Z",
                "tracking_id": "(tracking_id not set)"
            },
            {
                "date": "2015-09-25T10:34:57Z",
                "tracking_id": "(tracking_id not set)"
            },
            {
                "date": "2015-09-25T12:04:08Z",
                "tracking_id": "alice.morgan"
            },
        ]
    },
    "additional_info": ""
}
```

##### 3. With customer profile

You can record a detailed customer profile consisting of an `id` (obligatory), `name`, `email`, `description` and a `metadata` section that can include any data you wish.

```ruby
voucherify.redeem({
    "voucher" => "v1GiJYuuS",
    "customer" => {
        "id" => "alice.morgan",
        "name" => "Alice Morgan",
        "email" => "alice@morgan.com",
        "description" => "look ma no hands",
        "metadata" => {
            "locale" => "en-GB",
            "shoeSize" => 36.6
        }
    }
})
```

#### Listing redemptions

Use `voucherify.redemptions(filter)` to get a filtered list of redemptions.

Example - 1000 successful redemptions from April 2016:

```ruby
filter = {
    limit: 1000,
    page: 0,
    start_date: "2016-04-01T00:00:00",
    end_date: "2016-04-30T23:59:59",
    result: "Success"
}

voucherify.redemptions(filter)
```

#### Creating vouchers

Use `voucherify.create(code, options)` to create new vouchers.

```ruby
code = "EASTER-2016" # use given voucher code
code = nil # for an automatically generated string

# single-use voucher with 10% off discount that is valid throughout the whole 2016
opts = {
  category: "New Customers",
  discount: {
    percent_off: 10.0,
    type: "PERCENT"
  },
  start_date: "2016-01-01T00:00:00Z",
  expiration_date: "2016-12-31T23:59:59Z",
  redemption: {
    quantity: 1
  }
}

voucherify.create(code, opts)
```

Result (voucher details):
```json
{
    "code": "9Yi5g",
    "campaign": null,
    "category": "New Customers",
    "discount": {
        "type": "PERCENT",
        "percent_off": 10.0
    },
    "start_date": "2016-01-01T00:00:00Z",
    "expiration_date": "2016-12-31T23:59:59Z",
    "redemption": {
        "quantity": 1,
        "redeemed_quantity": 0,
        "redemption_entries": []
    },
    "active": true,
    "additional_info": null,
    "metadata": null
}
```

#### Disabling a voucher

```ruby
voucherify.disable("EASTER-2016")
```

The response has empty body.

#### Enabling a voucher

```ruby
voucherify.enable("EASTER-2016")
```

The response has empty body.

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/rspective/voucherify-ruby-sdk.

## Changelog
- **2016-04-13** - `0.1.2` - removed dependency to `pry`
- **2016-04-12** - `0.1.1` - minor gemspec changes
- **2016-04-12** - `0.1.0` - First version:
  - Authentication
  - Voucher information: *retrieve voucher*, *list vouchers*, *retrieve redemptions*, *list redemptions*
  - Voucher operations: *redeem voucher*, *publish voucher*, *create voucher*, *enable/disable voucher*


## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).
