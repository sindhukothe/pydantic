As well as accessing model attributes directly via their names (eg. `model.foobar`), models can be converted
and exported in a number of ways:

## `model.dict(...)`

The primary way of converting a model to a dictionary. Sub-models will be recursively converted to dictionaries.

Arguments:

* `include`: fields to include in the returned dictionary, see [below](#advanced-include-exclude)
* `exclude`: fields to exclude from the returned dictionary, see [below](#advanced-include-exclude)
* `by_alias`: whether field aliases should be used as keys in the returned dictionary, default `False`
* `skip_defaults`: whether fields which were not set when creating the model and have their default values should
  be excluded from the returned dictionary, default `False`

Example:

```py
{!./examples/export_dict.py!}
```

(This script is complete, it should run "as is")

## `dict(model)` and iteration

*pydantic* models can also be converted to dictionaries using `dict(model)`, you can also
iterate over a model's field using `for field_name, value in model:`. Here the raw field values are returned, eg.
sub-models will not be converted to dictionaries.

Example:

```py
{!./examples/export_iterate.py!}
```

(This script is complete, it should run "as is")

## `model.copy(...)`

`copy()` allows models to be duplicated, this is particularly useful for immutable models.

Arguments:

* `include`: fields to include in the returned dictionary, see [below](#advanced-include-exclude)
* `exclude`: fields to exclude from the returned dictionary, see [below](#advanced-include-exclude)
* `update`: dictionaries of values to change when creating the new model
* `deep`: whether to make a deep copy of the new model, default `False`

Example:

```py
{!./examples/export_copy.py!}
```

(This script is complete, it should run "as is")

## `model.json(...)`

The `.json()` method will serialise a model to JSON. Typically, `.json()` in turn calls `.dict()` and
serialises its result. (For models with a [custom root type](models.md#custom-root-types), after calling `.dict()`,
only the value for the `__root__` key is serialised.)

Serialisation can be customised on a model using the `json_encoders` config property, the keys should be types and
the values should be functions which serialise that type, see the example below.

Arguments:

* `include`: fields to include in the returned dictionary, see [below](#advanced-include-exclude)
* `exclude`: fields to exclude from the returned dictionary, see [below](#advanced-include-exclude)
* `by_alias`: whether field aliases should be used as keys in the returned dictionary, default `False`
* `skip_defaults`: whether fields which were not set when creating the model and have their default values should
  be excluded from the returned dictionary, default `False`
* `encoder`: a custom encoder function passed to the `default` argument of `json.dumps()`, defaults to a custom
  encoder designed to take care of all common types
* `**dumps_kwargs`: any other keyword argument are passed to `json.dumps()`, eg. `indent`.

Example:

```py
{!./examples/export_json.py!}
```

(This script is complete, it should run "as is")

By default timedelta's are encoded as a simple float of total seconds. The `timedelta_isoformat` is provided
as an optional alternative which implements ISO 8601 time diff encoding.

See [below](#custom-json-deserialisation) for details on how to use other libraries for more performant JSON encoding
and decoding

## `pickle.dumps(model)`

Using the same plumbing as `copy()` *pydantic* models support efficient pickling and unpicking.

```py
{!./examples/export_pickle.py!}
```

(This script is complete, it should run "as is")

## Advanced include and exclude

The `dict`, `json` and `copy` methods support `include` and `exclude` arguments which can either be
sets or dictionaries, allowing nested selection of which fields to export:

```py
{!./examples/advanced_exclude1.py!}
```

The `...` value indicates that we want to exclude or include entire key, just as if we included it in a set.

Of course same can be done on any depth level:

```py
{!./examples/advanced_exclude2.py!}
```

Same goes for `json` and `copy` methods.

## Custom JSON (de)serialisation

To improve the performance of encoding and decoding JSON, alternative JSON implementations can be used via the
[`json_loads`` and `json_dumps` properties of `Config`, e.g. `ujson](https://pypi.python.org/pypi/ujson).

```py
{!./examples/json_ujson.py!}
```

(This script is complete, it should run "as is")

`ujson` generally cannot be used to dump JSON since it doesn't support encoding of objects like datetimes and does
not accept a `default` fallback function argument. To do this you may use another library like
[orjson](https://github.com/ijl/orjson).

```py
{!./examples/json_orjson.py!}
```

(This script is complete, it should run "as is")

Note that `orjson` takes care of `datetime` encoding natively, making it faster than `json.dumps` but
meaning you cannot always customise encoding using `Config.json_encoders`.