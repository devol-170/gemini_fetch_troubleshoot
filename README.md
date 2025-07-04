# Issue
```
ℹ Configured MCP servers:
 
  🟢 mcpFetch - Ready (1 tools)
    - fetch



> use mcpFetch MCP Server to get https://modelcontextprotocol.io/introduction

✕ [API Error: {"error":{"message":"{\n  \"error\": {\n    \"code\": 400,\n    \"message\": \"* GenerateContentRequest.tools[0].function_declarations[11].parameters.properties[url].format: only 'enum' and 'date-time' are supported 
  for STRING type\\n\",\n    \"status\": \"INVALID_ARGUMENT\"\n  }\n}\n","code":400,"status":"Bad Request"}}]
```

# Troubleshooting

## Step 1: Original Value of properties.url.format

Copied the [Fetch Tool model code](https://github.com/modelcontextprotocol/servers/blob/6b0c30d1a807121fd1ba7b7f906b1aea8486fb35/src/fetch/src/mcp_server_fetch/server.py#L21) and  extracted only the pydantic Fetch code in [fetch_model.py](./fetch_model.py)

```
python fetch_model.py | jq .properties.url
{
  "description": "URL to fetch",
  "format": "uri",
  "minLength": 1,
  "title": "Url",
  "type": "string"
}

```

==> value is "uri"

## Step 2: Looking into function_declarations.parameters

from https://cloud.google.com/vertex-ai/generative-ai/docs/reference/rest/v1beta1/FunctionDeclaration#Schema

```
parameters
object (Schema)
Optional. Describes the parameters to this function in JSON Schema Object format. Reflects the Open API 3.03 Parameter Object. string Key: the name of the parameter. Parameter names are case sensitive. Schema value: the Schema defining the type used for the parameter. For function with no parameters, this can be left unset. Parameter names must start with a letter or an underscore and must only contain chars a-z, A-Z, 0-9, or underscores with a maximum length of 64. Example with 1 required and 1 optional parameter: type: OBJECT properties: param1: type: STRING param2: type: INTEGER required: - param1
```

[Open API 4.7.12.5 Parameter Object](https://spec.openapis.org/oas/v3.0.3.html#parameter-object
)  
example
```
{
  "name": "token",
  "in": "header",
  "description": "token to be passed as a header",
  "required": true,
  "schema": {
    "type": "array",
    "items": {
      "type": "integer",
      "format": "int64"
    }
  },
  "style": "simple"
}
```

## Conclusion
Somehow the allowed format for the "url" property changes from "uri" to "date-time" or "emum".

From the [fetch json schema](full_fetch_schema.json) we know that the "url" property is required and should be of Open API format "uri".

> dumped with: python fetch_model.py > full_fetch_schema.json

Both the Fetch Json Schema generated by pydantic and the Vertex AI Parameters should describe the input parameters of the fetch tool, so I assume that the error lies in the transformation of the .properties to the function_declarations.parameters Open API Parameter Objects.

I do not understand why parameters is used at all, when the json schema for parametersJsonSchema is already provided by the MCP server
