# Html Editor Converter Test (`html_editor.converter.test`)

**Transport name:** `html_editor.converter.test`  
**Storage name:** `html_editor_converter_test`  
**Kind:** persistent entity (one table)  
**Defined by package:** `html_editor`

Description: Html Editor Converter Test

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `char` | Char | single line text |  |  |
| `integer` | Integer | integer |  |  |
| `float` | Float | float |  |  |
| `numeric` | Numeric | float |  | precision `[16, 2]` |
| `many2one` | Many2one | many to one | `html_editor.converter.test.sub` |  |
| `binary` | Binary | binary |  |  |
| `date` | Date | date |  |  |
| `datetime` | Datetime | date and time |  |  |
| `selection_str` | Lorsqu'un pancake prend l'avion à destination de Toronto et qu'il fait une escale technique à St Claude, on dit: | selection |  |  |
| `html` | Hypertext markup language | rich text |  |  |
| `text` | Text | multi line text |  |  |

## Selection values

### `selection_str` (Lorsqu'un pancake prend l'avion à destination de Toronto et qu'il fait une escale technique à St Claude, on dit:)

| Value | Label |
|---|---|
| `A` | Qu'il n'est pas arrivé à Toronto |
| `B` | Qu'il était supposé arriver à Toronto |
| `C` | Qu'est-ce qu'il fout ce maudit pancake, tabernacle ? |
| `D` | La réponse D |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `html_editor` |

Machine-readable definition: `../../../schemas/data/entities/html_editor.converter.test.json`.
