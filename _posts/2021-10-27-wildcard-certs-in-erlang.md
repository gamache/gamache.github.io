---
layout: post
title: Handling Wildcard Certs in Elixir and Erlang
author: Pete Gamache
date: 2021-10-27
tags: elixir
---
For a language that is so good at TCP communications, Erlang (and by
extension, Elixir) does not make it dead simple to make TLS
connections&mdash;you know, like HTTPS requests. And the rules change
slightly if the remote host is using a wildcard certificate,
like `https://www.example.com` using a certificate for `*.example.com`.
This post has the configuration you need.

Recently, I had to handle a websocket connection to a Node.js server
from within a genserver. I chose the [Gun](https://github.com/ninenines/gun)
client, as it has the features I need and the developer has a great
track record. Gun supports HTTP/1.1 and 2.0, websockets (over HTTP/1.1),
and TLS with an interface that is convenient to the Erlang/OTP model
of programming.

Gun, unsurprisingly, relies on the native Erlang modules for handling
TCP connections, `:gen_tcp` and `:ssl`. `:ssl` won't connect to
anything before being pointed to some certificates that can verify that
a remote host is who they say they are. But Gun offers a `:tcp_opts`
option that passes through to the `:ssl` module, so we ought to be able
to configure our way out of trouble.

Most relevant to the average Elixir or Erlang developer are the
following `:ssl` options, shown here as passed to Gun:

{% highlight elixir %}
[
  tls_opts: [
    verify: :verify_peer,
    cacertfile: CAStore.file_path()
  ]
]
{% endhighlight %}

The `:verify_peer` option tells the `:ssl` module that we intend to
verify certificates (an insecure `:verify_none` option exists, but we
will not discuss it here).

Once you've provided this option, you need to
also give `:cacerts` (a collection of root certificates) or `:cacertfile`
(a file containing a collection of root certificates in PEM encoding).
This is the "missing piece" that is not included in the Erlang or Elixir
standard distribution.

Where can it be found? Two popular choices of library that provide
Mozilla's freely available certificate chain in the
Erlang/Elixir world are [Certifi](https://github.com/certifi/erlang-certifi)
and [CAStore](https://github.com/elixir-mint/castore). I used CAStore
because I was already using it in the project.

Great. These options ought to be enough to make a WSS connection, right?

Well, sometimes. Gun makes it clear in the docs that websocket connections
are only supported over HTTP/1.1. If your server uses HTTP/2, you're out
of luck. Unless you force HTTP 1.1 in Gun's `:protocols` option:

{% highlight elixir %}
[
  protocols: [:http],
  tls_opts: [
    verify: :verify_peer,
    cacertfile: CAStore.file_path()
  ]
]
{% endhighlight %}

That takes care of TLS and websockets. That's ought to be all we
need, right?

Well, not completely. This part was news to me. It turns out that
Erlang's `:ssl` module does not handle the certificates our production
servers were using, and the reason is because the certs are
wildcard-scoped instead of single-hostname.

The next part wasn't science: I googled it. After hacking around a
bit I found the `customize_hostname_check` option below:

{% highlight elixir %}
[
  protocols: [:http],
  tls_opts: [
    verify: :verify_peer,
    cacertfile: CAStore.file_path(),
    customize_hostname_check: [
      match_fun: :public_key.pkix_verify_hostname_match_fun(:https)
    ]
  ]
]
{% endhighlight %}

And finally, Gun was able to open a websocket.

