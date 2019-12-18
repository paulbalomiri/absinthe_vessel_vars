# AbsintheVesselVars

Implements `FilterScope` and `Anchor` for variables, which are transported through a vessel variable.

Usefulness by example:

## Terms

### Vessel

  The vessel is actual type used for transporting variable values of lower level values in the query tree.
  The place where it is applied is the **Anchor**.

  See the [Vessel Schema Definition documentation](./docs/vessel_schema.md)

### Anchor

  A variable contained in a vessel. The variable is only applied if both the **Filter scope** (`scope` field) and the **Filter defintion** (`filter` field) match

  A *non-existiong (or empty list as) scope* refers to the current object
  An *non-existing filter* always matches

## Anchor Schema

An anchored Type is defined by importing `AbsintheVessel.Schema`
and using `arg` definitions in the appropiate locations

```elixir
defmodule CashAppWeb.AbsintheSchema do
  use Absinthe.Schema
  def plugins do
    [AbsintheVessel.Data] ++ Absinthe.Plugin.defaults()
  end
  object :organisation do
    field :short_name, type: :string
    field :people, type: :list_of(person)
  end
  object :person do
    field :first_name, :string
    field :holds_memberships, type: list_of(:organisation) do
      resolve fn
        %{first_name: "Jim"},  %{first: first}, info->
          assert first == 3
        %{first_name: other_na,e}, %{first: first}, info ->
          assert first == 2
      end
    end
  end
  query do
    field :organisations, type: list_of(:organization) do
      arg :first, :vessel_int
      resolve fn _, %{first: first }, _ ->
        assert first == 2
      end
    end
  end
end
```

### Filter scope

### Example

Assuming that all lists in the following example accept the variables
`first` and `offset` for a rudimentary pagination implementation.

```graphql
query organizations ($first: int) {
  organisations(first: $first){ # organisation list
    short_name
    people($first:2){ # Person list
      first_name
      holds_memberships(first: 2) { # this is an organisation list
        short_name
      }
    }
  }
}
```

and this response:

```json
"organisations":[
  {
    "short_name": "IBM",
    "people": [
      {
        "first_name": "Jane",
        "holds_memberships":[{"short_name":"IBM"},{"short_name":"HP"}]
      },{
        "first_name": "Jim",
        "holds_memberships": [{"short_name":"HP"}, {"short_name":"MS"}]
      }
    ]
  },
]
```

Both `Jim` and `Jane` are also members in the `GREEN` organisation, which is omitted from the result
because of the `first` argument in the `holds_memberships(first: 2)` list

* How do you now specify that you want to recieve 3 instead of 2 responses for `Jim` (and `Jim` only) ?

by using a `filter` variable field in combination with a `scope` in an anchor.
The `$first: int` variable declaration must be changed to `$first: anchor_int`

```json
{
  "first":[
    {"value": 2},
    {
      "value": 3,
      "scope": ["organisations", "people"],
      "filter": {
          "key": "first_name",
          "val": "Jim"
      }
    }
  ]
}
```

* How do you know that `Jim` will actually be in the result?

**You don't!** nevertheles you can specify the value for `first` scoped by a `filter`, which will only apply
to the subset of people

## Installation

If [available in Hex](https://hex.pm/docs/publish), the package can be installed
by adding `absinthe_vessel_vars` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [
    {:absinthe_vessel_vars, "~> 0.1.0"}
  ]
end
```

Documentation can be generated with [ExDoc](https://github.com/elixir-lang/ex_doc)
and published on [HexDocs](https://hexdocs.pm). Once published, the docs can
be found at [https://hexdocs.pm/absinthe_vessel_vars](https://hexdocs.pm/absinthe_vessel_vars).

