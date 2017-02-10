---
layout: post
title: "Converting a Latin1 encoded HTML Document with Elixir (1)"
date: 2017-02-09
author: carp
comments: true
tags:
  - elixir
  - encoding
teaser: "
Content encoding isn't exactly the first thing that comes to mind when you
ask developers about a fun task. Our first article covering an Elixir topic
illustrates how to make an educated guess about an HTML document's non-unicode
encoding and how to convert it.
"
---

Elixir/Erlang gets some bad rep when it comes to String handling and the
encoding of character data. Also, detecting/guessing the content encoding just
based on heuristics is a [reasonably hard problem][so_detect_content_type] to
solve.

A recent Elixir/Phoenix project involved getting a remote HTML page with
[HTTPoison][httpoison], modifying parts of it and returning the changed HTML
document. It works perfectly fine when the source document is in UTF-8 unicode,
but not so much when the source is ISO-8859-X (Latin1 and its siblings). This
two-parter illustrates two mechanisms to get hints about the source document's
encoding.

Let's assume that the HTML document contains a `<meta http-equiv="Content-Type"
content="text/html; charset=ISO-8859-1" />` element in its document headers.

I'll cover two cases: first, if the `Content-Type` header response from the
remote webserver is missing (is that even allowed?) or not corresponding to
the actual encoding of the response body. And secondly, when the header and
the encoding match. Here goes the first part:

## Part 1: The proper http-equiv meta element

### Guess the content type with this one weird trick


Let's employ [Floki][floki] to get the header element from within the HTML document:

```elixir
defmodule Latin1Convert do

  @doc """
  Retrieves the content type indication from `html`.

  iex>"<html><head><meta http-equiv=\"Content-Type\" content=\"text/html; charset=ISO-8859-1\"></head></html>" |> Latin1Convert.meta_http_equiv_encoding
  "text/html; charset=ISO-8859-1"

  iex>Latin1Convert.meta_http_equiv_encoding("<html></html>")
  ""
  """
  @spec meta_http_equiv_encoding(String.t) :: String.t
  def meta_http_equiv_encoding(html) do
    String.downcase(html)
    |> Floki.attribute("head > meta[http-equiv=content-type]", "content")
    |> List.first
    |> to_string
  end
end
```

It's easier on my brain to map this to atoms:

```elixir
defmodule Latin1Convert do

  @doc """
  Looks for a <meta http-equiv="Content-Type"> node in the input
  string's HTML header and returns an atom representing the encoding.
  """
  @spec content_type_from_header(String.t) :: atom | nil
  def content_type_from_header(html) do
    encoding = meta_http_equiv_encoding(html)

    cond do
      Regex.match?(~r(iso-8859)i, encoding) ->
        :latin1
      Regex.match?(~r(utf-8)i, encoding) ->
        :unicode
      true ->
        nil
    end
  end

  def meta_http_equiv_encoding(html) do
    # See above
  end
end
```

### Convert and purge (now) erroneous markup

The last step would be to convert the HTML input to UTF-8 using the underlying
Erlang library. However, we don't want the HTML to identify as Latin1 anymore,
so we have to remove the `meta http-equiv` tag:

```elixir
defmodule Latin1Convert do

  @doc """
  Convert an input HTML string to UTF-8 unicode.
  """
  @spec call(String.t) :: String.t
  def call(html) do
    content_type = content_type_from_header(html)
    cond do
      content_type == :latin1 ->
        html
        |> :unicode.characters_to_binary(:latin1)
        |> remove_meta_http_equiv_encoding
      true ->
        html
    end
  end

  # Caveat: not really case-sensitive check for the DOM node.
  # Floki doesn't seem to understand `$=foo i` queries. We can't
  # `String.downcase` here as that will mess up the filter chain.
  defp remove_meta_http_equiv_encoding(html) do
    Floki.filter_out(html, "head > meta[http-equiv*=ontent-]")
    |> Floki.raw_html
  end

  def content_type_from_header(html) do
    # see above
  end

  def meta_http_equiv_encoding(html) do
    # See above
  end
end
```

The next part will cover how to take a matching `Content-Type` HTTP header to
short-circuit the guesswork. Check out the file so far:

{% gist carpodaster/2809ebdda26d016860b438e70bc842f2 %}


[so_detect_content_type]: http://stackoverflow.com/questions/3034714/set-a-script-to-automatically-detect-character-encoding-in-a-plain-text-file-in
[httpoison]: https://github.com/edgurgel/httpoison "An Elixir HTTP library"
[floki]: https://github.com/philss/floki
