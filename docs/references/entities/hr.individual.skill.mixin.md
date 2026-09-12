# Skill level (`hr.individual.skill.mixin`)

**Transport name:** `hr.individual.skill.mixin`  
**Storage name:** `hr_individual_skill_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `hr_skills`

Description: Skill level

## Identity and behavior

- Default ordering: `skill_type_id, skill_level_id`
- Display name field: `skill_id`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `skill_id` | Skill | many to one | `hr.skill` | required; computed by rule `_compute_skill_id` and stored; on delete of the target: cascade; restricted by domain `[('skill_type_id', '=', skill_type_id)]` |
| `skill_level_id` | Skill Level | many to one | `hr.skill.level` | required; computed by rule `_compute_skill_level_id` and stored; on delete of the target: cascade; restricted by domain `[('skill_type_id', '=', skill_type_id)]` |
| `skill_type_id` | Skill Type | many to one | `hr.skill.type` | required; default computed dynamically (_default_skill_type_id); on delete of the target: cascade |
| `level_progress` | Level Progress | integer |  | related through path `skill_level_id.level_progress` |
| `color` | Color | integer |  | related through path `skill_type_id.color` |
| `valid_from` | Validity Start | date |  | default computed dynamically (fields.Date.today()) |
| `valid_to` | Validity Stop | date |  |  |
| `levels_count` | Levels Count | integer |  | related through path `skill_type_id.levels_count` |
| `certification_skill_type_count` | Certification Skill Type Count | integer |  | computed by rule `_compute_certification_skill_type_count` (not stored) |
| `is_certification` | Is Certification | boolean |  | related through path `skill_type_id.is_certification` |
| `display_warning_message` | Display Warning Message | boolean |  |  |

## Operations (19)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_linked_field_name` | internal rule | self | `hr_skills` |  |  |
| `_get_passive_fields` | preparation rule | self | `hr_skills` |  | Return additional passive fields to be included during (versioned)skill creation.  Passive fields are preserved/included when new skill versions are created due to changes in any of the core/active fields (linked_field, skill_id, skill_level_id, skill_type_id),but modifying them DOES NOT trigger new (versioned)skill creation.  Core/Active fields (linked_field, skill_id, skill_level_id, skill_type_id) are automatically preserved and should NOT be included here.  :return: List of field names to copy to new skills :rtype: list[str] |
| `_can_edit_certification_validity_period` | internal rule | self | `hr_skills` |  |  |
| `_default_skill_type_id` | preparation rule | self | `hr_skills` |  |  |
| `_check_not_overlapping_regular_skill` | validation | self | `hr_skills` | constrains: | The following is the core functionality and difference for the two models Skills:     1. There can only be one active skill for each skill_id, f.ex only one level of English allowed.     2. Skills should not be deleted, unless they were created within the last 24 hours. Skills should instead         be archived to preserve the history of the skills linked to that particular record.     3. Skills should not be written to, instead the previous skill should be archived and a new skill with         the new values should be created. This is again to preserve the history of skills on the record. Cer |
| `_get_overlapping_individual_skill` | preparation rule | self, vals_list | `hr_skills` |  |  |
| `_check_date` | validation | self | `hr_skills` | constrains: `valid_from`, `valid_to` |  |
| `_check_skill_type` | validation | self | `hr_skills` | constrains: `skill_id`, `skill_type_id` |  |
| `_check_skill_level` | validation | self | `hr_skills` | constrains: `skill_type_id`, `skill_level_id` |  |
| `_compute_certification_skill_type_count` | computation | self | `hr_skills` |  |  |
| `_onchange_is_certification` | on change | self | `hr_skills` | onchange: `is_certification` |  |
| `_compute_skill_id` | computation | self | `hr_skills` | depends: `skill_type_id` |  |
| `_compute_skill_level_id` | computation | self | `hr_skills` | depends: `skill_id` |  |
| `_compute_display_name` | computation | self | `hr_skills` | depends: `skill_id`, `skill_level_id` |  |
| `_onchange_valid_date` | on change | self | `hr_skills` | onchange: `valid_to`, `valid_from` |  |
| `_expire_individual_skills` | internal rule | self | `hr_skills` |  | This function archive all individual skill in self. If the individual skill is not expired (valid_to < today) then valid_to will be set to yesterday if it's possible (not break a constraint) Else the individual skill is delete  Example: An individual already have the skill English A2 (added one month ago) and we want to delete it output: [[1, id('English A2'), {'valid_to': yesterday}]] @return {List[COMMANDS]} List of WRITE, UNLINK commands |
| `_create_individual_skills` | internal rule | self, vals_list | `hr_skills` |  | This function transform CREATE commands into CREATE, WRITE and UNLINK commands in order to keep the logs and to follow the constraints  Example: An individual already have the skill English A2 (added one month ago) and we want to add the skill English B1 This method will transform: {linked_field: id, skill_id: id('English'), skill_level_id: id('B1') skill_type_id: id('Languages')} into [     [1, id('English A2'), {'valid_to': yesterday}],     [0, 0, {         linked_field: id,         skill_id: id('English'),         skill_level_id: id('B1'),         skill_type_id: id('Languages')}     ] ] @pa |
| `_write_individual_skills` | internal rule | self, commands | `hr_skills` |  | Transform a list of write commands into a list of create, write and unlink commands according to the logic of how skills should behave. The relevant logic is as follows:  * If "skill_type_id", "skill_id", "skill_level_id", self._linked_field_name() are not in vals, this method will     behave like any standard write method. * Otherwise, the current record is archived, by changing valid_to to yesterday, and a new one is created with values from vals and self, with vals taking priority.   :param commands: list of WRITE commands :return: List of CREATE, WRITE, UNLINK commands |
| `_get_transformed_commands` | preparation rule | self, commands, individuals | `hr_skills` |  | Transform a list of ORM commands to fit with the business constraints and preserve the logic of how skills and certifications should behave. The key behaviors are as follows:  Skills: 1. Only one active skill per `skill_id` is allowed (e.g., one "English" skill per linked_field record).  Certifications (`is_certification=True`): 1. Multiple certifications with the same `skill_id` and `level_id` are allowed if their date ranges differ (e.g.,     "certified (2024-01-01 → 2024-12-31)" and "certified (2024-06-01 → 2025-05-31)" can coexist.)  Shared Rules: - Updates always create new reco |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_not_overlapping_regular_skill` | ValidationError | error_msg | `hr_skills` |
| `_check_date` | ValidationError | The following skills have their valid stop date prior to their valid start date: | `hr_skills` |
| `_check_skill_type` | ValidationError | The skill %(name)s and skill type %(type)s don't match | `hr_skills` |
| `_check_skill_level` | ValidationError | The skill level %(level)s is not valid for skill type: %(type)s | `hr_skills` |

Machine-readable definition: `../../../schemas/data/entities/hr.individual.skill.mixin.json`.
