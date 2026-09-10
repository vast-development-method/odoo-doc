# Public Employee (`hr.employee.public`)

**Transport name:** `hr.employee.public`  
**Storage name:** `hr_employee_public`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr`  
**Extended by packages:** `hr_attendance`, `hr_expense`, `hr_expense`, `hr_fleet`, `hr_gamification`, `hr_holidays`, `hr_homeworking`, `hr_maintenance`, `hr_org_chart`, `hr_org_chart`, `hr_presence`, `hr_skills`, `hr_skills_slides`, `hr_timesheet`

Description: Public Employee

## Identity and behavior

- Default ordering: `name, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (99)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `create_date` | Create Date | date and time |  | read only |
| `name` | Name | single line text |  | read only |
| `active` | Active | boolean |  | read only |
| `department_id` | Department | many to one | `hr.department` | read only |
| `member_of_department` | Member Of Department | boolean |  | computed by rule `_compute_member_of_department` (not stored); searchable through a search rule |
| `job_id` | Job | many to one | `hr.job` | read only |
| `job_title` | Job Title | single line text |  | related through path `employee_id.job_title` |
| `company_id` | Company | many to one | `res.company` | read only |
| `address_id` | Address | many to one | `res.partner` | read only |
| `mobile_phone` | Mobile Phone | single line text |  | read only |
| `work_phone` | Work Phone | single line text |  | read only |
| `work_email` | Work Email | single line text |  | read only |
| `share` | Share | boolean |  | related through path `employee_id.share` |
| `phone` | Phone | single line text |  | related through path `employee_id.phone` |
| `im_status` | Im Status | single line text |  | related through path `employee_id.im_status` |
| `email` | Email | single line text |  | related through path `employee_id.email` |
| `work_contact_id` | Work Contact | many to one | `res.partner` | read only |
| `work_location_id` | Work Location | many to one | `hr.work.location` | read only |
| `work_location_name` | Work Location Name | single line text |  | related through path `employee_id.work_location_name` |
| `work_location_type` | Work Location Type | selection |  | related through path `employee_id.work_location_type` |
| `user_id` | User | many to one | `res.users` | read only |
| `resource_id` | Resource | many to one | `resource.resource` | read only |
| `tz` | Tz | selection |  | related through path `resource_id.tz` |
| `color` | Color | integer |  | read only |
| `hr_presence_state` | Human resources Presence State | selection |  | computed by rule `_compute_presence_state` (not stored); default `out_of_working_hour` |
| `hr_icon_display` | Human resources Icon Display | selection |  | computed by rule `_compute_presence_icon` (not stored); values provided by rule `_get_selection_hr_icon_display` |
| `show_hr_icon_display` | Show Human resources Icon Display | boolean |  | computed by rule `_compute_presence_icon` (not stored) |
| `last_activity` | Last Activity | date |  | computed by rule `_compute_last_activity` (not stored) |
| `last_activity_time` | Last Activity Time | single line text |  | computed by rule `_compute_last_activity` (not stored) |
| `resource_calendar_id` | Resource Calendar | many to one | `resource.calendar` | read only |
| `country_code` | Country Code | single line text |  | computed by rule `_compute_country_code` (not stored) |
| `is_manager` | Is Manager | boolean |  | computed by rule `_compute_is_manager` (not stored) |
| `is_user` | Is User | boolean |  | computed by rule `_compute_is_user` (not stored) |
| `employee_id` | Employee | many to one | `hr.employee` | read only |
| `child_ids` | Direct subordinates | one to many | `hr.employee.public` | read only; inverse field `parent_id` |
| `image_1920` | Image | image |  | related through path `employee_id.image_1920` |
| `image_1024` | Image 1024 | image |  | related through path `employee_id.image_1024` |
| `image_512` | Image 512 | image |  | related through path `employee_id.image_512` |
| `image_256` | Image 256 | image |  | related through path `employee_id.image_256` |
| `image_128` | Image 128 | image |  | related through path `employee_id.image_128` |
| `avatar_1920` | Avatar | image |  | related through path `employee_id.avatar_1920` |
| `avatar_1024` | Avatar 1024 | image |  | related through path `employee_id.avatar_1024` |
| `avatar_512` | Avatar 512 | image |  | related through path `employee_id.avatar_512` |
| `avatar_256` | Avatar 256 | image |  | related through path `employee_id.avatar_256` |
| `avatar_128` | Avatar 128 | image |  | related through path `employee_id.avatar_128` |
| `parent_id` | Manager | many to one | `hr.employee.public` | read only |
| `coach_id` | Coach | many to one | `hr.employee.public` | read only |
| `user_partner_id` | User's partner | many to one |  | related through path `user_id.partner_id` |
| `birthday_public_display_string` | Public Date of Birth | single line text |  | related through path `employee_id.birthday_public_display_string` |
| `newly_hired` | Newly Hired | boolean |  | computed by rule `_compute_newly_hired` (not stored); searchable through a search rule |
| `attendance_state` | Attendance State | selection |  | read only; related through path `employee_id.attendance_state`; visible only to groups `hr_attendance.group_hr_attendance_officer` |
| `hours_today` | Hours Today | float |  | read only; related through path `employee_id.hours_today`; visible only to groups `hr_attendance.group_hr_attendance_officer` |
| `hours_last_month` | Hours Last Month | float |  | related through path `employee_id.hours_last_month` |
| `hours_last_month_overtime` | Hours Last Month Overtime | float |  | related through path `employee_id.hours_last_month_overtime` |
| `last_attendance_id` | Last Attendance | many to one |  | read only; related through path `employee_id.last_attendance_id`; visible only to groups `hr_attendance.group_hr_attendance_officer` |
| `total_overtime` | Total Overtime | float |  | read only; related through path `employee_id.total_overtime` |
| `attendance_manager_id` | Attendance Manager | many to one |  | related through path `employee_id.attendance_manager_id`; visible only to groups `hr_attendance.group_hr_attendance_officer` |
| `last_check_in` | Last Check In | date and time |  | related through path `employee_id.last_check_in`; visible only to groups `hr_attendance.group_hr_attendance_officer` |
| `last_check_out` | Last Check Out | date and time |  | related through path `employee_id.last_check_out`; visible only to groups `hr_attendance.group_hr_attendance_officer` |
| `display_extra_hours` | Display Extra Hours | boolean |  | related through path `company_id.hr_attendance_display_overtime` |
| `display_attendances` | Display Attendances | boolean |  | related through path `employee_id.display_attendances` |
| `filter_for_expense` | Filter For Expense | boolean |  | searchable through a search rule; visible only to groups `hr.group_hr_user` |
| `expense_manager_id` | Expense Manager | many to one | `res.users` | read only |
| `mobility_card` | Mobility Card | single line text |  | read only |
| `badge_ids` | Badge | one to many | `gamification.badge.user` | read only; computed by rule `_compute_badge_ids` (not stored) |
| `has_badges` | Has Badges | boolean |  | computed by rule `_compute_has_badges` (not stored) |
| `leave_manager_id` | Time Off Approver | many to one | `res.users` | computed by rule `_compute_leave_manager` and stored; restricted by domain `[('share', '=', False), ('company_ids', 'in', company_id)]`; Help: Select the user responsible for approving "Time Off" of this employee. If empty, the approval is done by an Administrator or Approver (determined in settings/users). |
| `leave_date_to` | To Date | date |  | computed by rule `_compute_leave_status` (not stored) |
| `show_leaves` | Able to see Remaining Time Off | boolean |  | computed by rule `_compute_show_leaves` (not stored) |
| `is_absent` | Absent Today | boolean |  | computed by rule `_compute_leave_status` (not stored); searchable through a search rule |
| `allocation_display` | Allocation Display | single line text |  | computed by rule `_compute_allocation_display` (not stored) |
| `allocation_remaining_display` | Allocation Remaining Display | single line text |  | related through path `employee_id.allocation_remaining_display` |
| `monday_location_id` | Monday | many to one | `hr.work.location` |  |
| `tuesday_location_id` | Tuesday | many to one | `hr.work.location` |  |
| `wednesday_location_id` | Wednesday | many to one | `hr.work.location` |  |
| `thursday_location_id` | Thursday | many to one | `hr.work.location` |  |
| `friday_location_id` | Friday | many to one | `hr.work.location` |  |
| `saturday_location_id` | Saturday | many to one | `hr.work.location` |  |
| `sunday_location_id` | Sunday | many to one | `hr.work.location` |  |
| `today_location_name` | Today Location Name | single line text |  |  |
| `equipment_count` | Equipment Count | integer |  | related through path `employee_id.equipment_count` |
| `subordinate_ids` | Subordinate | one to many |  | related through path `employee_id.subordinate_ids` |
| `is_subordinate` | Is Subordinate | boolean |  | related through path `employee_id.is_subordinate` |
| `child_all_count` | Child All Count | integer |  | computed by rule `_compute_child_all_count` (not stored) |
| `department_color` | Department Color | integer |  | computed by rule `_compute_department_color` (not stored) |
| `child_count` | Child Count | integer |  | computed by rule `_compute_child_count` (not stored) |
| `email_sent` | Email Sent | boolean |  | default  |
| `ip_connected` | Internet protocol Connected | boolean |  | default  |
| `manually_set_present` | Manually Set Present | boolean |  | default  |
| `manually_set_presence` | Manually Set Presence | boolean |  | default  |
| `hr_presence_state_display` | Human resources Presence State Display | selection |  | default `out_of_working_hour` |
| `resume_line_ids` | Resume lines | one to many | `hr.resume.line` | inverse field `employee_id` |
| `employee_skill_ids` | Skills | one to many | `hr.employee.skill` | restricted by domain `[["skill_type_id.active", "=", true]]`; inverse field `employee_id` |
| `current_employee_skill_ids` | Current Employee Skill | one to many | `hr.employee.skill` | related through path `employee_id.current_employee_skill_ids` |
| `certification_ids` | Certification | one to many | `hr.employee.skill` | related through path `employee_id.certification_ids` |
| `display_certification_page` | Display Certification Page | boolean |  | related through path `employee_id.display_certification_page` |
| `has_subscribed_courses` | Has Subscribed Courses | boolean |  | related through path `employee_id.has_subscribed_courses` |
| `courses_completion_text` | Courses Completion Text | single line text |  | related through path `employee_id.courses_completion_text` |
| `has_timesheet` | Has Timesheet | boolean |  | related through path `employee_id.has_timesheet` |

