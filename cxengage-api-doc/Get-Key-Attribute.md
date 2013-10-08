## Key Attribute Resources

```
  GET tenants/:iid/key-attribute
```

## Description

Retrieves key attribute for the given tenant.


### Parameters

- **iid** _(required)_ — Selected tenant

### Errors

All known errors will be returned in a JSON map with key "error".

- **404 Not Found** - The requested node or instance does not exist or has been deleted.

### Example

**Request**

```
  GET /tenants/tenant-one/key-attribute
```

```json
{
    "key": "id"
}
```
 


