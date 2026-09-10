# Sparse fields Test (`sparse_fields.test`)

**Transport name:** `sparse_fields.test`  
**Storage name:** `sparse_fields_test`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base_sparse_field`

Description: Sparse fields Test

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `boolean` | Boolean | boolean |  |  |
| `integer` | Integer | integer |  |  |
| `float` | Float | float |  |  |
| `char` | Char | single line text |  |  |
| `selection` | Selection | selection |  |  |
| `partner` | Partner | many to one | `res.partner` |  |

## Selection values

### `selection` (Selection)

| Value | Label |
|---|---|
| `one` | One |
| `two` | Two |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | no | `base_sparse_field` |

Machine-readable definition: `../../../schemas/data/entities/sparse_fields.test.json`.
