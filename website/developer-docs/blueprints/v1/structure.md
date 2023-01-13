# File structure

Blueprints are YAML files, which can use some [additional tags](./tags.md) to ease blueprint creation.

## Structure

```yaml
# The version of this blueprint, currently 1
version: 1
# Optional block of metadata, name is required if metadata is set
metadata:
    # Arbitrary key=value store, special labels are listed below
    labels:
        foo: bar
    name: example-blueprint
# Optional default context, instance context is merged over this.
context:
    foo: bar
# List of entries (required)
entries:
    - # Model in app.model notation, possibilities are listed in the schema (required)
      model: authentik_flows.flow
      # The state this object should be in (optional, can be "present", "created" or "absent")
      # Present will keep the object in sync with its definition here, created will only ensure
      # the object is created (and create it with the values given here), and "absent" will
      # delete the object
      state: present
      # An optional list of boolean-like conditions. If all conditions match (or
      # no condiitons are provided) the entry will be evaluated and acted upon
      # as normal. Otherwise, the entry is skipped as if not defined at all.
      # Each condition will be evaluated in Python to its boolean representation
      # bool(<condition>). Furthermore, complex conditions can be built using
      # a special !Condition tag. See the documentattion for custom tags for more
      # information.
      conditions:
          - true
          - text
          - 2
          - !Condition [AND, ...] # See custom tags section
      # Key:value filters to uniquely identify this object (required)
      identifiers:
          slug: initial-setup
      # Optional ID for use with !KeyOf
      id: flow
      # Attributes to set on the object. Only explicitly required settings should be stated
      # as these values will override existing attributes
      attrs:
          denied_action: message_continue
          designation: stage_configuration
          name: default-oobe-setup
          title: Welcome to authentik!
```

## Special Labels

The following special labels are applied as key-value children under the `labels` map.

```yaml
...
metadata:
    name: your-blueprint
    labels:
        blueprints.goauthentik.io/instantiate: "true"
...
```

#### `blueprints.goauthentik.io/system`:

Used by authentik's packaged blueprints to keep globals up-to-date. Should only be removed in special cases.

This label expects a value type of *string*, and can be `"true"` or the key and value should be removed entirely.

#### `blueprints.goauthentik.io/instantiate`:

Configure if this blueprint should automatically be instantiated (defaults to `"true"`). When set to `"false"`, blueprints are listed and available to be instantiated via API/Browser.

This label expects a value type of *string*, and can either be `"true"` or `"false"`.

## Quick YAML Refesher

Special attention should be paid to consistent indentation, spelling, value types, and special characters (colons and dashes). The aforementioned pain points are where most YAML errors occur.

YAML contains scalars, maps, and lists. The first level of a generic YAML file can have as many key-values as you like as it is effectively a map.

### Scalar

Scalars are the simplest type of value. They can either be numeric (`2023`), a string (`"hello world"`), a boolean (`true` or `false`), or `null`.

Be sure to check the expected scalar value is of the correct type. Confusion often occurs with strings such as `"true"` and some other type like a bools `true`. This is especially true since you don't need to use quotations to indicate a value is a string. However if a value does not fit into any other type it will automatically become a string. For instance `True` with a capital T is not a bool in all parsers, and thus may or may not be interpreted as a string. As a general rule stick to lower case true and false for bools.

```yaml
string: "true"
maybeString: True
bool: true
numeric: 1
nullVal: null
```

Also remember all keys must be strings, so in the adjacent example I could not have used `null: null` as the key (before the colon) is a `null` value type. Thus why I called it `nullVal` to make it a string and prevent it being interpreted as an actual `null` value

### Map

Maps, also known as dictionaries and associative arrays, depending on what spheres you come from. They can all hold marginally different meanings but here they are used interchangeably.

Fundamentally a map is an associative data structure, it stores a set of keys, and associates them to specific values. These values themselves can often be other maps, and lists.

```yaml
metadata:
    labels:
        foo: bar
    name: example-blueprint
```
In the adjoining example `metadata` is a map. The `metadata` map has two key value pairs, the key `labels` (another map), and the key `name` (a plain key-value scalar).

The keys of a map must be indented to the same level, and must be at a deeper indentation than the map name, in this case `metadata`. These must be consistent between each key.

An empty map can be defines like so, thanks to YAML being a superset of JSON:

```yaml
metadata: {}
```

Note this is not the same as the map not existing. Also map values can be in any order. Unlike a list the order is always non-consequential so you can sometimes see maps being auto-generated in ways you may not immediately recognise.

### List

A list is a collection of values.

```yaml
entries:
- someValue
```

An empty list can be defined like so thanks to YAML being a superset of JSON:
```yaml
entries: []
```

Each value must be on a new line, and must start with a hyphen and a space after it. They do not need to be on a new level. So you can sometimes see it as above or indented.

```yaml
entires:
    - someValue
```

Whichever you pick, make sure it is consistent throughout. Lastly just like any other values they can be scalars, lists, or maps of their own. Follows is an example in the blueprint above with the first / flattened indentation style with a list (`conditions`) as a value in another list (`entries`).

```yaml
...
entries:
- model: authentik_flows.flow
  state: present
  conditions:
  - true
  - text
  - 2
...
```
