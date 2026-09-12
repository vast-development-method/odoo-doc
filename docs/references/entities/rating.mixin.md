# Rating Mixin (`rating.mixin`)

**Transport name:** `rating.mixin`  
**Storage name:** `rating_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `rating`

Description: Rating Mixin

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `rating_last_value` | Rating Last Value | float |  | computed by rule `_compute_rating_last_value` and stored; visible only to groups `base.group_user`; aggregated with avg |
| `rating_last_feedback` | Rating Last Feedback | multi line text |  | related through path `rating_ids.feedback`; visible only to groups `base.group_user` |
| `rating_last_image` | Rating Last Image | binary |  | related through path `rating_ids.rating_image`; visible only to groups `base.group_user` |
| `rating_count` | Rating count | integer |  | computed by rule `_compute_rating_stats` (not stored) |
| `rating_avg` | Average Rating | float |  | computed by rule `_compute_rating_stats` (not stored); searchable through a search rule; visible only to groups `base.group_user` |
| `rating_avg_text` | Rating Avg Text | selection |  | computed by rule `_compute_rating_avg_text` (not stored); visible only to groups `base.group_user` |
| `rating_percentage_satisfaction` | Rating Satisfaction | float |  | computed by rule `_compute_rating_satisfaction` (not stored) |
| `rating_last_text` | Rating Text | selection |  | related through path `rating_ids.rating_text`; visible only to groups `base.group_user` |

## Operations (13)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_rating_last_value` | computation | self | `rating` | depends: `rating_ids`, `rating_ids.rating`, `rating_ids.consumed` |  |
| `_compute_rating_stats` | computation | self | `rating` | depends: `rating_ids.res_id`, `rating_ids.rating` | Compute avg and count in one query, as thoses fields will be used together most of the time. |
| `_search_rating_avg` | search rule | self, operator, value | `rating` |  |  |
| `_compute_rating_avg_text` | computation | self | `rating` | depends: `rating_avg` |  |
| `_compute_rating_satisfaction` | computation | self | `rating` | depends: `rating_ids.res_id`, `rating_ids.rating` | Compute the rating satisfaction percentage, this is done separately from rating_count and rating_avg since the query is different, to avoid computing if it is not necessary |
| `write` | lifecycle override | self, vals | `rating` |  | If the rated ressource name is modified, we should update the rating res_name too. If the rated ressource parent is changed we should update the parent_res_id too |
| `_rating_get_parent_field_name` | internal rule | self | `rating` |  | Return the parent relation field name. Should return a Many2One |
| `_rating_domain` | internal rule | self | `rating` |  | Returns a normalized domain on rating.rating to select the records to include in count, avg, ... computation of current model. |
| `_rating_get_repartition` | internal rule | self, add_stats, domain | `rating` |  | get the repatition of rating grade for the given res_ids. :param add_stats : flag to add stat to the result :type add_stats : boolean :param domain : optional extra domain of the rating to include/exclude in repartition :return dictionnary     if not add_stats, the dict is like         - key is the rating value (integer)         - value is the number of object (res_model, res_id) having the value     otherwise, key is the value of the information (string) : either stat name (avg, total, ...) or 'repartition'     containing the same dict if add_stats was False. |
| `rating_get_grades` | operation | self, domain | `rating` |  | Get the repartitions of rating grade for the given res_ids. :param domain: Optional domain of the rating to include/exclude     in the grades computation. :returns: A dictionary where the key is the rating and the value     is the count of unique `(res_model, res_id)` pairs whose     grades are associated with that rating.      The rates are:      * `"great"`, graded between 70 and 100     * `"okay"`, graded between 31 and 69     * `"bad"`, graded between 0 and 30 :rtype: dict[typing.Literal["great", "okay", "bad"], int] |
| `rating_get_stats` | operation | self, domain | `rating` |  | Get the statistics of the rating repartitions  :param domain : optional domain of the rating to include/exclude in statistic computation :returns: A dictionnary where:      - key is the name of the information (stat name)     - value is statistic value : 'percent' contains the repartition in percentage, 'avg' is the average rate       and 'total' is the number of rating |
| `_rating_get_stats_per_record` | internal rule | self, domain | `rating` |  | Computes rating statistics for each record individually.  :param domain: Optional domain to apply on the ratings. :return: A dictionary mapping each record ID to its statistics dictionary. :rtype: dict |
| `_allow_publish_rating_stats` | internal rule | self | `rating` | model | Override to allow the rating stats to be demonstrated. |

Machine-readable definition: `../../../schemas/data/entities/rating.mixin.json`.
