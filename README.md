# Lita

[![Gem Version](https://badge.fury.io/rb/lita.png)](http://badge.fury.io/rb/lita)
[![Build Status](https://travis-ci.org/jimmycuadra/lita.png?branch=master)](https://travis-ci.org/jimmycuadra/lita)
[![Code Climate](https://codeclimate.com/github/jimmycuadra/lita.png)](https://codeclimate.com/github/jimmycuadra/lita)
[![Coverage Status](https://coveralls.io/repos/jimmycuadra/lita/badge.png)](https://coveralls.io/r/jimmycuadra/lita)

![Lita](http://f.cl.ly/items/0c271a2P3k2V180B1R0X/lita.jpg)

**Lita** is a chat bot written in Ruby with persistent storage provided by [Redis](http://redis.io/). It can connect to any chat service (given that there is an [adapter](#adapters) available for it) and can have new behavior added via [handlers](#handlers). The plugin system is managed with regular RubyGems and [Bundler](http://gembundler.com/).

Automate your business and have fun with your very own robot companion.

## Features

* Can work with any chat service
* Simple installation and setup
* Easily extendable with plugins
* Data persistence with Redis
* Built-in web server and routing
* Support for outgoing HTTP requests
* Group-based authorization
* Configurable logging
* Generators for creating new plugins
* Built-in process daemonization

## Why?

Lita draws much inspiration from GitHub's fantastic [Hubot](http://hubot.github.com/), but has a few key differences and strengths:

* It's written in Ruby.
* It exposes the full power of Redis rather than using it to serialize JSON.
* It's easy to develop and test plugins for with the provied [RSpec](https://github.com/rspec/rspec) extras. Lita strongly encourages thorough testing of plugins.
* It uses the Ruby ecosystem's standard tools (RubyGems and Bundler) for plugin installation and loading.
* It's thoroughly documented.

## Is it any good?

Yes.

## Dependencies

* Ruby 2.0
* Redis

## Installation

First, install the gem with `gem install lita`. This gives you access to the `lita` command. Run `lita help` to list available tasks.

Generate a new Lita instance by running `lita new NAME`. This will create a new directory called NAME (defaults to "lita") with a Gemfile and Lita configuration file.

## Usage

To start your Lita instance, simply run `lita`. This will load up all the plugins (adapters and handlers) declared in your Gemfile, load any configuration you've defined (more on that later) and start the bot.

## Adapters

The core Lita gem by itself doesn't do much. To make real use of it, you'll want to install an adapter gem to allow Lita to connect to the chat service of your choice. Find the gem for the service you want to use on [the list of adapters](https://github.com/jimmycuadra/lita/wiki/Adapters), then add it to your Gemfile. For example:

``` ruby
gem "lita-hipchat"
```

Adapters will likely require some configuration to be able to connect. See the documentation for the adapter for details.

Without installing an adapter, you can use the default shell adapter to chat with Lita in your terminal. Lita doesn't respond to many messages by default, however, so you'll want to add some new behavior to Lita via handlers.

## Handlers

Handlers are gems that add new behavior to Lita. They are responsible for listening for incoming messages and responding to them appropriately. Find the handler gems you want for your bot on [the list of handlers](https://github.com/jimmycuadra/lita/wiki/Handlers), then add them to your Gemfile. For example:

``` ruby
gem "lita-karma"
```

## Configuration

To configure Lita, edit the file `lita_config.rb` generated by the `lita new` command. This is just a plain Ruby file that will be evaluated when the bot is starting up. A Lita config file looks something like this:

``` ruby
Lita.configure do |config|
  config.robot.name = "Sir Bottington"
  config.robot.mention_name = "bottington"
  config.robot.adapter = :example_chat_service
  config.adapter.username = "bottington"
  config.adapter.password = "secret"
  config.redis.host = "redis.example.com"
  config.handlers.karma.cooldown = 300
  config.handlers.google_images.safe_search = :off
end
```

The main config objects are:

* `robot` - General settings for Lita.
  * `name` (String) - The display name the bot will use on the chat service. Default: `"Lita"`.
  * `mention_name` (String) - The name the bot will look for in messages to determine if the message is being addressed to it. Usually this is the same as the display name, but in some cases it may not be. For example, in HipChat, display names are required to be a first and last name, such as "Lita Bot", whereas the mention system would use a name like "LitaBot". Default: `Lita.config.robot.name`.
  * `adapter` (Symbol, String) - The adapter to load. Default: `:shell`.
  * `log_level` (Symbol, String) - The severity level of log messages to output. Valid options are, in order of severity: `:debug`, `:info`, `:warn`, `:error`, and `:fatal`. For whichever level you choose, log messages of that severity and greater will be output. Default: `:info`.
  * `admins` (Array<String>) - An array of string user IDs which tell Lita which users are considered administrators. Only these users will have access to Lita's `auth` command. Default: `nil`.
* `redis` - Options for the Redis connection. See the [Redis gem](https://github.com/redis/redis-rb) documentation.
* `http` - Settings related to Lita's built-in web server.
  * `port` (Integer) - The port the server should run on. Default: `8080`.
  * `debug` (Boolean) - Set to true to display the web server's logs mixed in with Lita's own logs. Default: `false`.
* `adapter` - Options for the chosen adapter. See the adapter's documentation.
* `handlers` - Handlers may choose to expose a config object here with their own options. See the handler's documentation.

If you want to use a config file with a different name or location, invoke `lita` with the `-c` option and provide the path to the config file.

## Authorization

Access to commands can be allowed for only certain users by means of authorization groups. Users set as admins (by adding their user IDs to the `config.robot.admins` array in Lita's configuration) have access to two commands:

```
Lita: auth add joe committers
Lita: auth remove joe committers
```

The first command adds a user whose ID or name is "joe" to the authorization group "committers." If the group doesn't yet exist, it is created. The second command removes joe from the group. Handlers can specify that a route (a method that matches an incoming message) requires that the user sending the message be in a certain authorization group. See the section on writing handlers for more details.

To list all the authorization groups and the names of the users in them, send Lita this command:

```
Lita: auth list
```

You can optionally suffix the command with the name of a group if you're only interested in the memebers of one group.

## Online help

Message Lita `help` for a list of commands it knows about. You can also message it `help FOO` to list only commands beginning with FOO.

## Shell adapter

Lita ships with one adapter for use directly in the shell. Simply type text at the input to send messages, and Lita will respond with any registered handlers. The shell adapter has one configuration attribute:

* `private_chat` (Boolean) - If true, all messages will be treated as though they were sent in a private chat, so they will be considered commands even when not prefixed with the bot's name. Default: `false`.

## Writing an adapter

An adapter is a packaged as a RubyGem. The adapter is a class that inherits from `Lita::Adapter`, implements a few required methods, and is registered by calling `Lita.register_adapter(:symbol_that_identifies_the_adapter, TheAdapterClass)`.

To generate a starting template for a new adapter gem, run `lita adapter NAME`, where NAME is the name of the new gem.

### Example

Here is a bare bones example of an adapter for the fictious chat service, FancyChat.

``` ruby
module Lita
  module Adapters
    class FancyChat < Adapter
      # Optional. Makes the bot produce an error message and quit upon start up
      # if `config.adapter.username` or `config.adapter.password` are not set.
      require_configs :username, :password

      # Connects to the chat service and dispatches incoming messages to a
      # Lita::Robot instance.
      def run
      end

      # Sends a message from the robot to a user or room on the chat service.
      def send_messages(target, strings)
      end

      # Sets the topic for a chat room.
      def set_topic(target, topic)
      end

      # Does any clean up necessary when disconnecting from the chat service.
      def shut_down
      end
    end

    Lita.register_adapter(:fancy_chat, FancyChat)
  end
end
```

It's important to note that each adapter should employ its own thread or event mechanism so that incoming messages can still be processed even while a handler is processing a previous message.

For more detailed examples, check out the built in shell adapter, [lita-hipchat](https://github.com/jimmycuadra/lita-hipchat), or [lita-irc](https://github.com/jimmycuadra/lita-irc). See the API documentation for the exact methods and signatures adapters must implement.

## Writing a handler

A handler is packaged as a RubyGem. A handler is a class that inherits from `Lita::Handler` and is registered by calling `Lita.register_handler(TheHandlerClass)`. There are two components to a handler: route definitions, and the methods that implement those routes. There are both chat routes and HTTP routes available to handlers.

To generate a starting template for a new handler gem, run `lita handler NAME`, where NAME is the name of the new gem.

### Chat routes

To define a route, use the class method `route`:

``` ruby
route /^echo\s+(.+)/, :echo
```

`route` takes a regular expression that will be used to determine whether or not an incoming message should trigger the route, and the name of the method that should be called when this route is triggered. `route` takes a few additional options:

* `:command` (Boolean) - If set to true, the route will only trigger when "directed" at the robot. Directed means that it's sent via a private message, or the message is prefixed with the bot's name in some form (optionally prefixed with an @, and optionally followed by a colon or comma and white space). This prefix is stripped from the message body itself, but `Lita::Message#command?` available in handlers can be used if you need to determine whether or not a message was a command after it's been routed. Default: `false`.
* `:restrict_to` (Symbol, String, Array<String, Symbol>) - Authorization groups necessary to trigger the route. The user sending the message must be a member of at least one of the supplied groups. See the section on authorization for more information. Default: `nil`.
* `:help` (Hash<String>) - A map of example invocations of the route and descriptions of what they do. These values will be used to generate the listing for the built-in "help" handler. The robot's mention name will automatically be added to the front of the example if the route is a command. Default: `{}`.

Here is an example of a route declaration with all the options:

``` ruby
route /^echo\s+(.+)/, to: :echo, command: true, restrict_to: [:testers, :committers], help => {
  "echo FOO" => "Replies back with FOO."
}
```

Each method that is called by a route takes one argument, a `Lita::Response` object. This object has the following useful methods:

* `reply` - Sends one or more string messages back to the source of the original message, either a private message or a chat room.
* `reply_privately` - Sends one or more string messages back to the user who sent the original message, whether it initated in a private message or a chat room.
* `matches` - An array of regular expression matches obtained by calling `body_of_message.scan(route_regex)`.
* `match_data` - A `MatchData` object obtained by calling `route_regex.match(body_of_message)`.
* `args` - The user's message as an array of strings, as it would be parsed by `Shellwords.split`. For example, if the message was "Lita: auth add joe committers", calling `args` would return `["add", "joe", "committers"]`. ("auth" is considered the command and so is not included in the arguments.) This is very handy for commands that take arguments in a way similar to how a UNIX shell would work.
* `message` - A `Lita::Message` object for the incoming message.
* `user` - A `Lita::User` object for the user who sent the message.

Additionally, handlers have access to these top-level methods:

* `robot` - Direct access to the currently running `Lita::Robot` object.
* `redis` - A `Redis::Namespace` object which provides each handler with its own isolated Redis store, suitable for many data persistence and manipulation tasks.
* `http` - A `Faraday::Connection` object for making HTTP requests. Takes an optional hash of options and optional block which are passed on to [Faraday](https://github.com/lostisland/faraday).

If a handler method crashes, the backtrace will be output to Lita's log with the `:error` level, but it will not crash the robot itself.

### HTTP routes

In addition to chat routes, handlers can also define HTTP routes for the built-in web server. This is done with the class-level `http` method. `http` returns a `Lita::HTTPRoute` object, which has methods for the most common HTTP methods. These methods take two arguments: the path for the route, and the name of the method that it will invoke as a symbol. The callback method takes two arguments: a `Rack::Request` and a `Rack::Response`. For example:

``` ruby
http.get "/foo/bar", :baz

def baz(request, response)
  response.body = "Hello, world!"
end
```

### Handler-specific configuration

If you want your handler to expose config settings to the user, use the class-level `default_config` method. This method accepts a single config object as an argument, which will be exposed to the user as `Lita.config.handlers.your_handler_namespace`.

``` ruby
module Lita
  module Handlers
    class HandlerWithConfig < Handler
      def self.default_config(config)
        config.enabled = true
      end
    end
  end
end

Lita.config.handlers.handler_with_config.enabled # => true
```

### Examples

Here is a basic handler which simply echoes back whatever the user says.

``` ruby
module Lita
  module Handlers
    class Echo < Handler
      route /^echo\s+(.+)/, :echo, help: { "echo FOO" => "Echoes back FOO." }

      def echo(response)
        response.reply(response.matches)
      end
    end

    Lita.register_handler(Echo)
  end
end
```

Here is a handler that tells a user who their United States congressional representative is based on zip code with data from a fictional HTTP API. The results are saved in the handler's namespaced Redis store to save HTTP calls on future requests.

``` ruby
module Lita
  module Handlers
    class Representative < Handler
      route /representative\s+(\d{5})/, :lookup, command: true, help: {
        "representative ZIP_CODE" => "Looks up the United States congressional representative for your zip code."
      }

      def lookup(response)
        zip = response.matches[0][0]
        rep = redis.get(zip)
        rep = get_rep(zip) unless rep
        response.reply "The representative for #{zip} is #{rep}."
      end

      private

      def get_rep(zip)
        http_response = http.get(
          "http://www.example.com/api/represenative",
          zip_code: zip
        )

        data = MultiJson.load(http_response.body)
        rep = data["representative"]["name"]
        redis.set(zip, rep)
        rep
      end
    end

    Lita.register_handler(Representative)
  end
end
```

For more detailed examples, check out the built in authorization, help, and web handlers, or external handlers [lita-karma](https://github.com/jimmycuadra/lita-karma) and [lita-google-images](https://github.com/jimmycuadra/lita-google-images). See the API documentation for exact specifications for handlers' methods.

## Testing

It's a core philosophy of Lita that any plugins you write for your robot should be as thoroughly tested as any other program you would write. To make this easier, Lita ships with some handy extras for [RSpec](https://github.com/rspec/rspec) that make testing a handler dead simple. They require the full RSpec suite (rspec-core, rspec-expectations, and rspec-mocks) version 2.14 or higher, as they use the newer `expect(obj).to receive(:message)` syntax.

### Testing handlers

To include Lita's RSpec extras for testing a handler, require "lita/rspec", then add `lita_handler: true` to the metadata for the example group.

``` ruby
require "lita/rspec"

describe Lita::Handlers::MyHandler, lita_handler: true do
  # ...
end
```

This provides the following:

* All Redis interaction will be namespaced to a test environment and automatically cleared out before each example.
* Lita's logger is stubbed to prevent log messages from cluttering up your test output.
* Lita's configuration is cleared out before each example, so that the first call to `Lita.config` will start from the default configuration.
* `Lita.handlers` will return an array with only the class you're testing (`described_class`).
* Strings sent with `Lita::Robot#send_messages` will be pushed to an array accessible as `replies` so you can make expectations about output from the robot.
* You have access to the following cached objects set with `let`: `robot`, `source`, and `user`. Note that these objects are instances of the real classes and not test doubles.

The custom helper methods are where `Lita::RSpec` really shines. You can test routes (both chat and HTTP routes) very easily using this syntax:

``` ruby
it { routes("some message").to(:some_method) }
it { routes_command("directed message").to(:some_command_method) }
it { doesnt_route("message").to(:some_command_method) }
it { routes_http(:get, "/foo/bar").to(:baz) }
it { doesnt_route_http(:post, "/foo/bar").to(:baz) }
```

* `routes` - Sets an expectation that the given string will trigger the given method when overheard by the robot.
* `routes_command` - Sets an expectation that the given string will trigger the given method when directed at the robot, either in a private message, or by prefixing a message in a chat room with the robot's mention name.
* `doesnt_route` - Sets an expectation that is the inverse of the one set by `routes`. Also aliased to `does_not_route`.
* `doesnt_route_command` - Sets an expectation that is the inverse of the one set by `routes_command`. Also aliased to `does_not_route_command`.
* `routes_http` - Sets an expectation that an HTTP request with the given HTTP method and path will route to the given handler method.
* `doesnt_route_http` - Sets an expectation that is the inverse of `routes_http`. Also aliased to `does_not_route_http`.

**Note: These routing helpers bypass authorization for routes restricted to authorization groups.**

To send a message to the robot, use `send_message` and `send_command`. Then set expectations about the contents of the `replies` array.

``` ruby
it "lets everyone know when someone is happy" do
  send_message("I'm happy!")
  expect(replies.last).to eq("Hey, everyone! #{user.name} is happy! Isn't that nice?")
end

it "greets anyone that says hi to it" do
  send_command("hi")
  expect(repliest.last).to eq("Hello, #{user.name}!")
end
```

If you want to send a message or command from a user other than the default test user (set up for you with `let(:user)` by `Lita::RSpec`), you can invoke either method with the `:as` option, supplying a `Lita::User` object.

``` ruby
it "lets everyone know that Carl is happy" do
  carl = Lita::User.create(123, name: "Carl")
  send_message("I'm happy!", as: carl)
  expect(replies.last).to eq("Hey, everyone! Carl is happy! Isn't that nice?")
end
```

* `send_message(string, as: user)` - Sends the given string to the robot.
* `send_command(string, as: user)` - Sends the given string to the robot, prefixing it with the robot's mention name.

### Testing adapters or other code

If you use `lita: true` instead of `lita_handler: true` in the metadata for your example group, only a small subset of Lita's RSpec extras will be enabled:

* All Redis interaction will be namespaced to a test environment and automatically cleared out before each example.
* Lita's logger is stubbed to prevent log messages from cluttering up your test output.
* Lita's configuration is cleared out before each example, so that the first call to `Lita.config` will start from the default configuration.

## Running as a daemon

Lita has built-in support for daemonization on Unix systems. When run as a daemon, Lita will redirect standard output and standard error to a log file, and write the process ID to a PID file. To start Lita as a daemon, run `lita -d`. There are additional command line flags for specifying the path of the log and PID files, which override the defaults. If an existing Lita process is running when `lita -d` is invoked, Lita will abort and leave the original process running, unless the `-k` flag is specified, in which case it will kill the existing process. Run `lita help` for information about all the possible command line flags.

## Deploying to Heroku

There are a few things worth mentioning when deploying an instance of Lita to Heroku:

1. Your Procfile should contain one process: `web: bundle exec lita`.

1. To use the Redis To Go add-on and the HTTP port set by Heroku, configure Lita like this:

    ``` ruby
    Lita.configure do |config|
      config.redis.url = ENV["REDISTOGO_URL"]
      config.http.port = ENV["PORT"]
    end
    ```

1. Consider using a service like [Uptime Robot](http://www.uptimerobot.com/) to monitor your Lita instance and keep it from [sleeping](https://blog.heroku.com/archives/2013/6/20/app_sleeping_on_heroku) when running on a free dyno. `/lita/info` is a reliable path to hit from the web to keep it running.

## API documentation

Complete documentation for all of Lita's classes and methods can be found at [rdoc.info](http://rdoc.info/gems/lita/frames).

## Available plugins

* [Adapters](https://github.com/jimmycuadra/lita/wiki/Adapters)
* [Handlers](https://github.com/jimmycuadra/lita/wiki/Handlers)

If you release a Lita plugin of your own, be sure to add it to one of the above lists!

## Questions, feedback, and discussion

* [Google Group](http://groups.google.com/group/litaio)
* [IRC](https://webchat.freenode.net/) (`#lita.io` on the Freenode network)

## Bug reports

* [GitHub Issues](https://github.com/jimmycuadra/lita/issues)

## Contributing

See the [contribution guide](https://github.com/jimmycuadra/lita/blob/master/CONTRIBUTING.md).

## History

For a history of releases, see the [Releases](https://github.com/jimmycuadra/lita/releases) page.

## License

[MIT](http://opensource.org/licenses/MIT)
