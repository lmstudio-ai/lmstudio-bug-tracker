# lmstudio-bug-tracker
Bug tracking for the LM Studio desktop application

## Fixes in Updated Beta Release

* The updated beta release fixes the issue with structured output in LM Studio 0.3.5.
* The fix involved modifying the JSON schema to work correctly in both the UI and the server.
* The updated beta release also includes a fix for pasting text from Microsoft Word giving an error about pasting an image.
* The issue with the Pydantic use case and the `$ref/$def` format was identified and addressed in the updated beta release.
* The user provided a working code snippet to resolve references in the JSON schema, which was confirmed to work with the updated beta release.

## Automate Schema Generation

Create a utility function that generates the JSON schema from Pydantic models, including resolving references and setting additional properties to false.

## Integrate Schema Validation Library

Integrate a schema validation library that can handle the `$ref/$def` format and other complex schema structures.

## User-Provided Working Code Snippet

```python
def resolve_refs(schema: dict, definitions: dict) -> dict:
    """Recursively resolve all $ref references in a JSON schema."""
    if isinstance(schema, dict):
        if '$ref' in schema:
            ref_path = schema['$ref']
            if ref_path.startswith('#/$defs/'):
                ref_name = ref_path.split('/')[-1]
                return resolve_refs(definitions[ref_name], definitions)
        
        return {k: resolve_refs(v, definitions) for k, v in schema.items() if k != '$defs'}
    elif isinstance(schema, list):
        return [resolve_refs(item, definitions) for item in schema]
    else:
        return schema

def create_json_schema(model: Type[BaseModel]) -> Dict:
    schema = model.model_json_schema()
    
    # Store the definitions
    definitions = schema.get('$defs', {})
    
    # Resolve all references
    resolved_schema = resolve_refs(schema, definitions)
    
    return {
        "type": "json_schema",
        "json_schema": {
            "name": "test_schema",
            "strict": True,
            "schema": resolved_schema,
        },
    }
```
