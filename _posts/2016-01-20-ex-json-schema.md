---
layout: post
title: "Validating Data in Elixir with ExJsonSchema"
date: 2016-01-20 09:22:00 -0500
author: Pete Gamache
excerpt_separator: <!--more-->
tags: elixir api json schema
---

Any good service developer has spent a lot more time than they originally
planned to spend on validating their input and output data.  They
probably started with validation-by-gauntlet, i.e. if nothing breaks
then the data was valid, and if something breaks, tough.  Then on to
basic validations -- this field should be an integer, this field should
look like an email address, that sort of thing -- and from there, they
piled on custom and semi-custom validations until there was a cozy
little robin's nest of validation code.

Oh yeah, and that nest of code has to emit good error messages.  And it
needs to be maintained forever.

<!--more-->

There is a better way.  The [JSON Schema](http://json-schema.org/)
format allows one to describe, using JSON structures, the form of
different objects their app deals with.  Support is included for required
fields, data type validation, collections (arrays) of data, and
references to other JSON Schemas that may or may not originate from the
same document.  It's a very simple format, but quite powerful for
describing and enforcing constraints on data formats.

[ExJsonSchema](https://github.com/jonasschmidt/ex_json_schema) is an
Elixir library which handles the loading of JSON Schemas and the
verification of data against them.  In particular, it offers the
`ExJsonSchema.Validator.validate` function, which takes as input a JSON
Schema and a piece of data, and returns a list of validation errors and
where they occurred.  With just a little bit of effort, it's easy to
wrap this functionality so that it can be used easily within your app.

## Make a Schema

The first step is to create a JSON Schema document.  We called ours
`schema.json` and put it in the root directory of our app.  The full
JSON Schema specification is outside the scope of this post, so instead,
here's an example schema for a theoretical events API.  This API takes
in `event_collection` objects, which contain an array of `event`
objects.

{% highlight json %}
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "title": "Events API Schema",

  "definitions": {

    "event_collection": {
      "type": "object",
      "required": ["events"],
      "properties": {
        "events": {
          "type": "array",
          "items": {
            "$ref": "#/definitions/event"
          }
        }
      }
    },

    "event": {
      "type": "object",
      "required": ["name", "timestamp"],
      "properties": {
        "name": {
          "type": "string"
        },
        "timestamp": {
          "type": ["string", "integer"],
          "format": "date-time"
        },
        "attributes": {
          "type": "object"
        }
      }
    }
  }
}
{% endhighlight %}

A couple of things about this:

* We've declared a nested data structure `event-collection`, which
  references another data type `event` from the same document using
  `$ref` and a [JSON Pointer](https://tools.ietf.org/html/rfc6901)
  string.

* Event `timestamp` fields can be in either integer (like a Unix
  timestamp) or string (like ISO8601 date-times) format.  If provided
  in string format, the JSON Schema will validate it as a date-time.

* Rather than creating separate JSON Schema files for each of the
  data types, we place our data type schemata together under the
  `definitions` key.


## Test the Schema (Find Problems)

Now that we have a JSON Schema to work from, we need to load it into
ExJsonSchema.  This involves reading the file from disk, JSON decoding
it, and passing it to `ExJsonSchema.Schema.resolve`:

{% highlight iex %}
iex> schema = File.read!("schema.json") |> Poison.decode! |> ExJsonSchema.Schema.resolve
{% endhighlight %}

The `resolve` call is potentially expensive, as it may reach out to
external network resources to do its job.  We'll want to make sure we
only call it once.

We're ready to start validating some data.  Since our data type schemata
are under the `definitions` key, we need to point to the schema we want
when we call `validate`.  We can do it like this:

{% highlight iex %}
iex> event_schema = schema.schema["definitions"]["event"]

iex> ExJsonSchema.Validator.validate(schema, event_schema, %{})
[{"Required property name was not present.", []},
 {"Required property timestamp was not present.", []}]

iex> ExJsonSchema.Validator.validate(schema, event_schema,
...> %{"name" => "hi", "timestamp" => 1})
[]
{% endhighlight %}

That `schema.schema["definitions"]["your-data-type-here"]` thing is
going to get old fast.

Anyway, it works on single data structures;
let's check out nested data.

{% highlight iex %}
iex> event_collection_schema = schema.schema["definitions"]["event_collection"]
iex> ExJsonSchema.Validator.validate(schema, event_collection_schema,
...> %{"events" => [
...>    %{"name" => "event 1", "attributes" => %{"awesome" => true}},
...>    %{"name" => "event 2", "timestamp" => "whenever"},
...>    %{"name" => "event 3", "timestamp" => 1234567890}
...> ]})
[{"Required property timestamp was not present.", ["events", 0]},
 {"Expected \"whenever\" to be a valid ISO 8601 date-time.",
  ["events", 1, "timestamp"]}]
{% endhighlight %}

Excellent, it's not only giving us good error messages, it's giving us a
path to where the errors occurred: at indices 0 and 1 of the `events`
array.  We can use that to build some handsome validation error
messages.

Alas, there are a few gotchas.

The first is that the schema validation only works on maps with string
keys.

{% highlight iex %}
iex> ExJsonSchema.Validator.validate(schema, event_schema,
...> %{name: "hi", timestamp: 1})
[{"Required property name was not present.", []},
 {"Required property timestamp was not present.", []}]
{% endhighlight %}

This isn't too surprising; allocating symbols at runtime is
frowned upon because they don't get garbage-collected, and JSON only
supports string keys anyhow.

The second is more of a bug than a gotcha.  ExJsonSchema falls over
when validating seriously malformed nested objects:

{% highlight iex %}
iex> ExJsonSchema.Validator.validate(schema, event_collection_schema,
...> %{"events" => [ 22 ] })
** (BadMapError) expected a map, got: 22
{% endhighlight %}

Though it's inconvenient, it does provide an indication of malformed
data, so at least it can be made useful.

At this point we've identified several things we'll want to abstract
away:

* `ExJsonSchema.Schema.resolve` should only be called once;
* Typing out `schema.schema["definition"]` is brutish and should be
  avoided;
* Input data should be transformed so that all map keys become
  strings;
* Putting non-objects where nested objects should go causes a
  `BadMapError` to be raised.
* The output from `ExJsonSchema.Validator.validate` is not suitable
  for direct inclusion in a JSON document, because it contains tuples.
  The validation errors need to be massaged before sending them to
  the end user.


## Wrap ExJsonSchema (Create Solutions)

The first criterion above, that `resolve` should only be called once,
implies that we need persistent state.  One idiomatic approach to this
in Elixir is implementing
[GenServer](http://elixir-lang.org/docs/v1.1/elixir/GenServer.html),
a generic interface which models any client/server interaction.
(This does not imply communicating across a network; it's just an
abstraction around managing access to state.)

We start by adding our GenServer module to the [application supervision
tree](http://elixir-lang.org/getting-started/mix-otp/supervisor-and-application.html).
This takes place in `lib/myapp.ex`, like so:

{% highlight elixir %}
  # See http://elixir-lang.org/docs/stable/elixir/Application.html
  # for more information on OTP Applications
  def start(_type, _args) do
    import Supervisor.Spec, warn: false

    children = [
      # Define workers and child supervisors to be supervised
      # worker(MyApp.Worker, [arg1, arg2, arg3]),
      worker(JsonSchema, [[name: :json_schema]])
    ]

    # See http://elixir-lang.org/docs/stable/elixir/Supervisor.html
    # for other strategies and supported options
    opts = [strategy: :one_for_one, name: MyApp.Supervisor]
    Supervisor.start_link(children, opts)
  end
{% endhighlight %}


We then must create our GenServer module,
overriding `init` and `handle_call` to fulfill just enough
of the GenServer interface.  In Erlang parlance,
a `call` is a synchronous request, and a `cast` is async.  We're only
interested in synchronous function calls -- after all, what good is a
validator that doesn't give you answers -- so we won't implement
`handle_cast`.

{% highlight elixir %}
defmodule JsonSchema do
  @moduledoc ~S"""
  A service which validates objects according to types defined
  in `schema.json`.
  """

  use GenServer

  def init(_) do
    schema = File.read!("./schema.json")
             |> Poison.decode!
             |> ExJsonSchema.Schema.resolve
    {:ok, schema}
  end

  def handle_call({:validate, object, type}, _from, schema) do
    errors = get_validation_errors(object, type, schema)
             |> transform_errors
    {:reply, errors, schema}
  end
{% endhighlight %}

Those two functions are enough to allow interaction using the GenServer
module directly, but the syntax is awkward enough that it's worth
pouring some sugar on.  Let's add a `JsonSchema.validate` function
that makes the GenServer call for us.

{% highlight elixir %}
  def validate(server \\ :json_schema, object, type) do
    GenServer.call(server, {:validate, object, type})
  end
{% endhighlight %}

Pay attention to that default value of `:json_schema`; it will come up
later.

Next, let's implement `get_validation_errors/3`, which is invoked from
`handle_call`.  We can take care of smoothing over
`schema.schema["definition"]` and catching exceptions here.

{% highlight elixir %}
  defp get_validation_errors(object, type, schema) do
    type_string = type |> to_string
    type_schema = schema.schema["definitions"][type_string]

    not_a_struct = case object do
      %{__struct__: _} -> Map.from_struct(object)
      _ -> object
    end

    string_keyed_object = ensure_key_strings(not_a_struct)

    ## validate throws a BadMapError on certain kinds of invalid
    ## input; absorb it (TODO fix ExJsonSchema upstream)
    try do
      ExJsonSchema.Validator.validate(schema, type_schema, string_keyed_object)
    rescue
      _ -> [{"Failed validation", []}]
    end
  end
{% endhighlight %}

So where are we now?  We can keep state, we can validate objects against
individual schemata, and we can catch exceptions thrown by ExJsonSchema.
We still need to convert map keys to strings, and transform error messages
into a JSON-compatible data structure.

Next!

{% highlight elixir %}
  @doc ~S"""
  Makes sure that all the keys in the map are strings and not atoms.
  Works on nested data structures.
  """
  defp ensure_key_strings(x) do
    cond do
      is_map x ->
        Enum.reduce x, %{}, fn({k,v}, acc) ->
          Map.put acc, to_string(k), ensure_key_strings(v)
        end
      is_list x ->
        Enum.map(x, fn (v) -> ensure_key_strings(v) end)
      true ->
        x
    end
  end
{% endhighlight %}

(Yeah, I know that `@doc` is ignored for private methods. It's still
the best way to document them.)

And to get the validation errors into JSON-compatible format, one easy way
is to just collect the error messages themselves:

{% highlight elixir %}
  def errors_to_json(errors) do
    errors |> Enum.map(fn ({msg, _cols}) -> msg end)
  end
{% endhighlight %}

That's just about all there is to build.  Just about.


## Make It Production-ready

One more thing remains before this code is ready for prime time.  We need
to make sure that it works when packaged as a
[release](http://www.erlang.org/doc/design_principles/release_structure.html).

We need to ensure that `schema.json` is included in the release. The easiest
way to do this is to move it to a directory which is already getting included
in the release, such as `lib`, or `priv` on Phoenix applications.  Let's use
`priv` for the example.

To reference a file within the app's source code directory, use the
`Application.app_dir/1` function with the name of your app:

{% highlight elixir %}
  def init(_) do
    schema = File.read!(Application.app_dir(:myapp) <> "/priv/schema.json")
             |> Poison.decode!
             |> ExJsonSchema.Schema.resolve
    {:ok, schema}
  end
{% endhighlight %}

You may of course put "/priv/schema.json" into a config parameter if you like.

Now we have a robust JSON Schema validation system, ready for usage in
our app and tests.

Let's use it!

## Validating Input Data

We can use our JSON Schema to ensure that data given as input to our app
(e.g., JSON data from a POST request) is well-formed.  Here's a tiny example
from a Phoenix controller:

{% highlight elixir %}
defmodule MyApp.EventsController do
  use MyApp.Web, :controller

  plug :validate_params

  def save_events(conn, params) do
    event_collection = conn.assigns[:event_collection]
    # ... do something here
    conn |> put_status(202) |> json(%{ok: true})
  end

  defp validate_params(conn, _params) do
    case JsonSchema.validate(conn.params, :event_collection) do
      [] ->
        conn |> assign(:event_collection, conn.params)
      errors ->
        json_errors = errors |> JsonSchema.errors_to_json
        conn |> put_status(422) |> json(%{errors: json_errors}) |> halt
    end
  end
end
{% endhighlight %}

Using the JSON Schema, we reduce the burden of writing validation code
ourselves and scattering it around the codebase.  If we've written the
JSON Schema correctly and the input data passes validation, we don't need
validation in our downstream functions.


## Validating Output Data

Validating output data all the time may be impractical; for instance,
validating every single response served by an API may add too much
response latency.  But we can validate against the JSON Schema in our
tests quite easily.  Using a Phoenix controller test as an example:

{% highlight elixir %}
defmodule MyApp.EventsControllerTest do
  use Plug.Test

  test "it returns well-formed event collection" do
    resp = conn(:post, "url goes here", %{params: ...}) |> MyApp.Router.call(MyApp.Router.init([]))
    resp_object = resp.resp_body |> Poison.decode!
    assert([] == JsonSchema.validate(resp_object, :event_collection))
  end

  # ...
end
{% endhighlight %}


## Summary

We were faced with a problem: validating data and generating good validation
error messages is undoubtedly a best practice, but it's a time sink and can
become a real PITA when nested objects are involved.

We solved this problem using the best tools available on the open market:
the JSON Schema internet draft standard, and an open-source Elixir library
to use JSON Schemas.  We improved the usability of this JSON Schema
library so that we could use it liberally in our code.  And finally, we
used this service we created to improve the correctness of our app on both
the input and output sides.

This post uses Elixir, but the technique of using JSON Schema to validate
input and output data is widely applicable, and pretty convenient to boot.
I'll be using this trick again and again.

<br>

<i>Code for this post is [available on GitHub](https://gist.github.com/gamache/e8e24eee5bd3f190de23).</i>