## Selection values

### `hr_presence_state` (Human resources Presence State)

| Value | Label |
|---|---|
| `present` | Present |
| `absent` | Absent |
| `archive` | Archived |
| `out_of_working_hour` | Off-Hours |

### `hr_presence_state_display` (Human resources Presence State Display)

| Value | Label |
|---|---|
| `out_of_working_hour` | Off-Hours |
| `present` | Present |
| `absent` | Absent |

## State fields

State machine fields of this entity: `hr_presence_state`, `attendance_state`. Transitions are specified in the domain documents.

## Operations (34)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_selection_hr_icon_display` | preparation rule | self | `hr` |  |  |
| `_compute_from_employee` | computation | self, field_names | `hr` |  |  |
| `_compute_last_activity` | computation | self | `hr` | depends: `user_id` |  |
| `_compute_country_code` | computation | self | `hr` |  |  |
| `_compute_is_manager` | computation | self | `hr` | depends_context: `uid`; depends: `parent_id` |  |
| `_compute_is_user` | computation | self | `hr` | depends_context: `uid` |  |
| `_compute_presence_state` | computation | self | `hr` |  |  |
| `_compute_presence_icon` | computation | self | `hr` |  |  |
| `_compute_member_of_department` | computation | self | `hr` |  |  |
| `_get_manager_only_fields` | preparation rule | self | `hr` |  |  |
| `_search_part_of_department` | search rule | self, operator, value | `hr` |  |  |
| `_get_valid_employee_for_user` | preparation rule | self | `hr` |  |  |
| `_compute_manager_only_fields` | computation | self | `hr` | depends_context: `uid` |  |
| `_compute_newly_hired` | computation | self | `hr` |  |  |
| `_search_newly_hired` | search rule | self, operator, value | `hr` |  |  |
| `_get_fields` | preparation rule | self | `hr` | model |  |
| `init` | lifecycle override | self | `hr` |  |  |
| `get_avatar_card_data` | operation | self, fields | `hr` |  |  |
| `action_open_last_month_attendances` | user action | self | `hr_attendance` |  |  |
| `_search_filter_for_expense` | search rule | self, operator, value | `hr_expense` |  |  |
| `_compute_has_badges` | computation | self | `hr_gamification` |  |  |
| `_compute_badge_ids` | computation | self | `hr_gamification` |  |  |
| `_compute_show_leaves` | computation | self | `hr_holidays` |  |  |
| `_compute_leave_manager` | computation | self | `hr_holidays` |  |  |
| `_compute_leave_status` | computation | self | `hr_holidays` |  |  |
| `_search_absent_employee` | search rule | self, operator, value | `hr_holidays` |  |  |
| `_compute_allocation_display` | computation | self | `hr_holidays` |  |  |
| `action_time_off_dashboard` | user action | self | `hr_holidays` |  |  |
| `action_open_time_off_calendar` | user action | self | `hr_holidays` |  | Open the time off calendar filtered on this employee. |
| `_compute_child_all_count` | computation | self | `hr_org_chart` |  |  |
| `_compute_department_color` | computation | self | `hr_org_chart` |  |  |
| `_compute_child_count` | computation | self | `hr_org_chart` |  |  |
| `action_open_courses` | user action | self | `hr_skills_slides` |  |  |
| `action_timesheet_from_employee` | user action | self | `hr_timesheet` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `hr` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Employee multi company rule | global (all users) | `['\|', '\|', '\|',             ('company_id', 'in', company_ids + [False]),             ('parent_id.user_id', '=', user.id),             ('id', '=', user.employee_id.parent_id.id),             ('user_id', '=', user.id)         ]` | True | True | True | True |

