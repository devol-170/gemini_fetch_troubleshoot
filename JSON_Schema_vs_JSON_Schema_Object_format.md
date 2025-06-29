# .parametersJsonSchema example
```json
{
  "type": "object",
  "properties": {
    "name": { "type": "string" },
    "age": { "type": "integer" }
  },
  "additionalProperties": false,
  "required": ["name", "age"],
  "propertyOrdering": ["name", "age"]
}
```

# .parameters example
```json
{
  "name": {
    "name": "name",
    "in": "query",
    "required": true
  },
  "age": {
    "name": "age",
    "in": "query",
    "required": true
  }
}
```