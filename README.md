#	hugo-mod-time

[![Project Status: WIP – Initial development is in progress, but there has not yet been a stable,
usable release suitable for the public.][wip-svg]][wip]

[wip-svg]: https://www.repostatus.org/badges/latest/wip.svg
[wip]: https://www.repostatus.org/#wip

> This module is still in early development. The partial names and available options may change in
future versions. Hopefully there will be few breaking changes, and warnings will be used where
possible to signal deprecated options.

A module for the [Hugo] static site generator, which generates correctly formatted `<time>` elements.
It also supports ordinals for day formatting. Additionally, it supports including GitHub's
[relative-time-element] package, automatically updating dates and times on the client, according to
their location.

[Hugo]: https://gohugo.io/
[relative-time-element]: https://github.com/github/relative-time-element

##	Installation

Add hugo-mod-time to your Hugo project configuration

-	config.toml

	```toml
	[module]
	[[module.imports]]
	path = 'github.com/kcastellino/hugo-mod-time'
	```

-	config.yaml

	```yaml
	module:
	imports:
	- path: github.com/kcastellino/hugo-mod-time
	```

If you are not using the [client-side time conversion] option, and you don't want to import the
third-party module for that feature, set `ignoreImports` on the module import to `true`:

-	config.toml

	```toml
	[module]
	[[module.imports]]
	path = 'github.com/kcastellino/hugo-mod-time'
	ignoreImports = true
	```

-	config.yaml

	```yaml
	module:
	imports:
	- path: github.com/kcastellino/hugo-mod-time
	  ignoreImports: true
	```

[client-side time conversion]: #enabling-client-side-time-conversion

##	How to use

To insert a date without a time, use the `date` partial with an appropriate format string:

```gotmpl
{{ partial "date" (dict "Time" $time "Format" "2 January 2006") }}
```

To insert a complete datetime, use the `time` partial:

```gotmpl
{{ partial "time" (dict "Time" $time "Format" "2 January 2006 3:04 PM MST") }}
```

Both partials take a dictionary as the 'context' (the single argument), which requires the
following properties:

<dl>
<dt><strong>Time</strong></dt>
<dd>
A <a href="https://godoc.org/time#Time">time.Time</a> struct. Page variables such as
<code>.PublishDate</code> and <code>.Lastmod</code> can be passed straight into this argument;
timestamps can be converted to objects using the <code>time</code> function.
</dd>

<dt><strong>Format</strong></dt>
<dd>
A <a href="https://gohugo.io/functions/format/#gos-layout-string">Go Layout String</a> as used by
the <code>time.Format</code> function to render a datetime.
</dd>

<dt><strong>Options</strong></dt>
<dd>
A dictionary containing key-value pairs corresponding to the
<a href="https://github.com/github/relative-time-element#attributes">attributes for the
<code>&lt;relative-time&gt;</code> element</a>
</dd>
</dl>

The `time` partial above will produce the following HTML:

```html
<time datetime="2023-01-30T00:00:00+1100">30 January 2023 12:00 AM AEDT</time>
```

###	Using ordinals in dates

With the `date` and `time` partials, you can also use an ordinal in the layout string to signify
you would like dates to be formatted as such:

```gotmpl
{{ partial "date" (dict "Time" $time "Format" "January 2nd 2006") }}
```

This case is specially handled to produce the following HTML:

```html
<time datetime="2023-01-30">January 30th 2023</time>
```

##	Enabling client-side time conversion

Client-side time conversion is not enabled by default, as it requires including
[relative-time-element] as an additional dependency. To enable it, you need to set
`params.time.enableRelative` in your config to `true`.

-	config.toml:

	```toml
	[params.time]
	enableRelative = true
	```

-	config.yaml:

	```yaml
	params:
	  time:
	    enableRelative: true
	```

You will also need to include the JavaScript files required to enable the custom elements used.
The simple option is to just include the `time/script-import` partial in your `<head>`

```html
<head>
	<!-- omitted -->
	{{ partialCached "time/script-import" dict }}
</head>
```

> Currently, the "context" provided to the `script-import` partial does nothing, however the next
version will instead provide an [options object to js.Build][js-build-options]. If you do not wish
to set any options, you can use the `dict` function with no options to pass an empty dictionary.

Alternatively, you can import `/assets/relative-time-element/index.ts` into your own bundle.

After enabling relative time conversion, the generated HTML will look like this:

```html
<time datetime="2023-01-30T00:00:00+1100">
	<relative-time datetime="2023-01-30T00:00:00+1100"
		month="long" day="numeric" year="numeric"
		hour="numeric" minute="2-digit" timezonename="short">
		30 January 2023 12:00 AM AEDT
	</relative-time>
</time>
```

The template will use the provided layout string to automatically configure the `relative-time`
element so it will match as close as possible to the date format produced by Hugo.

[js-build-options]: https://gohugo.io/hugo-pipes/js/#options

###	Additional options

You may pass in options to change the appearance of the `relative-time` element, by passing in an
options dictionary. You can configure the element using [any available attribute][relative-time-attrs].

```gotmpl
{{ $timeOptions := dict "format" "relative" "precision" "day" "threshold" "P7D" }}

{{ partial "time" (dict "Time" $time "Format" "2 January 2006 3:04 PM MST" "Options" $timeOptions) }}
```

A default configuration can be provided by setting `params.time.defaultOptions` in your site config:

-	config.toml:

	```toml
	[params.time]
	enableRelative = true

	[params.time.defaultOptions]
	format = "relative"
	precision = "day"
	threshold = "P7D"
	```

-	config.yaml:

	```yaml
	params:
	  time:
	    enableRelative: true
	    defaultOptions:
	      format: "relative"
	      precision: "day"
	      threshold: "P7D"
	```

[relative-time-attrs]: https://github.com/github/relative-time-element#attributes

##	Licensing and contributions

This module is licensed under the University of Illinois/NCSA license, which can be read in
[LICENSE.txt]. It is legally identical to the 3-clause "Modified" BSD License, and contains an extra
clause (Clause 3) compared to the MIT license.

By default, using this module will download the `relative-time-element` package to your computer,
which is created by GitHub, Inc. and licensed under the [MIT license][rte-license]. If you don't
want to download this package, [set `ignoreImports` under your module import to `true`](#installation).
Additionally, the `<relative-time>` element will not be used on generated pages, and the JavaScript
package will not be distributed to clients, unless the
[`enableRelative` option is set to true][client-side time conversion].

[LICENSE.txt]: ./LICENSE.txt
[rte-license]: https://github.com/github/relative-time-element/blob/main/LICENSE

Contributions to this module are welcome! To contribute, please feel free to create an issue or a
pull request in this repository.