## Views (16)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr.hr_employee_public_view_search` | search |  | `name`, `company_id`, `department_id`, `parent_id`, `job_title`, `job_id` |  | `My Team`, `My Department`, `Newly Hired`, `Archived`, `Manager`, `Department`, `Job`, `Company` | `hr` |
| `hr.hr_employee_public_view_form` | form |  | `image_1920`, `show_hr_icon_display`, `name`, `work_email`, `work_phone`, `mobile_phone`, `hr_icon_display`, `company_id`, `department_id`, `job_title`, `job_id`, `parent_id`, `address_id`, `work_location_id` |  |  | `hr` |
| `hr.hr_employee_public_view_tree` | list |  | `name`, `work_phone`, `work_email`, `company_id`, `department_id`, `job_id`, `parent_id`, `coach_id` |  |  | `hr` |
| `hr.hr_employee_public_view_kanban` | kanban |  | `show_hr_icon_display`, `image_128`, `avatar_128`, `image_1024`, `hr_icon_display`, `name`, `job_title`, `job_id`, `work_email`, `work_phone`, `birthday_public_display_string`, `user_id` |  |  | `hr` |
| `hr_attendance.hr_employee_public_view_form` | xpath | `hr.hr_employee_public_view_form` | `hours_last_month`, `hours_last_month_overtime`, `hours_last_month_overtime`, `hours_last_month`, `hours_last_month_overtime`, `hours_last_month_overtime` | `action_open_last_month_attendances`, `action_open_last_month_attendances` |  | `hr_attendance` |
| `hr_gamification.hr_employee_public_view_form` | page | `hr.hr_employee_public_view_form` | `has_badges`, `badge_ids` | `Grant a Badge` |  | `hr_gamification` |
| `hr_holidays.hr_kanban_view_public_employees_kanban` | xpath | `hr.hr_employee_public_view_kanban` | `is_absent` |  |  | `hr_holidays` |
| `hr_holidays.hr_employee_public_form_view_inherit` | xpath | `hr.hr_employee_public_view_form` | `show_leaves` | `action_open_time_off_calendar` |  | `hr_holidays` |
| `hr_maintenance.hr_employee_public_view_form` | xpath | `hr.hr_employee_public_view_form` | `employee_id`, `equipment_count` | `%(maintenance.hr_equipment_action)d` |  | `hr_maintenance` |
| `hr_org_chart.hr_employee_public_hierarchy_view` | hierarchy |  | `name`, `job_id`, `department_color`, `hr_icon_display`, `department_id`, `image_1024`, `name`, `hr_icon_display`, `job_id` |  |  | `hr_org_chart` |
| `hr_org_chart.hr_employee_public_view_form_inherit_org_chart` | xpath | `hr.hr_employee_public_view_form` | `child_ids` |  |  | `hr_org_chart` |
| `hr_skills.hr_employee_public_view_search` | xpath | `hr.hr_employee_public_view_search` | `employee_skill_ids`, `resume_line_ids` |  |  | `hr_skills` |
| `hr_skills.hr_employee_public_view_form_inherit` | page | `hr.hr_employee_public_view_form` | `resume_line_ids`, `line_type_id`, `name`, `description`, `date_start`, `date_end`, `is_course`, `duration`, `external_url`, `employee_id`, `current_employee_skill_ids`, `skill_type_id`, `skill_id`, `skill_level_id`, `level_progress`, `valid_to` |  |  | `hr_skills` |
| `hr_skills_slides.hr_employee_public_resume_view_form_inherit` | xpath | `hr_skills.hr_employee_public_view_form_inherit` | `course_url` |  |  | `hr_skills_slides` |
| `hr_skills_slides.hr_employee_public_view_form` | xpath | `hr.hr_employee_public_view_form` | `has_subscribed_courses`, `courses_completion_text` | `action_open_courses` |  | `hr_skills_slides` |
| `hr_timesheet.hr_employee_public_view_form` | xpath | `hr.hr_employee_public_view_form` | `has_timesheet` | `action_timesheet_from_employee` |  | `hr_timesheet` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr.hr_employee_public_action` | Employees | kanban,list,form | `[('company_id', 'in', allowed_company_ids)]` | `{'chat_icon': True}` |  | `hr` |
| `hr_attendance.hr_employee_attendance_action_kanban` | Employees | kanban |  |  | fullscreen | `hr_attendance` |
| `hr_org_chart.action_hr_employee_public_org_chart` | Org Chart | hierarchy,kanban,list,form,graph,pivot | `[]` | `{'chat_icon': True}` |  | `hr_org_chart` |

Machine-readable definition: `../../../schemas/data/entities/hr.employee.public.json`; views: `../../../schemas/interfaces/views/hr.employee.public.json`.
