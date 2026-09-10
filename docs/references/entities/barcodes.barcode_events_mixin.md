# Barcode Event Mixin (`barcodes.barcode_events_mixin`)

**Transport name:** `barcodes.barcode_events_mixin`  
**Storage name:** `barcodes_barcode_events_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `barcodes`

Description: Barcode Event Mixin

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `_barcode_scanned` | Barcode Scanned | single line text |  | Help: Value of the last barcode scanned. |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_on_barcode_scanned` | on change | self | `barcodes` | onchange: `_barcode_scanned` |  |
| `on_barcode_scanned` | operation | self, barcode | `barcodes` |  |  |

Machine-readable definition: `../../../schemas/data/entities/barcodes.barcode_events_mixin.json`.
