# on_failure cookbook

[![Cookbook Version](http://img.shields.io/github/release/arangamani-cookbooks/on_failure.svg)][release]
[![Build Status](http://img.shields.io/travis/arangamani-cookbooks/on_failure.svg)][travis]

[release]: http://github.com/arangamani-cookbooks/on_failure/releases
[travis]: http://travis-ci.org/arangamani-cookbooks/on_failure

This cookbook is a proof of concept for the proposed [on_failure Chef RFC]. This cookbook doesn't implement all
possible features proposed but it covers most use cases for the feature. There are still [discussions] going
on in the proposal on how do certain things. This cookbook will help playing with the feature and proposing changes to
the RFC.

Follow this [blog post] for seeing this in action.

[on_failure Chef RFC]: https://github.com/opscode/chef-rfc/blob/sethvargo/on_failure/on_failure.md
[discussions]: https://github.com/opscode/chef-rfc/pull/1
[blog post]: http://blog.arangamani.net/blog/2014/04/08/chef-on-failure-handler/

# Requirements

* Requires Chef 11.
* Willingness to take risk (as it is still a brand new concept).

# Usage

Add the following `depends` statement to your `metadata.rb`.

```ruby
depends 'on_failure'
```

Once this cookbook is loaded, you will have access to the `on_failure` construct in any resource.

### Simple Example

```ruby
meal 'breakfast' do
  on_failure { notify :eat, 'food[bacon]' }
end
```

If the  `meal[breakfast]` resource's `:eat` action fails, it will notify the `:eat` action of `'food[bacon]'` resource
and then retry the `:eat` action again.

### Handling specific exceptions

A specific exception can be specified in the `on_failure` construct which will catch only the matching exceptions.
For example:

```ruby
meal 'breakfast' do
  on_failure(HungryError) { notify :eat, 'food[bacon]' }
end
```

In this example, we only want to catch the `HungryError` exception. All other exceptions will immediately raise.

### Specifying retries

By default the action will be retried only once. But you can specify the number of times the action should be retried.

```ruby
meal 'breakfast' do
  on_failure(retries: 5) { notify :eat, 'food[bacon]' }
end
```

In this example, the action will be retried 5 times before the exception is raised. If the action succeeds before all
retries, we are good and nothing wil be raised.

### Specifying multiple exceptions

Multiple exceptions classes can be specified to be caught. You can also include the options such as retries along with
specifying exceptions.

```ruby
meal 'breakfast' do
  on_failure(UncookedError, HungryError, retries: 3) { notify :fry, 'food[bacon]' }
end
```

### Specifying multiple blocks

If each exception should be handled differently, you can specify them in separate `on_failure` blocks. They will be
executed in the order they are defined (top-bottom).

```ruby
meal 'breakfast' do
  on_failure(UncookedError) { notify :fry, 'food[bacon]' }
  on_failure(ColdError) { notify :microwave, 'food[bacon]' }
end
```

### Access to resource attributes

The attributes of the resource can be accessed inside the block of `on_failure`. The following example demonstrates
feature.

```ruby
meal 'breakfast' do
  time '2014-04-09 08:00:00 -0700'
  on_failure { |breakfast| Chef::Log.info "Tried eating breakfast at: #{breakfast.time}" }
end
```

This handler simply logs the time specified in the resource.

# Testing/Playing

There are three cookbooks available in the `test/cookbooks` directory that will help playing with this resource.

### Cookbooks

* `sample` - A sample cookbook that uses the `on_failure` feature to demonstrate how it works.
* `meal` - The cookbook that provides the `meal` resource.
* `food` - The cookbook that provides the `food` resource.

This cookbook has a `Vagrantfile` that can be used to provision a virtual machine and start playing with stuffs.

The sample cookbook has recipes for each use cases explained in the Usage section. Just pick any recipe, put it in
the runlist of the virtualmachine and provision it to see it in action.

# Attributes

There are no attributes in this cookbook.

# Recipes

There are no recipes in this cookbook.

# License & Author

Author:: Kannan Manickam (<me@arangamani.net>)

```
Copyright (C) 2014 Kannan Manickam

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
