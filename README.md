# Filter plugin for modifying each event record for [Fluentd](http://fluentd.org)

Adding arbitary field to event record without custmizing existence plugin.

For example, generated event from *in_tail* doesn't contain "hostname" of running machine.
In this case, you can use *record_modifier* to add "hostname" field to event record.

## Installation

Use RubyGems:

    gem install fluent-plugin-record-modifier

## Configuration

Use `record_modifier` filter.

    <filter pattern>
      @type record_modifier

      <record>
        gen_host ${hostname}
        foo bar
      </record>
    </filter>

If following record is passed:

```js
{"message":"hello world!"}
```

then you got new record like below:

```js
{"message":"hello world!", "gen_host":"oreore-mac.local", "foo":"bar"}
```

You can also use `record_transformer` like `${xxx}` placeholders and access `tag`, `time`, `record` and `tag_parts` values by Ruby code.

    <filter pattern>
      @type record_modifier

      <record>
        tag ${tag}
        tag_extract ${tag_parts[0]}-${tag_pars[1]}-foo
        formatted_time ${Time.at(time).to_s}
        new_field foo:${record['key1'] + record['dict']['key']} 
      </record>
    </filter>

`record_modifier` is faster than `record_transformer`. See [this comment](https://github.com/repeatedly/fluent-plugin-record-modifier/pull/7#issuecomment-169843012).
But unlike `record_transformer`, `record_modifier` doesn't support following features for now.

- tag_suffix and tag_prefix
- dynamic key placeholder

### record_modifier output

In v0.10, you can use `record_modifier` output to emulate filter. `record_modifier` output doesn't support `<record>` way.

    <match pattern>
      type record_modifier
      tag foo.filtered

      gen_host ${hostname}
      foo bar
    </match>

### char_encoding

Fluentd including some plugins treats the logs as a BINARY by default to forward.
But an user sometimes processes the logs depends on their requirements, e.g. handling char encoding correctly.

`char_encoding` parameter is useful for this case.

```conf
<filter pattern>
  type record_modifier

  # set UTF-8 encoding information to string.
  char_encoding utf-8

  # change char encoding from 'UTF-8' to 'EUC-JP'
  char_encoding utf-8:euc-jp
</filter>
```

### remove_keys

The logs include needless record keys in some cases.
You can remove it by using `remove_keys` parameter.

```conf
<filter pattern>
  type record_modifier

  # remove key1 and key2 keys from record
  remove_keys key1,key2
</filter>
```

If following record is passed:

```js
{"key1":"hoge", "key2":"foo", "key3":"bar"}
```

then you got new record like below:

```js
{"key3":"bar"}
```

### Mixins

* [fluent-mixin-config-placeholders](https://github.com/tagomoris/fluent-mixin-config-placeholders)

## TODO

* Adding following features if needed

    * Use HandleTagNameMixin to keep original tag

    * Replace record value


## Copyright

<table>
  <tr>
    <td>Author</td><td>Masahiro Nakagawa <repeatedly@gmail.com></td>
  </tr>
  <tr>
    <td>Copyright</td><td>Copyright (c) 2013- Masahiro Nakagawa</td>
  </tr>
  <tr>
    <td>License</td><td>MIT License</td>
  </tr>
</table>
