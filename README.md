# ValidEmail2
[![Build Status](https://travis-ci.org/micke/valid_email2.svg?branch=master)](https://travis-ci.org/micke/valid_email2)
[![Gem Version](https://badge.fury.io/rb/valid_email2.png)](http://badge.fury.io/rb/valid_email2)

Validate emails with the help of the `mail` gem instead of some clunky regexp.
Aditionally validate that the domain has a MX record.
Optionally validate against a static [list of disposable email services](config/disposable_email_domains.yml).
Optionally validate that the email is not subaddressed ([RFC5233](https://tools.ietf.org/html/rfc5233)).

### Why?

There are lots of other gems and libraries that validates email addresses but most of them use some clunky regexp.
I also saw a need to be able to validate that the email address is not coming from a "disposable email" provider.

### Is it production ready?

Yes, it is used in several production apps.

## Installation

Add this line to your application's Gemfile:

```ruby
gem "valid_email2"
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install valid_email2

## Usage

### Use with ActiveModel

If you just want to validate that it is a valid email address:
```ruby
class User < ActiveRecord::Base
  validates :email, presence: true, 'valid_email_2/email': true
end
```

To validate that the domain has a MX record:
```ruby
validates :email, 'valid_email_2/email': { mx: true }
```

To validate that the domain is not a disposable email:
```ruby
validates :email, 'valid_email_2/email': { disposable: true }
```

To validate that the domain is not a disposable email or a disposable email but whitelisted (under config/whitelisted_email_domains.yml):
```ruby
validates :email, 'valid_email_2/email': { disposable_with_whitelist: true }
```

To validate that the domain is not blacklisted (under config/blacklisted_email_domains.yml):
```ruby
validates :email, 'valid_email_2/email': { blacklist: true }
```

To validate that email is not subaddressed:
```ruby
validates :email, 'valid_email_2/email': { disallow_subaddressing: true }
```

To validate create your own custom message:
```ruby
validates :email, 'valid_email_2/email': { message: "is not a valid email" }
```

All together:
```ruby
validates :email, 'valid_email_2/email': { mx: true, disposable: true, disallow_subaddressing: true}
```

> Note that this gem will let an empty email pass through so you will need to
> add `presence: true` if you require an email

### Use without ActiveModel

```ruby
address = ValidEmail2::Address.new("lisinge@gmail.com")
address.valid? => true
address.disposable? => false
address.valid_mx? => true
address.subaddressed? => false
```

### Test environment

If you are validating `mx` then your specs will fail without an internet connection.
It is a good idea to stub out that validation in your test environment.  
Do so by adding this in your `spec_helper`:
```ruby
config.before(:each) do
  allow_any_instance_of(ValidEmail2::Address).to receive(:valid_mx?).and_return(true)
end
```

Validating `disposable` e-mails triggers a `mx` validation alongside checking if
the domain is disposable. The above stub does not apply to the `disposable`
validation and should therefore be individually stubbed with:
```ruby
config.before(:each) do
  allow_any_instance_of(ValidEmail2::Address).to receive(:mx_server_is_in?).and_return(false)
end
```

## Requirements

This gem requires Rails 3.2 or 4.0 or higher. It is tested against both versions using:
* Ruby-2.3
* Ruby-2.4
* Ruby-2.5
* Ruby-2.6

## Upgrading to v3.0.0

In version v3.0.0 I decided to move __and__ rename the config files from the
vendor directory to the config directory. That means:

`vendor/blacklist.yml` -> `config/blacklisted_email_domains.yml`  
`vendor/whitelist.yml` -> `config/whitelisted_email_domains.yml`

The `disposable` validation has been improved with a `mx` check. Apply the
stub, as noted in the Test environment section, if your tests have slowed
down or if they do not work without an internet connection.

## Upgrading to v2.0.0

In version 1.0 of valid_email2 we only defined the `email` validator.  
But since other gems also define a `email` validator this can cause some unintended
behaviours and emails that shouldn't be valid are regarded valid because the
wrong validator is used by rails.

So in version 2.0 we decided to deprecate using the `email` validator directly
and instead define a `valid_email_2/email` validator to be sure that the correct
validator gets used.

So this:
```ruby
validates :email, email: { mx: true, disposable: true }
```

Becomes this:
```ruby
validates :email, 'valid_email_2/email': { mx: true, disposable: true }
```

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
