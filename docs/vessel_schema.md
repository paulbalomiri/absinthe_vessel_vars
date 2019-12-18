# Schema for Vessel variable

This is a simple schema, where complexities pertinent to the  `input filter` are not adressed, ny implementing string equality only

```graphql

input filter{
  key: String
  value: String
}

input vesselInt{
  value: Int
  scope: [String!]
  filter: [filter!]
}

type person{
  first_name: String
  holds_memberships: [organisation]
}

type organisation{
  short_name: String
  members: [person]
}

type query{
  organisations: [organisation]
}
schema{
  query: query
}
```
