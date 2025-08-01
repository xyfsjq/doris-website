---
{
    "title": "JSONB_EXTRACT",
    "language": "en"
}
---

## jsonb_extract

jsonb_extract

### description

#### Syntax

```sql
JSONB jsonb_extract(JSONB j, VARCHAR json_path)
BOOLEAN jsonb_extract_isnull(JSONB j, VARCHAR json_path)
BOOLEAN jsonb_extract_bool(JSONB j, VARCHAR json_path)
INT jsonb_extract_int(JSONB j, VARCHAR json_path)
BIGINT jsonb_extract_bigint(JSONB j, VARCHAR json_path)
DOUBLE jsonb_extract_double(JSONB j, VARCHAR json_path)
STRING jsonb_extract_string(JSONB j, VARCHAR json_path)
```

jsonb_extract functions extract field specified by json_path from JSONB. A series of functions are provided for different datatype.
- jsonb_extract extract and return JSONB datatype
- jsonb_extract_isnull check if the field is json null and return BOOLEAN datatype
- jsonb_extract_bool extract and return BOOLEAN datatype
- jsonb_extract_int extract and return INT datatype
- jsonb_extract_bigint extract and return BIGINT datatype
- jsonb_extract_double extract and return DOUBLE datatype
- jsonb_extract_STRING extract and return STRING datatype

Exception handling is as follows:
- if the field specified by json_path does not exist, return NULL
- if datatype of the field specified by json_path is not the same with type of jsonb_extract_t, return t if it can be cast to t else NULL


## jsonb_exists_path and jsonb_type
### description

#### Syntax

```sql
BOOLEAN jsonb_exists_path(JSONB j, VARCHAR json_path)
STRING jsonb_type(JSONB j, VARCHAR json_path)
```

There are two extra functions to check field existence and type
- jsonb_exists_path check the existence of the field specified by json_path, return TRUE or FALS
- jsonb_exists_path get the type as follows of the field specified by json_path, return NULL if it does not exist
  - object
  - array
  - null
  - bool
  - int
  - bigint
  - double
  - string

### example

refer to [jsonb tutorial](../../sql-reference/Data-Types/JSONB.md) for more.


### keywords
JSONB, JSON, jsonb_extract, jsonb_extract_isnull, jsonb_extract_bool, jsonb_extract_int, jsonb_extract_bigint, jsonb_extract_double, jsonb_extract_string, jsonb_exists_path, jsonb_type
