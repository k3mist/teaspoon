Teabag
======

Teabag is a Javascript test runner built on top of Rails. It can run headless using PhantomJS or in the browser within
your Rails application. It's intention is to be the simplest, but most complete Javascript testing solution for Rails
and the asset pipeline. It ships with the ability to use Jasmine or Mocha and has custom reporters for both libraries.

We've only just released Teabag, but we expect to be working on it for a while longer to get a glossy shine to
everything.  Check it out, write a few specs, and let us know what you think.

Ok, another test runner, right? Really? Yeah, that's a tough one, but we're pretty confident Teabag is the nicest one
you'll use. If not, you can swing by our offices in Denver and we'll give you a beer (and not even a crappy one).


## Installation

Add it to your Gemfile. In almost all cases you'll want to restrict it to the `:asset`, or `:development, :test`
groups.

```ruby
group :assets do
  gem "teabag"
end
```

In addition, to get setup you can install the initializer and get the `spec/javascripts` path created for you with a
basic spec helper by running the install generator.

```
rails generate teabag:install
```


## Usage

Teabag uses the Rails asset pipeline to serve your files, so you're free to use things like CoffeeScript in addition to
Javascript. This simplifies the fixtures as well and lets you do some pretty awesome things like use builder, hamlc,
rabl, etc. for your fixtures.

If you want a more visual experience you can browse to the specs in the browser, or use the rake task to run them on the
command line.  By default Teabag uses PhantomJS to run your specs headless via the console.  This turned out really
well, so I encourage checking Teabag out just for that.

### Browser

```
http://localhost:3000/teabag
```

And to browse to a specific suite use:

```
http://localhost:3000/teabag/my_fantasic_suite
```

### Console

```
rake teabag
```

You can also specify the suite with the rake task by using:

```
rake teabag suite=my_fantasic_suite
```

When a failure is encounter a url will be generated for you so you can pop open a browser and load a focus run for the
specific failures.

**Note:** We like to include our javascript specs into the default rake task, and here's an example of using it between
rspec and cucumber.

```ruby
task :default => [:spec, :teabag, :cucumber]
```


## Writing Specs

