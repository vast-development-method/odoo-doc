# Rating Parent Mixin (`rating.parent.mixin`)

**Transport name:** `rating.parent.mixin`  
**Storage name:** `rating_parent_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `rating`

Description: Rating Parent Mixin

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `rating_ids` | Ratings | one to many | `rating.rating` | visible only to groups `base.group_user`; restricted by domain `lambda self: [('parent_res_model', '=', self._name)]`; inverse field `parent_res_id` |
| `rating_percentage_satisfaction` | Rating Satisfaction | integer |  | computed by rule `_compute_rating_percentage_satisfaction` (not stored); Help: Percentage of happy ratings |
| `rating_count` | # Ratings | integer |  | computed by rule `_compute_rating_percentage_satisfaction` (not stored) |
| `rating_avg` | Average Rating | float |  | computed by rule `_compute_rating_percentage_satisfaction` (not stored); searchable through a search rule; visible only to groups `base.group_user` |
| `rating_avg_percentage` | Average Rating (%) | float |  | computed by rule `_compute_rating_percentage_satisfaction` (not stored); visible only to groups `base.group_user` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_rating_percentage_satisfaction` | computation | self | `rating` | depends: `rating_ids.rating`, `rating_ids.consumed` |  |
| `_search_rating_avg` | search rule | self, operator, value | `rating` |  |  |

Machine-readable definition: `../../../schemas/data/entities/rating.parent.mixin.json`.
