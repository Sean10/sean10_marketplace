# Generate Product Code

Generate a complete product management module for marketplace applications.

## Usage

```
/generate-product [--type=api|component|schema] [--name=ProductName]
```

## Options

- `--type`: Type of code to generate (api, component, schema)
- `--name`: Name of the entity (default: Product)

## Examples

```
/generate-product --type=api
/generate-product --type=component --name=Book
/generate-product --type=schema
```

## Output

Generates:
- TypeScript interfaces
- API handlers/endpoints
- React components (if component type)
- Database schema (if schema type)
- Unit tests