Depending on what framework you use, this can be slightly different. This expects a certain level of understanding of
the test framework that you're using, and there's some great resources for reading more about them elsewhere --
[Jasmine](http://pivotal.github.com/jasmine/) and [Mocha](http://visionmedia.github.com/mocha/).

Since we have the asset pipeline at our fingertips we're free to use the `= require` directive throughout our specs and
spec helpers, which makes your job a whole lot easier.

Here's a basic spec written in Javascript using Jasmine.

```javascript
//= require jquery
describe("My great feature", function() {

  it("will change the world", function() {
    expect(true).toBe(true);
    expect(jQuery).toBeDefined();
  });

});
```

Here's the same test written in CoffeeScript using Mocha + [expect.js](https://github.com/LearnBoost/expect.js) (note:
we provide expect.js and other great support libraries like [jasmine-jquery](https://github.com/velesin/jasmine-jquery)
-- you can read more about them below).

```coffeescript
#= require jquery
describe "My great feature", ->

  it "will change the world", ->
    expect(true).to.be(true)
    expect(true).to.not.be(undefined)
```

### Pending Specs

We've normalized defining a spec as pending between the two libraries as a service to you. Since Jasmine lacks the
concept entirely we've added it in, and since Mocha has several ways to accomplish it so we thought it would be worth
going over what we consider the standard between the two to be.

To mark a spec as pending you can simply not provide a function as a second argument, or you can use `xit` and
`xdescribe`.  Mocha provides some additional things like `it.skip`, but to keep it consistent we've normalized on what
they both support.

```coffeescript
describe "My great feature", ->

  it "hasn't been tested yet"

  xit "has a test I can't figure out" ->
    expect("complexity").to.be("easily testable")

  xdescribe "A whole section that I've not gotten to", ->

    it "hasn't been tested yet", ->
      expect(true).to.be(false)
```

If you're using one library and you want to take advantage of the things that that library provides you're completely
free to do so, and this is provided as a suggestion more than anything. Both the reporters understand the standard above
and have specs for them.


## Suites

Teabag comes with the concept of top level suites. These suites are run in isolation from one another, and can have
entirely different configurations.

When Teabag is run via the rake task, it will stop at any point that a suite fails, which allows you to create a
hierarchy of suites -- crafting tiers of pass expectation.

You can define suites in the configuration. For brevity `config` is the argument passed to the `Teabag.setup` block.

When creating a suite definition you simply have to pass it a name, and a configuration block. The following example
defines a suite named "my_suite". You can focus to just this suite by browsing to `teabag/my_suite` or running the rake
task with `suite=my_suite`.

```ruby
config.suite :my_suite do |suite|
  suite.helper = "my_spec_helper.js"
end
```

There's always a "default" suite defined, and you can modify this suite if you don't specify a name, or use `:default`.
In this example we're just adjusting the default suite configuration.

```ruby
config.suite do |suite|
  suite.helper = "other_spec_helper.js"
end
```

It's worth noting that suites don't inherit from the default suite values, but instead always fall back to the defaults
outlined below.

### Suite Configuration Directives

- **`matcher`**<br/>
   A file matcher for your specs.<br/><br/>
   default: `"{app/assets,lib/assets/,spec/javascripts}/**/*_spec.{js,js.coffee,coffee}"`

- **`helper`**<br/>
   Spec helper file (you can require other support libraries from this.)<br/><br/>
   default: "spec_helper"

- **`javascripts`**<br/>
   Primary Javascript files. Use "teabag-jasmine" or "teabag-mocha".  For coffeescript you can use "teabag/jasmine" /
   "teabag/mocha".<br/><br/>
   default: `["teabag-jasmine"]`

- **`stylesheets`**<br/>
   The stylesheets to load in this suite.<br/><br/>
   default: `["teabag"]`

**Note:** It's strongly encouraged to only include the base files in the `javascripts` directive. You can require
whatever libraries you need in your spec helper, which makes it easier to maintain because you won't have to restart the
server.


## Configuration

The best way to read about the configuration options is to generate the initializer, but we've included some info here
because we're nice guys.

**Note:** `Teabag.setup` may not be available in all environments, so keep that in mind. The generator provides a check
wrapped around Teabag.setup.

- **`mount_at`**<br/>
  This determines where the Teabag spec path will be mounted. Changing this to `"/jasmine"` would allow you to browse to
  `http://localhost:3000/jasmine` to run your specs.<br/><br/>
  default: `"/teabag"`

- **`root`**<br/>
  The root path defaults to Rails.root if left nil, but if you're testing an engine using a dummy application it's
  useful to be able to set this to your engines root.. E.g. `Teabag::Engine.root`<br/><br/>
  default: `nil`

- **`asset_paths`**<br/>
  These paths are appended to the rails assets paths (relative to config.root), and by default is an array that you can
  replace or add to.<br/><br/>
  default `["spec/javascripts", "spec/javascripts/stylesheets"]`

- **`fixture_path`**<br/>
  Fixtures are different than the specs, in that Rails is rendering them directly through a controller. This means you
  can use haml, erb builder, rabl, etc. to render content in the views available in this path.<br/><br/>
  default: `"spec/javascripts/fixtures"`

- **`server_timeout`**<br/>
  Timeout for starting the server in seconds. If your server is slow to start you may have to bump the timeout, or you
  may want to lower this if you know it shouldn't take long to start.<br/><br/>
  default: `20`


## Jasmine

Jasmine is a pretty big aspect of the default setup. We've been using Jasmine for a long time, and have been pretty
happy with it.  It lacks a few things that I think are important in a test framework, but we've done a little bit of
work to make that nicer.  Like pending spec support for example.


## Mocha

Mocha came up while we were working on Teabag -- I finally got around to reading up about it, and it's a pretty awesome
library with some really great approaches to some of the things that some of us browser types should consider more
often, so we included it and added support for it. We encourage you to give it a try.

Read [more about Mocha](http://visionmedia.github.com/mocha/).

To use Mocha just change your suite to include the `teabag-mocha` javascript.

```ruby
config.suite do |suite|
  suite.javascripts = ["teabag-mocha"]
end
```


## CI Support

More on this shortly....


## Support Libraries

Because we know that testing usually requires more than just the test framework we've included some of the great
libraries that we use on a consistent basis.

- [expect.js](https://github.com/LearnBoost/expect.js) Minimalistic BDD assertion toolkit (Mocha).
- [jasmine-jquery.js](https://github.com/velesin/jasmine-jquery) Great jQuery matchers and fixture support (Jasmine).
- [Sinon.JS](https://github.com/cjohansen/Sinon.JS) Great for stubbing Ajax (Mocha/Jasmine).


## License

Licensed under the [MIT License](http://creativecommons.org/licenses/MIT/)
