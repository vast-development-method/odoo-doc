# Employee (`hr.employee`)

**Transport name:** `hr.employee`  
**Storage name:** `hr_employee`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr`  
**Extended by packages:** `hr_attendance`, `hr_expense`, `hr_fleet`, `hr_gamification`, `hr_holidays`, `hr_holidays_attendance`, `hr_homeworking`, `hr_holidays_homeworking`, `hr_homeworking_calendar`, `hr_hourly_cost`, `hr_maintenance`, `hr_org_chart`, `hr_org_chart`, `hr_presence`, `hr_recruitment`, `hr_skills`, `hr_skills_slides`, `hr_timesheet`, `hr_work_entry`, `l10n_in_hr_holidays`, `pos_hr`, `project_timesheet_holidays`, `sale_timesheet`

Description: Employee

## Identity and behavior

- Mixins (classical inheritance): `mail.thread.main.attachment`, `mail.activity.mixin`, `resource.mixin`, `avatar.mixin`, `pos.load.mixin`
- Delegation inheritance: embeds `hr.version` through field `version_id`
- Default ordering: `name, id`
- Posting a message requires access: read
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (179)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `version_id` | Version | many to one | `hr.version` | required; computed by rule `_compute_version_id` (not stored); searchable through a search rule; visible only to groups `hr.group_hr_user`; on delete of the target: cascade |
| `current_version_id` | Current Version | many to one | `hr.version` | computed by rule `_compute_current_version_id` and stored |
| `current_date_version` | Current Date Version | date |  | related through path `current_version_id.date_version`; visible only to groups `hr.group_hr_user` |
| `version_ids` | Employee Versions | one to many | `hr.version` | required; visible only to groups `hr.group_hr_user`; inverse field `employee_id` |
| `versions_count` | Versions Count | integer |  | computed by rule `_compute_versions_count` (not stored); visible only to groups `hr.group_hr_user` |
| `version_revision` | Version Revision | single line text |  | computed by rule `_compute_version_revision` (not stored); visible only to groups `hr.group_hr_user` |
| `name` | Employee Name | single line text |  | related through path `resource_id.name` and stored; changes are tracked in the message thread |
| `resource_id` | Resource | many to one | `resource.resource` | required |
| `resource_calendar_id` | Resource Calendar | many to one |  | related through path `version_id.resource_calendar_id`; must belong to the same company |
| `user_id` | User | many to one | `res.users` | related through path `resource_id.user_id` and stored; indexed (btree_not_null); on delete of the target: restrict; must belong to the same company; precomputed before insertion |
| `user_partner_id` | User's partner | many to one |  | related through path `user_id.partner_id` |
| `share` | Share | boolean |  | related through path `user_id.share` |
| `phone` | Phone | single line text |  | related through path `user_id.phone` |
| `im_status` | Im Status | single line text |  | related through path `user_id.im_status` |
| `email` | Email | single line text |  | related through path `user_id.email` |
| `hr_presence_state` | Human resources Presence State | selection |  | computed by rule `_compute_presence_state` (not stored); default `out_of_working_hour` |
| `last_activity` | Last Activity | date |  | computed by rule `_compute_last_activity` (not stored) |
| `last_activity_time` | Last Activity Time | single line text |  | computed by rule `_compute_last_activity` (not stored) |
| `hr_icon_display` | Human resources Icon Display | selection |  | computed by rule `_compute_presence_icon` (not stored); extended by packages `hr_holidays`, `hr_homeworking` |
| `show_hr_icon_display` | Show Human resources Icon Display | boolean |  | computed by rule `_compute_presence_icon` (not stored) |
| `newly_hired` | Newly Hired | boolean |  | computed by rule `_compute_newly_hired` (not stored); searchable through a search rule |
| `active` | Active | boolean |  | related through path `resource_id.active` and stored; default `True` |
| `company_id` | Company | many to one | `res.company` | required; changes are tracked in the message thread |
| `company_country_id` | Company Country | many to one | `res.country` | read only; related through path `company_id.country_id`; visible only to groups `base.group_system,hr.group_hr_user` |
| `company_country_code` | Company Country Code | single line text |  | read only; related through path `company_country_id.code`; visible only to groups `base.group_system,hr.group_hr_user` |
| `work_phone` | Work Phone | single line text |  | computed by rule `_compute_work_contact_details` and stored; writable through an inverse rule; changes are tracked in the message thread |
| `mobile_phone` | Work Mobile | single line text |  |  |
| `work_email` | Work Email | single line text |  | computed by rule `_compute_work_contact_details` and stored; writable through an inverse rule |
| `work_contact_id` | Work Contact | many to one | `res.partner` | indexed (btree_not_null); not copied on duplication |
| `legal_name` | Legal Name | single line text |  | computed by rule `_compute_legal_name` and stored; visible only to groups `hr.group_hr_user` |
| `is_user_active` | User's active | boolean |  | related through path `user_id.active`; visible only to groups `hr.group_hr_user` |
| `private_phone` | Private Phone | single line text |  | visible only to groups `hr.group_hr_user` |
| `private_email` | Private Email | single line text |  | visible only to groups `hr.group_hr_user` |
| `lang` | Lang | selection |  | visible only to groups `hr.group_hr_user` |
| `place_of_birth` | Place of Birth | single line text |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `country_of_birth` | Country of Birth | many to one | `res.country` | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `birthday` | Birthday | date |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `birthday_public_display` | Show to all employees | boolean |  | default ; visible only to groups `hr.group_hr_user` |
| `birthday_public_display_string` | Public Date of Birth | single line text |  | computed by rule `_compute_birthday_public_display_string` (not stored); default `hidden` |
| `bank_account_ids` | Bank Accounts | many to many | `res.partner.bank` | changes are tracked in the message thread; not copied on duplication; visible only to groups `hr.group_hr_user`; restricted by domain `[('partner_id', '=', work_contact_id), '\|', ('company_id', '=', False), ('company_id', '=', company_id)]`; association table `employee_bank_account_rel`; Help: Employee bank accounts to pay salaries |
| `is_trusted_bank_account` | Is Trusted Bank Account | boolean |  | computed by rule `_compute_is_trusted_bank_account` (not stored); visible only to groups `hr.group_hr_user` |
| `primary_bank_account_id` | Primary Bank Account | many to one | `res.partner.bank` | computed by rule `_compute_primary_bank_account_id` (not stored); visible only to groups `hr.group_hr_user` |
| `has_multiple_bank_accounts` | Has Multiple Bank Accounts | boolean |  | computed by rule `_compute_has_multiple_bank_accounts` (not stored); default ; visible only to groups `hr.group_hr_user` |
| `salary_distribution` | Salary Distribution | structured document |  | computed by rule `_sync_salary_distribution` and stored; visible only to groups `hr.group_hr_user` |
| `permit_no` | Work Permit No | single line text |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `visa_no` | Visa No | single line text |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `visa_expire` | Visa Expiration Date | date |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `work_permit_expiration_date` | Work Permit Expiration Date | date |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `has_work_permit` | Work Permit | binary |  | visible only to groups `hr.group_hr_user` |
| `work_permit_scheduled_activity` | Work Permit Scheduled Activity | boolean |  | default ; visible only to groups `hr.group_hr_user` |
| `work_permit_name` | work_permit_name | single line text |  | computed by rule `_compute_work_permit_name` (not stored); visible only to groups `hr.group_hr_user` |
| `certificate` | Certificate Level | selection |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user`; values provided by rule `_get_certificate_selection` |
| `study_field` | Field of Study | single line text |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `study_school` | School | single line text |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `emergency_contact` | Emergency Contact | single line text |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `emergency_phone` | Emergency Phone | single line text |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `work_location_name` | Work Location Name | single line text |  | computed by rule `_compute_work_location_name` (not stored) |
| `work_location_type` | Work Location Type | selection |  | computed by rule `_compute_work_location_type` (not stored); changes are tracked in the message thread |
| `contract_date_start` | Contract Date Start | date |  | related through path `version_id.contract_date_start`; visible only to groups `hr.group_hr_manager` |
| `contract_date_end` | Contract Date End | date |  | related through path `version_id.contract_date_end`; visible only to groups `hr.group_hr_manager` |
| `trial_date_end` | Trial Date End | date |  | related through path `version_id.trial_date_end`; visible only to groups `hr.group_hr_manager` |
| `contract_wage` | Contract Wage | monetary |  | related through path `version_id.contract_wage`; visible only to groups `hr.group_hr_manager` |
| `date_start` | Date Start | date |  | related through path `version_id.date_start`; visible only to groups `hr.group_hr_manager` |
| `date_end` | Date End | date |  | related through path `version_id.date_end`; visible only to groups `hr.group_hr_manager` |
| `is_current` | Is Current | boolean |  | related through path `version_id.is_current`; visible only to groups `hr.group_hr_manager` |
| `is_past` | Is Past | boolean |  | related through path `version_id.is_past`; visible only to groups `hr.group_hr_manager` |
| `is_future` | Is Future | boolean |  | related through path `version_id.is_future`; visible only to groups `hr.group_hr_manager` |
| `is_in_contract` | Is In Contract | boolean |  | related through path `version_id.is_in_contract`; visible only to groups `hr.group_hr_manager` |
| `structure_type_id` | Structure Type | many to one |  | related through path `version_id.structure_type_id`; visible only to groups `hr.group_hr_manager` |
| `contract_type_id` | Contract Type | many to one |  | related through path `version_id.contract_type_id`; visible only to groups `hr.group_hr_manager` |
| `parent_id` | Manager | many to one | `hr.employee` | changes are tracked in the message thread; indexed; restricted by domain `['\|', ('company_id', '=', False), ('company_id', 'in', allowed_company_ids)]` |
| `child_ids` | Direct subordinates | one to many | `hr.employee` | restricted by domain `[["active", "=", true]]`; inverse field `parent_id` |
| `coach_id` | Coach | many to one | `hr.employee` | computed by rule `_compute_coach` and stored; restricted by domain `['\|', ('company_id', '=', False), ('company_id', 'in', allowed_company_ids)]`; Help: Select the "Employee" who is the coach of this employee. The "Coach" has no specific rights or responsibilities by default. |
| `category_ids` | Tags | many to many | `hr.employee.category` | visible only to groups `hr.group_hr_user`; association table `employee_category_rel` |
| `tz` | Tz | selection |  | changes are tracked in the message thread |
| `color` | Color Index | integer |  | default  |
| `barcode` | Badge identifier | single line text |  | not copied on duplication; visible only to groups `hr.group_hr_user`; Help: ID used for employee identification. |
| `pin` | PIN | single line text |  | not copied on duplication; visible only to groups `hr.group_hr_user`; Help: PIN used to Check In/Out in the Kiosk Mode of the Attendance application (if enabled in Configuration) and to change the cashier in the Point of Sale application. |
| `message_main_attachment_id` | Message Main Attachment | many to one |  | visible only to groups `hr.group_hr_user` |
| `id_card` | identifier Card Copy | binary |  | visible only to groups `hr.group_hr_user` |
| `driving_license` | Driving License | binary |  | visible only to groups `hr.group_hr_user` |
| `private_car_plate` | Private Car Plate | single line text |  | visible only to groups `hr.group_hr_user`; Help: If you have more than one car, just separate the plates by a space. |
| `currency_id` | Currency | many to one | `res.currency` | read only; related through path `company_id.currency_id`; visible only to groups `hr.group_hr_user` |
| `related_partners_count` | Related Partners Count | integer |  | computed by rule `_compute_related_partners_count` (not stored); visible only to groups `hr.group_hr_user` |
| `employee_properties` | Properties | properties |  | visible only to groups `hr.group_hr_user` |
| `activity_ids` | Activity | one to many |  | visible only to groups `hr.group_hr_user` |
| `activity_state` | Activity State | selection |  | visible only to groups `hr.group_hr_user` |
| `activity_user_id` | Activity User | many to one |  | visible only to groups `hr.group_hr_user` |
| `activity_type_id` | Activity Type | many to one |  | visible only to groups `hr.group_hr_user` |
| `activity_type_icon` | Activity Type Icon | single line text |  | visible only to groups `hr.group_hr_user` |
| `activity_date_deadline` | Activity Date Deadline | date |  | visible only to groups `hr.group_hr_user` |
| `my_activity_date_deadline` | My Activity Date Deadline | date |  | visible only to groups `hr.group_hr_user` |
| `activity_summary` | Activity Summary | single line text |  | visible only to groups `hr.group_hr_user` |
| `activity_exception_decoration` | Activity Exception Decoration | selection |  | visible only to groups `hr.group_hr_user` |
| `activity_exception_icon` | Activity Exception Icon | single line text |  | visible only to groups `hr.group_hr_user` |
| `message_is_follower` | Message Is Follower | boolean |  | visible only to groups `hr.group_hr_user` |
| `message_follower_ids` | Message Follower | one to many |  | visible only to groups `hr.group_hr_user` |
| `message_partner_ids` | Message Partner | many to many |  | visible only to groups `hr.group_hr_user` |
| `message_ids` | Message | one to many |  | visible only to groups `hr.group_hr_user` |
| `has_message` | Has Message | boolean |  | visible only to groups `hr.group_hr_user` |
| `message_needaction` | Message Needaction | boolean |  | visible only to groups `hr.group_hr_user` |
| `message_needaction_counter` | Message Needaction Counter | integer |  | visible only to groups `hr.group_hr_user` |
| `message_has_error` | Message Has Error | boolean |  | visible only to groups `hr.group_hr_user` |
| `message_has_error_counter` | Message Has Error Counter | integer |  | visible only to groups `hr.group_hr_user` |
| `message_attachment_count` | Message Attachment Count | integer |  | visible only to groups `hr.group_hr_user` |
| `attendance_manager_id` | Attendance Approver | many to one | `res.users` | visible only to groups `hr_attendance.group_hr_attendance_officer`; restricted by domain `[('share', '=', False), ('company_ids', 'in', company_id)]`; Help: The user set in Attendance will access the attendance of the employee through the dedicated app and will be able to edit them. |
| `attendance_ids` | Attendance | one to many | `hr.attendance` | visible only to groups `hr_attendance.group_hr_attendance_officer,hr.group_hr_user`; inverse field `employee_id` |
| `last_attendance_id` | Last Attendance | many to one | `hr.attendance` | computed by rule `_compute_last_attendance_id` and stored; visible only to groups `hr_attendance.group_hr_attendance_officer,hr.group_hr_user` |
| `last_check_in` | Last Check In | date and time |  | related through path `last_attendance_id.check_in` and stored; visible only to groups `hr_attendance.group_hr_attendance_officer,hr.group_hr_user` |
| `last_check_out` | Last Check Out | date and time |  | related through path `last_attendance_id.check_out` and stored; visible only to groups `hr_attendance.group_hr_attendance_officer,hr.group_hr_user` |
| `attendance_state` | Attendance Status | selection |  | computed by rule `_compute_attendance_state` (not stored); visible only to groups `hr_attendance.group_hr_attendance_officer,hr.group_hr_user` |
| `hours_last_month` | Hours Last Month | float |  | computed by rule `_compute_hours_last_month` (not stored) |
| `hours_last_month_overtime` | Hours Last Month Overtime | float |  | computed by rule `_compute_hours_last_month` (not stored) |
| `hours_today` | Hours Today | float |  | computed by rule `_compute_hours_today` (not stored); visible only to groups `hr_attendance.group_hr_attendance_officer,hr.group_hr_user` |
| `hours_previously_today` | Hours Previously Today | float |  | computed by rule `_compute_hours_today` (not stored); visible only to groups `hr_attendance.group_hr_attendance_officer,hr.group_hr_user` |
| `last_attendance_worked_hours` | Last Attendance Worked Hours | float |  | computed by rule `_compute_hours_today` (not stored); visible only to groups `hr_attendance.group_hr_attendance_officer,hr.group_hr_user` |
| `hours_last_month_display` | Hours Last Month Display | single line text |  | computed by rule `_compute_hours_last_month` (not stored); visible only to groups `hr.group_hr_user` |
| `overtime_ids` | Overtime | one to many | `hr.attendance.overtime.line` | visible only to groups `hr_attendance.group_hr_attendance_officer,hr.group_hr_user`; inverse field `employee_id` |
| `total_overtime` | Total Overtime | float |  | computed by rule `_compute_total_overtime` (not stored) |
| `display_extra_hours` | Display Extra Hours | boolean |  | related through path `company_id.hr_attendance_display_overtime` |
| `ruleset_id` | Ruleset | many to one |  | related through path `version_id.ruleset_id`; visible only to groups `hr.group_hr_manager` |
| `display_attendances` | Display Attendances | boolean |  | computed by rule `_compute_display_attendances` (not stored) |
| `expense_manager_id` | Expense Approver | many to one | `res.users` | computed by rule `_compute_expense_manager` and stored; restricted by domain `_group_hr_expense_user_domain`; Help: Select the user responsible for approving "Expenses" of this employee. If empty, the approval is done by an Administrator or Approver (determined in settings/users). |
| `filter_for_expense` | Filter For Expense | boolean |  | searchable through a search rule; visible only to groups `hr.group_hr_user,hr_expense.group_hr_expense_manager` |
| `employee_cars_count` | Cars | integer |  | computed by rule `_compute_employee_cars_count` (not stored); visible only to groups `fleet.fleet_group_manager` |
| `car_ids` | Vehicles (private) | one to many | `fleet.vehicle` | visible only to groups `fleet.fleet_group_manager,hr.group_hr_user`; inverse field `driver_employee_id` |
| `license_plate` | License Plate | single line text |  | computed by rule `_compute_license_plate` (not stored); searchable through a search rule; visible only to groups `hr.group_hr_user` |
| `mobility_card` | Mobility Card | single line text |  | visible only to groups `fleet.fleet_group_user` |
| `goal_ids` | Employee human resources Goals | one to many | `gamification.goal` | computed by rule `_compute_employee_goals` (not stored); visible only to groups `hr.group_hr_user` |
| `badge_ids` | Employee Badges | one to many | `gamification.badge.user` | computed by rule `_compute_employee_badges` (not stored); Help: All employee badges, linked to the employee either directly or through the user |
| `has_badges` | Has Badges | boolean |  | computed by rule `_compute_employee_badges` (not stored) |
| `direct_badge_ids` | Direct Badge | one to many | `gamification.badge.user` | visible only to groups `hr.group_hr_user`; inverse field `employee_id`; Help: Badges directly linked to the employee |
| `leave_manager_id` | Time Off Approver | many to one | `res.users` | computed by rule `_compute_leave_manager` and stored; restricted by domain `[('share', '=', False), ('company_ids', 'in', company_id)]`; Help: Select the user responsible for approving "Time Off" of this employee. If empty, the approval is done by an Administrator or Approver (determined in settings/users). |
| `current_leave_id` | Current Time Off Type | many to one | `hr.leave.type` | computed by rule `_compute_current_leave` (not stored); visible only to groups `hr.group_hr_user` |
| `current_leave_state` | Current Time Off Status | selection |  | computed by rule `_compute_leave_status` (not stored); visible only to groups `hr.group_hr_user` |
| `leave_date_from` | From Date | date |  | computed by rule `_compute_leave_status` (not stored); visible only to groups `hr.group_hr_user` |
| `leave_date_to` | To Date | date |  | computed by rule `_compute_leave_status` (not stored) |
| `allocation_count` | Total number of days allocated. | float |  | computed by rule `_compute_allocation_count` (not stored); visible only to groups `hr.group_hr_user` |
| `allocations_count` | Total number of allocations | integer |  | computed by rule `_compute_allocation_count` (not stored); visible only to groups `hr.group_hr_user` |
| `show_leaves` | Able to see Remaining Time Off | boolean |  | computed by rule `_compute_show_leaves` (not stored) |
| `is_absent` | Absent Today | boolean |  | computed by rule `_compute_leave_status` (not stored); searchable through a search rule |
| `allocation_display` | Allocation Display | single line text |  | computed by rule `_compute_allocation_remaining_display` (not stored) |
| `allocation_remaining_display` | Allocation Remaining Display | single line text |  | computed by rule `_compute_allocation_remaining_display` (not stored) |
| `monday_location_id` | Monday | many to one | `hr.work.location` |  |
| `tuesday_location_id` | Tuesday | many to one | `hr.work.location` |  |
| `wednesday_location_id` | Wednesday | many to one | `hr.work.location` |  |
| `thursday_location_id` | Thursday | many to one | `hr.work.location` |  |
| `friday_location_id` | Friday | many to one | `hr.work.location` |  |
| `saturday_location_id` | Saturday | many to one | `hr.work.location` |  |
| `sunday_location_id` | Sunday | many to one | `hr.work.location` |  |
| `exceptional_location_id` | Current | many to one | `hr.work.location` | computed by rule `_compute_exceptional_location_id` (not stored); visible only to groups `hr.group_hr_user`; Help: This is the exceptional, non-weekly, location set for today. |
| `today_location_name` | Today Location Name | single line text |  |  |
| `hourly_cost` | Hourly Cost | monetary |  | default ; changes are tracked in the message thread; visible only to groups `hr.group_hr_user`; currency taken from `currency_id` |
| `equipment_ids` | Equipment | one to many | `maintenance.equipment` | visible only to groups `hr.group_hr_user`; inverse field `employee_id` |
| `equipment_count` | Equipment Count | integer |  | computed by rule `_compute_equipment_count` (not stored) |
| `subordinate_ids` | Subordinates | one to many | `hr.employee` | computed by rule `_compute_subordinates` (not stored); Help: Direct and indirect subordinates |
| `is_subordinate` | Is Subordinate | boolean |  | computed by rule `_compute_is_subordinate` (not stored); searchable through a search rule |
| `child_all_count` | Indirect Subordinates Count | integer |  | computed by rule `_compute_subordinates` (not stored); recursive dependency |
| `department_color` | Department Color | integer |  | related through path `department_id.color` |
| `child_count` | Direct Subordinates Count | integer |  | computed by rule `_compute_child_count` (not stored); recursive dependency |
| `email_sent` | Email Sent | boolean |  | default  |
| `ip_connected` | Internet protocol Connected | boolean |  | default  |
| `manually_set_present` | Manually Set Present | boolean |  | default  |
| `manually_set_presence` | Manually Set Presence | boolean |  | default  |
| `hr_presence_state_display` | Human resources Presence State Display | selection |  | default `out_of_working_hour` |
| `applicant_ids` | Applicants | one to many | `hr.applicant` | visible only to groups `hr.group_hr_user`; inverse field `employee_id` |
| `resume_line_ids` | Resume lines | one to many | `hr.resume.line` | inverse field `employee_id` |
| `employee_skill_ids` | Skills | one to many | `hr.employee.skill` | restricted by domain `[["skill_type_id.active", "=", true]]`; inverse field `employee_id` |
| `current_employee_skill_ids` | Current Employee Skill | one to many | `hr.employee.skill` | computed by rule `_compute_current_employee_skill_ids` (not stored) |
| `skill_ids` | Skill | many to many | `hr.skill` | computed by rule `_compute_skill_ids` and stored; visible only to groups `hr.group_hr_user` |
| `certification_ids` | Certification | one to many | `hr.employee.skill` | computed by rule `_compute_certification_ids` (not stored) |
| `display_certification_page` | Display Certification Page | boolean |  | computed by rule `_compute_display_certification_page` (not stored) |
| `subscribed_courses` | Subscribed Courses | many to many | `slide.channel` | related through path `user_partner_id.slide_channel_ids` |
| `has_subscribed_courses` | Has Subscribed Courses | boolean |  | computed by rule `_compute_courses_completion_text` (not stored) |
| `courses_completion_text` | Courses Completion Text | single line text |  | computed by rule `_compute_courses_completion_text` (not stored) |
| `has_timesheet` | Has Timesheet | boolean |  | computed by rule `_compute_has_timesheet` (not stored) |
| `has_work_entries` | Has Work Entries | boolean |  | computed by rule `_compute_has_work_entries` (not stored); visible only to groups `base.group_system,hr.group_hr_user` |
| `work_entry_source` | Work Entry Source | selection |  | related through path `version_id.work_entry_source`; visible only to groups `hr.group_hr_manager` |
| `work_entry_source_calendar_invalid` | Work Entry Source Calendar Invalid | boolean |  | related through path `version_id.work_entry_source_calendar_invalid`; visible only to groups `hr.group_hr_manager` |

## Selection values

### `hr_presence_state` (Human resources Presence State)

| Value | Label |
|---|---|
| `present` | Present |
| `absent` | Absent |
| `archive` | Archived |
| `out_of_working_hour` | Off-Hours |

### `hr_icon_display` (Human resources Icon Display)

| Value | Label |
|---|---|
| `presence_present` | Present |
| `presence_out_of_working_hour` | Off-Hours |
| `presence_absent` | Absent |
| `presence_archive` | Archived |
| `presence_undetermined` | Undetermined |
| `presence_holiday_absent` | On leave |
| `presence_holiday_present` | Present but on leave |
| `presence_home` | At Home |
| `presence_office` | At Office |
| `presence_other` | At Other |

### `work_location_type` (Work Location Type)

| Value | Label |
|---|---|
| `home` | Home |
| `office` | Office |
| `other` | Other |

### `attendance_state` (Attendance Status)

| Value | Label |
|---|---|
| `checked_out` | Checked out |
| `checked_in` | Checked in |

### `current_leave_state` (Current Time Off Status)

| Value | Label |
|---|---|
| `confirm` | Waiting Approval |
| `refuse` | Refused |
| `validate1` | Waiting Second Approval |
| `validate` | Approved |
| `cancel` | Cancelled |

### `hr_presence_state_display` (Human resources Presence State Display)

| Value | Label |
|---|---|
| `out_of_working_hour` | Off-Hours |
| `present` | Present |
| `absent` | Absent |

## State fields

State machine fields of this entity: `hr_presence_state`, `activity_state`, `attendance_state`, `current_leave_state`. Transitions are specified in the domain documents.

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_barcode_uniq` | Constraint | `unique (barcode)` | The Badge ID must be unique, this one is already assigned to another employee. | `hr` |
| `_user_uniq` | Constraint | `unique (user_id, company_id)` | A user cannot be linked to multiple employees in the same company. | `hr` |

## Operations (206)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_lang_get` | internal rule | self | `hr` | model |  |
| `_prepare_create_values` | preparation rule | self, vals_list | `hr` |  |  |
| `_compute_is_trusted_bank_account` | computation | self | `hr` | depends: `bank_account_ids.allow_out_payment` |  |
| `_compute_has_multiple_bank_accounts` | computation | self | `hr` | depends: `bank_account_ids` |  |
| `_sync_salary_distribution` | computation | self | `hr` | depends: `bank_account_ids.active` |  |
| `_check_salary_distribution` | validation | self | `hr` | constrains: `salary_distribution` |  |
| `_create` | internal rule | self, data_list | `hr` | model |  |
| `check_field_access_rights` | operation | self, operation, field_names | `hr` | model |  |
| `_has_field_access` | internal rule | self, field, operation | `hr` |  |  |
| `check_no_existing_contract` | operation | self, date | `hr` |  |  |
| `_onchange_contract_template_id` | on change | self | `hr` | onchange: `contract_template_id` |  |
| `_onchange_contract_date_start` | on change | self | `hr` | onchange: `contract_date_start` |  |
| `_onchange_private_state_id` | on change | self | `hr` | onchange: `private_state_id` |  |
| `_onchange_phone_validation_employee` | on change | self | `hr` | onchange: `work_phone`, `mobile_phone`, `company_country_id`, `company_id` |  |
| `_get_new_hire_field` | preparation rule | self | `hr` | model |  |
| `_compute_newly_hired` | computation | self | `hr` |  |  |
| `_compute_presence_icon` | computation | self | `hr_attendance`, `hr_holidays_homeworking`, `hr_holidays`, `hr_homeworking`, `hr` | depends: `resource_calendar_id`, `hr_presence_state`; depends: `exceptional_location_id` | This method compute the state defining the display icon in the kanban view. It can be overriden to add other possibilities, like time off or attendances recordings. |
| `_get_certificate_selection` | preparation rule | self | `hr` | model |  |
| `_get_first_versions` | preparation rule | self | `hr` |  |  |
| `_get_first_versions_filtered` | preparation rule | self, no_gap | `hr` |  |  |
| `_get_first_version_date` | preparation rule | self, no_gap | `hr` |  |  |
| `_get_first_contract_date` | preparation rule | self, no_gap | `hr` |  |  |
| `_compute_legal_name` | computation | self | `hr` | depends: `name` |  |
| `_compute_version_id` | computation | self | `hr` | depends: `current_version_id`; depends_context: `version_id` |  |
| `_compute_work_location_name` | computation | self | `hr_homeworking`, `hr` | depends: `version_id.work_location_id.name`; depends: `exceptional_location_id` |  |
| `_compute_work_location_type` | computation | self | `hr_homeworking`, `hr` | depends: `version_id.work_location_id.location_type`; depends: `exceptional_location_id` |  |
| `_compute_current_version_id` | computation | self | `hr` | depends: `version_ids.date_version`, `version_ids.active`, `active` |  |
| `_cron_update_current_version_id` | background operation | self | `hr` |  |  |
| `_search_version_id` | search rule | self, operator, value | `hr` |  |  |
| `_field_to_sql` | internal rule | self, alias, field_expr, query | `hr` |  | This is required to search for the related fields of version_id as version_id is not stored |
| `_get_version` | preparation rule | self, date | `hr` |  | Return the version that should be used for the given date. If no valid version is found, we return the very first version of the employee. |
| `create_version` | operation | self, values | `hr_work_entry`, `hr` |  |  |
| `create_contract` | operation | self, date | `hr` |  |  |
| `_is_in_contract` | internal rule | self, date | `hr` |  |  |
| `_get_contracts` | preparation rule | self, date_start, date_end, use_latest_version, domain | `hr` |  | Retrieve the contracts for employees within a specified date range and based on specified criteria, such as domain filtering and version selection.  This method is used to collect and organize employee contracts based on their versions, date ranges, and other specified options. The resulting contracts are grouped by employee, and their selection logic depends on whether the latest version should be used or not. It supports flexibility in contract retrieval by allowing optional filters for date range and domain.  Args:     date_start (Optional[datetime.date]): The start date to filter the contr |
| `_get_contract_versions` | preparation rule | self, date_start, date_end, domain | `hr` |  | Retrieves contract versions for employees within the specified date range and domain. The function constructs a dynamic domain to filter contracts based on the provided arguments and retrieves grouped results. The grouping ensures organization by employee and date, and the results are stored in a structured format for ease of use.  Args:     date_start (datetime.date \| None): The start date for filtering contracts.     date_end (datetime.date \| None): The end date for filtering contracts.     domain (list \| None): Additional domain constraints for filtering.  Returns:     dict: A dictionary |
| `_get_all_contract_dates` | preparation rule | self | `hr` |  | Return a list of intervals (date_from, date_to) where the employee is in contract. For a permanent contract, the interval is (date_from, False). |
| `_get_contract_dates` | preparation rule | self, date | `hr` |  | Return a tuple (date_from, date_to) of the contract at the date given. (False, False) if the employee is not in contract at that date. |
| `_compute_versions_count` | computation | self | `hr` |  |  |
| `_compute_version_revision` | computation | self | `hr` | depends: `version_ids.write_date` |  |
| `_search_newly_hired` | search rule | self, operator, value | `hr` |  |  |
| `_create_work_contacts` | internal rule | self | `hr` |  |  |
| `_compute_coach` | computation | self | `hr` | depends: `parent_id` |  |
| `_compute_work_contact_details` | computation | self | `hr` | depends: `work_contact_id`, `work_contact_id.phone`, `work_contact_id.email` |  |
| `_inverse_work_contact_details` | inverse computation | self | `hr` |  |  |
| `_get_employee_working_now` | preparation rule | self | `hr` | model | Sudo needed to get resource_calendar_id as its normally only accessible by hr_users on version model (accessible on employee by inherits). |
| `_compute_presence_state` | computation | self | `hr_attendance`, `hr_holidays`, `hr_presence`, `hr` | depends: `user_id.im_status`; depends: `user_id.im_status`, `attendance_state`; depends: `user_id.im_status`, `hr_presence_state_display` | This method is overritten in several other modules which add additional presence criterions. e.g. hr_attendance, hr_holidays |
| `_compute_last_activity` | computation | self | `hr` | depends: `user_id` |  |
| `_compute_avatar_1920` | computation | self | `hr` | depends: `name`, `user_id.avatar_1920`, `image_1920` |  |
| `_compute_avatar_1024` | computation | self | `hr` | depends: `name`, `user_id.avatar_1024`, `image_1024` |  |
| `_compute_avatar_512` | computation | self | `hr` | depends: `name`, `user_id.avatar_512`, `image_512` |  |
| `_compute_avatar_256` | computation | self | `hr` | depends: `name`, `user_id.avatar_256`, `image_256` |  |
| `_compute_avatar_128` | computation | self | `hr` | depends: `name`, `user_id.avatar_128`, `image_128` |  |
| `_compute_avatar` | computation | self, avatar_field, image_field | `hr` |  |  |
| `_compute_birthday_public_display_string` | computation | self | `hr` | depends: `birthday_public_display` |  |
| `_compute_work_permit_name` | computation | self | `hr` | depends: `name`, `permit_no` |  |
| `_get_partner_count_depends` | preparation rule | self | `hr_recruitment`, `hr` |  |  |
| `_compute_related_partners_count` | computation | self | `hr` | depends: |  |
| `_get_related_partners` | preparation rule | self | `hr_recruitment`, `hr` |  |  |
| `action_related_contacts` | user action | self | `hr` |  |  |
| `action_create_user` | user action | self | `hr` |  |  |
| `action_create_users_confirmation` | user action | self | `hr` |  |  |
| `action_create_users` | user action | self | `hr` |  |  |
| `_compute_display_name` | computation | self | `hr_timesheet`, `hr` | depends: `company_id`, `user_id`; depends_context: `allowed_company_ids` |  |
| `search_fetch` | operation | self, domain, field_names, offset, limit, order | `hr` | model |  |
| `fetch` | operation | self, field_names | `hr` |  |  |
| `_check_access` | validation | self, operation | `hr` |  |  |
| `_check_private_fields` | validation | self, field_names | `hr` |  | Check whether `field_names` contain private fields. |
| `_copy_cache_from` | internal rule | self, public, field_names | `hr` |  |  |
| `notify_expiring_contract_work_permit` | operation | self | `hr` | model |  |
| `get_view` | lifecycle override | self, view_id, view_type, **options | `hr` | model |  |
| `get_views` | operation | self, views, options | `hr_homeworking`, `hr` | model |  |
| `_search` | search rule | self, domain, offset, limit, order, bypass_access, **kwargs | `hr` | model | We override the _search because it is the method that checks the access rights This is correct to override the _search. That way we enforce the fact that calling search on an hr.employee returns a hr.employee recordset, even if you don't have access to this model, as the result of _search (the ids of the public employees) is to be browsed on the hr.employee model. This can be trusted as the ids of the public employees exactly match the ids of the related hr.employee. |
| `_load_demo_data` | internal rule | self | `hr` |  |  |
| `get_formview_id` | operation | self, access_uid | `hr` |  | Override this method in order to redirect many2one towards the right model depending on access_uid |
| `get_formview_action` | operation | self, access_uid | `hr` |  | Override this method in order to redirect many2one towards the right model depending on access_uid |
| `_verify_pin` | validation | self | `hr` | constrains: `pin` |  |
| `_verify_barcode` | validation | self | `hr` | constrains: `barcode` |  |
| `_onchange_user` | on change | self | `hr` | onchange: `user_id` |  |
| `_onchange_timezone` | on change | self | `hr` | onchange: `resource_calendar_id` |  |
| `_remove_work_contact_id` | internal rule | self, user, employee_company | `hr` |  | Remove work_contact_id for previous employee if the user is assigned to a new employee |
| `_sync_user` | internal rule | self, user, employee_has_image | `hr` |  |  |
| `_prepare_resource_values` | preparation rule | self, vals, tz | `hr` |  |  |
| `new` | operation | self, values, origin, ref | `hr` | model |  |
| `create` | lifecycle override | self, vals_list | `hr_attendance`, `hr_holidays`, `hr_recruitment`, `hr_skills`, `hr`, `project_timesheet_holidays` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `hr_attendance`, `hr_fleet`, `hr_holidays`, `hr_presence`, `hr_skills`, `hr`, `project_timesheet_holidays` |  |  |
| `unlink` | lifecycle override | self | `hr` |  |  |
| `_get_employee_m2o_to_empty_on_archived_employees` | preparation rule | self | `hr` |  |  |
| `_get_user_m2o_to_empty_on_archived_employees` | preparation rule | self | `hr_expense`, `hr_holidays`, `hr` |  |  |
| `action_unarchive` | lifecycle override | self | `hr` |  |  |
| `action_archive` | lifecycle override | self | `hr_attendance`, `hr` |  |  |
| `_onchange_company_id` | on change | self | `hr` | onchange: `company_id` |  |
| `_load_scenario` | internal rule | self | `hr_skills`, `hr` |  |  |
| `generate_random_barcode` | operation | self | `hr` |  |  |
| `_get_tz` | preparation rule | self | `hr` |  |  |
| `_get_tz_batch` | preparation rule | self | `hr` |  |  |
| `_get_calendar_tz_batch` | preparation rule | self, dt | `hr` |  | Return a mapping { employee id : employee's effective schedule's (at dt) timezone } |
| `_get_calendars` | preparation rule | self, date_from | `hr` |  |  |
| `_get_version_periods` | preparation rule | self, start, stop, field, check_contract | `hr` |  |  |
| `_get_calendar_periods` | preparation rule | self, start, stop, check_contract | `hr` |  | :param datetime start: the start of the period :param datetime stop: the stop of the period |
| `_adjust_leaves` | internal rule | self, leave_intervals | `hr_holidays`, `hr` |  |  |
| `_get_employee_unavailable_intervals` | preparation rule | self, start, stop | `hr` |  | returns a dict {employee_id: [{start, stop}]} for the unavailability intervals of each employee which is used for _gantt_unavailability |
| `_get_all_versions_with_contract_overlap_with_period` | preparation rule | self, date_from, date_to | `hr` | model | Returns the versions of all employees between date_from and date_to that have at least 1 day in contract during that period |
| `_get_unusual_days` | preparation rule | self, date_from, date_to | `hr` |  |  |
| `_employee_attendance_intervals` | internal rule | self, start, stop, lunch | `hr` |  |  |
| `_get_expected_attendances` | preparation rule | self, date_from, date_to | `hr` |  |  |
| `_get_calendar_attendances` | preparation rule | self, date_from, date_to | `hr` |  |  |
| `get_import_templates` | operation | self | `hr` | model |  |
| `_get_age` | preparation rule | self, target_date | `hr` |  |  |
| `_get_departure_date` | preparation rule | self | `hr` |  |  |
| `_get_versions_with_contract_overlap_with_period` | preparation rule | self, date_from, date_to | `hr` |  | Returns the versions of the employee between date_from and date_to that have at least 1 day in contract during that period |
| `get_avatar_card_data` | operation | self, fields | `hr` |  |  |
| `_phone_get_number_fields` | internal rule | self | `hr` |  |  |
| `_mail_get_partner_fields` | messaging hook | self, introspect_fields | `hr` |  |  |
| `action_open_versions` | user action | self | `hr` |  |  |
| `_get_store_avatar_card_fields` | preparation rule | self, target | `hr_holidays`, `hr` |  |  |
| `_compute_primary_bank_account_id` | computation | self | `hr` | depends: `bank_account_ids` |  |
| `get_accounts_with_fixed_allocations` | operation | self | `hr` |  |  |
| `get_bank_account_salary_allocation` | operation | self, account_id | `hr` |  |  |
| `get_remaining_percentage` | operation | self | `hr` |  |  |
| `action_open_allocation_wizard` | user action | self | `hr` |  |  |
| `action_toggle_primary_bank_account_trust` | user action | self | `hr` |  |  |
| `_compute_display_attendances` | computation | self | `hr_attendance` | depends_context: `uid`; depends: `user_id`, `user_id.group_ids` |  |
| `_compute_total_overtime` | computation | self | `hr_attendance` | depends: `overtime_ids.manual_duration`, `overtime_ids`, `overtime_ids.status` |  |
| `_compute_hours_last_month` | computation | self | `hr_attendance` |  | Compute hours and overtime hours in the current month, if we are the 15th of october, will compute from 1 oct to 15 oct |
| `_compute_hours_today` | computation | self | `hr_attendance` |  |  |
| `_compute_last_attendance_id` | computation | self | `hr_attendance` | depends: `attendance_ids` |  |
| `_compute_attendance_state` | computation | self | `hr_attendance` | depends: `last_attendance_id.check_in`, `last_attendance_id.check_out`, `last_attendance_id` |  |
| `_attendance_action_change` | internal rule | self, geo_information | `hr_attendance` |  | Check In/Check Out action Check In: create a new attendance record Check Out: modify check_out field of appropriate attendance record |
| `get_overtime_data` | operation | self, domain, employee_id | `hr_attendance` | model |  |
| `action_open_last_month_attendances` | user action | self | `hr_attendance` |  |  |
| `open_barcode_scanner` | operation | self | `hr_attendance` |  |  |
| `_get_schedules_by_employee_by_work_type` | preparation rule | self, start, stop, version_periods_by_employee | `hr_attendance` |  |  |
| `_group_hr_expense_user_domain` | internal rule | self | `hr_expense` |  |  |
| `_search_filter_for_expense` | search rule | self, operator, value | `hr_expense` |  |  |
| `_compute_expense_manager` | computation | self | `hr_expense` | depends: `parent_id` |  |
| `action_open_employee_cars` | user action | self | `hr_fleet` |  |  |
| `_compute_license_plate` | computation | self | `hr_fleet` | depends: `private_car_plate`, `car_ids.license_plate` |  |
| `_search_license_plate` | search rule | self, operator, value | `hr_fleet` |  |  |
| `_compute_employee_cars_count` | computation | self | `hr_fleet` |  |  |
| `_check_work_contact_id` | validation | self | `hr_fleet` | constrains: `work_contact_id` |  |
| `_compute_employee_goals` | computation | self | `hr_gamification` | depends: `user_id.goal_ids.challenge_id.challenge_category` |  |
| `_compute_employee_badges` | computation | self | `hr_gamification` | depends: `direct_badge_ids`, `user_id.badge_ids.employee_id` |  |
| `_compute_current_leave` | computation | self | `hr_holidays` |  |  |
| `_compute_allocation_count` | computation | self | `hr_holidays` |  |  |
| `_compute_allocation_remaining_display` | computation | self | `hr_holidays` |  |  |
| `_get_first_working_interval` | preparation rule | self, dt | `hr_holidays` |  |  |
| `_compute_leave_status` | computation | self | `hr_holidays` |  |  |
| `_compute_leave_manager` | computation | self | `hr_holidays` | depends: `parent_id` |  |
| `_compute_show_leaves` | computation | self | `hr_holidays` |  |  |
| `_search_absent_employee` | search rule | self, operator, value | `hr_holidays` |  |  |
| `action_time_off_dashboard` | user action | self | `hr_holidays` |  |  |
| `get_mandatory_days` | operation | self, start_date, end_date | `hr_holidays` |  |  |
| `get_special_days_data` | operation | self, date_start, date_end | `hr_holidays`, `l10n_in_hr_holidays` | model |  |
| `get_public_holidays_data` | operation | self, date_start, date_end | `hr_holidays` | model |  |
| `get_time_off_dashboard_data` | operation | self, target_date | `hr_holidays` | model |  |
| `get_allocation_requests_amount` | operation | self | `hr_holidays` | model |  |
| `_get_public_holidays` | preparation rule | self, date_start, date_end | `hr_holidays` |  |  |
| `get_mandatory_days_data` | operation | self, date_start, date_end | `hr_holidays` | model |  |
| `_get_mandatory_days` | preparation rule | self, start_date, end_date | `hr_holidays` |  |  |
| `_get_contextual_employee` | preparation rule | self | `hr_holidays` | model |  |
| `_get_consumed_leaves` | preparation rule | self, leave_types, target_date, ignore_future | `hr_holidays` |  | This method won't call `_get_future_leaves_on` for the allocations contained by this variable (it will only use the current value of the `number_of_days` of the allocation, alias `number_of_hours_display`)  `precomputed_allocations`: context variable (recordset) which can be used to pass allocation that are considered to be already computed |
| `_get_hours_per_day` | preparation rule | self, date_from | `hr_holidays` |  | Return 24H to handle the case of Fully Flexible (ones without a working calendar) |
| `_get_deductible_employee_overtime` | preparation rule | self | `hr_holidays_attendance` |  |  |
| `get_overtime_data_by_employee` | operation | self | `hr_holidays_attendance` |  | Provide a summary of an employee's overtime. A compensable overtime is an overtime that can be cumulated to be used as time off. Extra hours and overtime is used interchangably. |
| `_get_current_day_location_field` | preparation rule | self | `hr_homeworking` | model |  |
| `_compute_exceptional_location_id` | computation | self | `hr_homeworking` |  |  |
| `_get_worklocation` | preparation rule | self, start_date, end_date | `hr_homeworking_calendar` |  |  |
| `_compute_equipment_count` | computation | self | `hr_maintenance` | depends: `equipment_ids` |  |
| `_get_subordinates` | preparation rule | self, parents | `hr_org_chart` |  | Helper function to compute subordinates_ids. Get all subordinates (direct and indirect) of an employee. An employee can be a manager of his own manager (recursive hierarchy; e.g. the CEO is manager of everyone but is also member of the RD department, managed by the CTO itself managed by the CEO). In that case, the manager in not counted as a subordinate if it's in the 'parents' set. |
| `_compute_subordinates` | computation | self | `hr_org_chart` | depends: `child_ids`, `child_ids.child_all_count` |  |
| `_compute_is_subordinate` | computation | self | `hr_org_chart` | depends_context: `uid`, `company`; depends: `parent_id` |  |
| `_search_is_subordinate` | search rule | self, operator, value | `hr_org_chart` |  |  |
| `_compute_child_count` | computation | self | `hr_org_chart` |  |  |
| `_check_presence` | validation | self | `hr_presence` | model |  |
| `get_presence_server_action_data` | operation | self | `hr_presence` |  |  |
| `_action_set_manual_presence` | internal rule | self, state | `hr_presence` |  |  |
| `action_set_present` | user action | self | `hr_presence` |  |  |
| `action_set_absent` | user action | self | `hr_presence` |  |  |
| `action_open_leave_request` | user action | self | `hr_presence` |  |  |
| `action_send_sms` | user action | self | `hr_presence` |  |  |
| `action_send_log` | user action | self | `hr_presence` |  |  |
| `_compute_current_employee_skill_ids` | computation | self | `hr_skills` | depends: `employee_skill_ids` |  |
| `_compute_skill_ids` | computation | self | `hr_skills` | depends: `employee_skill_ids.skill_id` |  |
| `_compute_certification_ids` | computation | self | `hr_skills` | depends: `employee_skill_ids` |  |
| `_compute_display_certification_page` | computation | self | `hr_skills` |  |  |
| `_add_certification_activity_to_employees` | internal rule | self | `hr_skills` | model |  |
| `get_internal_resume_lines` | operation | self, res_id, res_model | `hr_skills` | model |  |
| `_compute_courses_completion_text` | computation | self | `hr_skills_slides` | depends_context: `lang`; depends: `subscribed_courses`, `user_partner_id.slide_channel_completed_ids` |  |
| `action_open_courses` | user action | self | `hr_skills_slides` |  |  |
| `_compute_has_timesheet` | computation | self | `hr_timesheet` |  |  |
| `action_unlink_wizard` | user action | self | `hr_timesheet` |  |  |
| `action_timesheet_from_employee` | user action | self | `hr_timesheet` |  |  |
| `_compute_has_work_entries` | computation | self | `hr_work_entry` |  |  |
| `action_open_work_entries` | user action | self, initial_date | `hr_work_entry` |  |  |
| `generate_work_entries` | operation | self, date_start, date_stop, force | `hr_work_entry` |  |  |
| `_get_optional_holidays_data` | preparation rule | self, date_start, date_end | `l10n_in_hr_holidays` |  |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `pos_hr` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `pos_hr` | model |  |
| `_server_date_to_domain` | internal rule | self, domain | `pos_hr` |  |  |
| `_load_pos_data_read` | internal rule | self, records, config | `pos_hr` | model |  |
| `get_barcodes_and_pin_hashed` | operation | self | `pos_hr` |  |  |
| `_unlink_except_active_pos_session` | internal rule | self | `pos_hr` | ondelete |  |
| `_delete_future_public_holidays_timesheets` | internal rule | self | `project_timesheet_holidays` |  |  |
| `_create_future_public_holidays_timesheets` | internal rule | self, employees | `project_timesheet_holidays` |  |  |
| `default_get` | lifecycle override | self, fields | `sale_timesheet` | model |  |

## Validation and error messages (21)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_salary_distribution` | ValidationError | Total salary distribution on bank accounts must be exactly 100%. | `hr` |
| `_check_salary_distribution` | ValidationError | Each amount percentage must be a number between 0 and 100. | `hr` |
| `check_no_existing_contract` | ValidationError | The employee is already in contract on %s. Please select a date outside existing contracts | `hr` |
| `_get_first_versions_filtered` | AccessError | Only HR users can access first version date on an employee. | `hr` |
| `_create_work_contacts` | UserError | Some employee already have a work contact | `hr` |
| `action_create_user` | ValidationError | This employee already has an user. | `hr` |
| `_check_private_fields` | AccessError | The fields “%s”, which you are trying to read, are not available for employee public profiles. | `hr` |
| `_search` | AccessError | You do not have access to this document. | `hr` |
| `_search` | AccessError | You do not have access to this document. | `hr` |
| `_verify_pin` | ValidationError | The PIN must be a sequence of digits. | `hr` |
| `_verify_barcode` | ValidationError | The Badge ID must be alphanumeric without any accents and no longer than 18 characters. | `hr` |
| `_get_version_periods` | UserError | This field %(field_name)s doesn't exist on this model (hr.version). | `hr` |
| `_attendance_action_change` | UserError | Cannot perform check out on %(empl_name)s, could not find corresponding check in. Your attendances have probably been modified manually by human resources. | `hr_attendance` |
| `_check_work_contact_id` | ValidationError | Cannot remove address from employees with linked cars. | `hr_fleet` |
| `write` | ValidationError | Changing this working schedule results in the affected employee(s) not having enough leaves allocated to accomodate for their leaves already taken in the future. Please review this employee's leaves and adjust their allocation accordingly. | `hr_holidays` |
| `_action_set_manual_presence` | UserError | You don't have the right to do this. Please contact an Administrator. | `hr_presence` |
| `action_send_sms` | UserError | You don't have the right to do this. Please contact an Administrator. | `hr_presence` |
| `action_send_log` | UserError | You don't have the right to do this. Please contact an Administrator. | `hr_presence` |
| `get_internal_resume_lines` | AccessError | You cannot access the resume of this employee. | `hr_skills` |
| `action_unlink_wizard` | UserError | You cannot delete employees who have timesheets. | `hr_timesheet` |
| `_unlink_except_active_pos_session` | UserError | error_msg | `pos_hr` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_user` | yes | yes | yes | yes | `hr` |
| `base.group_system` | no | yes | no | no | `hr` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Employee multi company rule | global (all users) | `['\|', '\|', '\|',             ('company_id', 'in', company_ids + [False]),             ('parent_id.user_id', '=', user.id),             ('id', '=', user.employee_id.parent_id.id),             ('user_id', '=', user.id)         ]` | True | True | True | True |

## Views (43)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr.view_employee_filter` | search |  | `name`, `department_id`, `company_id`, `department_id`, `parent_id`, `contract_date_start`, `job_id`, `coach_id`, `category_ids`, `private_car_plate`, `resource_calendar_id`, `company_id` |  | `Unread Messages`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `My Team`, `My Department`, `Newly Hired`, `In Contract`, `Out of Contract`, `Contract Start Date`, `Contract End Date`, `Archived`, `Manager`, `Department`, `Job Position`, `Employee Type`, `group_birthday`, `Start Date`, `Tags`, `Properties` | `hr` |
| `hr.view_employee_form` | form |  | `version_id`, `version_revision`, `versions_count`, `image_1920`, `show_hr_icon_display`, `name`, `work_email`, `work_phone`, `mobile_phone`, `category_ids`, `hr_icon_display`, `employee_properties`, `company_id`, `department_id`, `job_id`, `job_title`, `parent_id`, `address_id`, `work_location_id`, `departure_reason_id`, `departure_description`, `departure_date`, `additional_note`, `private_email`, `private_phone`, `has_multiple_bank_accounts`, `is_trusted_bank_account`, `bank_account_ids`, `bank_account_ids`, `bank_account_ids`, `legal_name`, `birthday`, `birthday_public_display`, `place_of_birth`, `country_of_birth`, `sex`, `emergency_contact`, `emergency_phone`, `visa_no`, `visa_expire`, `permit_no`, `work_permit_expiration_date`, `work_permit_name`, `has_work_permit`, `country_id`, `identification_id`, `ssnid`, `passport_id`, `passport_expiration_date`, `private_street`, `private_street2`, `private_city`, `private_state_id`, `private_zip`, `private_country_id`, `distance_home_work`, `distance_home_work_unit`, `marital`, `spouse_complete_name`, `spouse_birthdate` | `Create User`, `Launch Plan`, `action_open_versions`, `Salary Allocation`, `action_toggle_primary_bank_account_trust`, `action_toggle_primary_bank_account_trust`, `Load a Template`, `Generate`, `Print Badge` |  | `hr` |
| `hr.hr_employee_view_graph` | graph |  | `contract_date_start`, `id`, `color`, `distance_home_work`, `km_home_work`, `children` |  |  | `hr` |
| `hr.hr_employee_view_pivot` | pivot |  | `job_id`, `contract_date_start`, `id`, `color`, `distance_home_work`, `km_home_work`, `children` |  |  | `hr` |
| `hr.view_employee_tree` | list |  | `avatar_128`, `name`, `work_phone`, `work_email`, `hr_presence_state`, `contract_date_start`, `contract_date_end`, `currency_id`, `wage`, `employee_type`, `contract_type_id`, `structure_type_id`, `activity_ids`, `activity_user_id`, `activity_date_deadline`, `company_id`, `department_id`, `job_id`, `parent_id`, `address_id`, `company_id`, `birthday`, `work_location_id`, `coach_id`, `user_id`, `active`, `category_ids`, `country_id` | `Launch Plan` |  | `hr` |
| `hr.hr_employee_list_view` | list |  | `name`, `company_id`, `department_id`, `job_id`, `parent_id`, `contract_date_start`, `resource_calendar_id` |  |  | `hr` |
| `hr.hr_employee_list_activites_view` | list |  | `avatar_128`, `name`, `work_phone`, `work_email`, `contract_date_start`, `contract_date_end`, `currency_id`, `wage`, `employee_type`, `contract_type_id`, `structure_type_id`, `activity_ids`, `activity_user_id`, `activity_date_deadline`, `company_id`, `department_id`, `job_id`, `parent_id`, `address_id`, `company_id`, `birthday`, `work_location_id`, `coach_id`, `user_id`, `active`, `category_ids`, `country_id` |  |  | `hr` |
| `hr.hr_kanban_view_employees` | kanban |  | `show_hr_icon_display`, `image_128`, `company_id`, `image_1024`, `avatar_128`, `hr_icon_display`, `name`, `job_title`, `work_email`, `work_phone`, `contract_date_start`, `contract_date_end`, `birthday_public_display_string`, `employee_properties`, `category_ids`, `user_id`, `activity_ids` |  |  | `hr` |
| `hr.view_employee_form_smartbutton_inherited` | button | `view_employee_form` | `related_partners_count` | `action_open_versions`, `action_related_contacts` |  | `hr` |
| `hr.hr_employee_view_activity` | activity |  | `id`, `name`, `job_id` |  |  | `hr` |
| `hr_attendance.hr_employee_search_view` | xpath | `hr.view_employee_filter` |  |  | `Attendance Approver` | `hr_attendance` |
| `hr_attendance.view_employee_form_inherit_hr_attendance` | button | `hr.view_employee_form` | `attendance_state`, `hours_last_month`, `display_attendances`, `hours_last_month`, `hours_last_month_overtime`, `hours_last_month_overtime`, `hours_last_month`, `hours_last_month_overtime`, `hours_last_month_overtime` | `action_open_versions`, `action_open_last_month_attendances`, `action_open_last_month_attendances` |  | `hr_attendance` |
| `hr_attendance.hr_employees_view_kanban` | kanban |  | `attendance_state`, `avatar_128`, `name`, `job_id`, `work_location_id` |  |  | `hr_attendance` |
| `hr_attendance.view_employee_tree_inherit_leave` | xpath | `hr.view_employee_tree` | `attendance_manager_id` |  |  | `hr_attendance` |
| `hr_expense.hr_employee_view_form_inherit_expense` | xpath | `hr.view_employee_form` | `expense_manager_id` |  |  | `hr_expense` |
| `hr_expense.view_employee_tree_inherit_expense` | xpath | `hr.view_employee_tree` | `expense_manager_id` |  |  | `hr_expense` |
| `hr_expense.hr_employee_search_view` | xpath | `hr.view_employee_filter` |  |  | `Expense Approver` | `hr_expense` |
| `hr_fleet.view_employee_form` | button | `hr.view_employee_form` | `employee_cars_count` | `action_open_versions`, `action_open_employee_cars` |  | `hr_fleet` |
| `hr_fleet.view_employee_filter` | xpath | `hr.view_employee_filter` | `license_plate` |  |  | `hr_fleet` |
| `hr_gamification.hr_hr_employee_view_form` | xpath | `hr.view_employee_form` | `has_badges`, `badge_ids` | `Grant a Badge` |  | `hr_gamification` |
| `hr_holidays.hr_employee_view_search` | xpath | `hr.view_employee_filter` |  |  | `At work`, `On Time Off` | `hr_holidays` |
| `hr_holidays.hr_kanban_view_employees_kanban` | xpath | `hr.hr_kanban_view_employees` | `current_leave_id`, `current_leave_state`, `leave_date_from`, `leave_date_to`, `is_absent` |  |  | `hr_holidays` |
| `hr_holidays.view_employee_form_leave_inherit` | xpath | `hr.view_employee_form` | `leave_manager_id` |  |  | `hr_holidays` |
| `hr_holidays.view_employee_tree_inherit_leave` | xpath | `hr.view_employee_tree` | `leave_manager_id` |  |  | `hr_holidays` |
| `hr_holidays_attendance.hr_employee_view_form_inherit` | xpath | `hr.view_employee_form` | `total_overtime` | `Deduct Extra Hours` |  | `hr_holidays_attendance` |
| `hr_homeworking.view_employee_filter` | xpath | `hr.view_employee_filter` |  |  | `Work location` | `hr_homeworking` |
| `hr_homeworking.view_employee_form` | xpath | `hr.view_employee_form` | `monday_location_id`, `tuesday_location_id`, `wednesday_location_id`, `thursday_location_id`, `friday_location_id`, `saturday_location_id`, `sunday_location_id` |  |  | `hr_homeworking` |
| `hr_homeworking.view_employee_tree` | xpath | `hr.view_employee_tree` | `work_location_name` |  |  | `hr_homeworking` |
| `hr_hourly_cost.view_employee_form` | group | `hr.view_employee_form` |  |  |  | `hr_hourly_cost` |
| `hr_maintenance.hr_employee_view_form` | button | `hr.view_employee_form` | `equipment_count` | `action_open_versions`, `%(maintenance.hr_equipment_action)d` |  | `hr_maintenance` |
| `hr_org_chart.hr_employee_view_form_inherit_org_chart` | div | `hr.view_employee_form` | `child_ids` |  |  | `hr_org_chart` |
| `hr_org_chart.hr_employee_view_pivot_inherit_org_chart` | xpath | `hr.hr_employee_view_pivot` | `department_color` |  |  | `hr_org_chart` |
| `hr_org_chart.hr_employee_view_graph_inherit_org_chart` | xpath | `hr.hr_employee_view_graph` | `department_color` |  |  | `hr_org_chart` |
| `hr_org_chart.hr_employee_hierarchy_view` | hierarchy |  | `name`, `job_id`, `department_color`, `hr_icon_display`, `department_id`, `image_1024`, `name`, `hr_icon_display`, `job_title` |  |  | `hr_org_chart` |
| `hr_presence.hr_employee_view_search` | filter | `hr.view_employee_filter` |  |  | `at_work`, `Absent`, `Off-Hours` | `hr_presence` |
| `hr_skills.hr_employee_view_search` | xpath | `hr.view_employee_filter` | `employee_skill_ids`, `resume_line_ids` |  |  | `hr_skills` |
| `hr_skills.hr_employee_view_form` | div | `hr.view_employee_form` | `employee_id`, `resume_line_ids`, `line_type_id`, `name`, `description`, `date_start`, `date_end`, `is_course`, `duration`, `external_url` |  |  | `hr_skills` |
| `hr_skills_slides.hr_employee_view_form` | button | `hr.view_employee_form` | `has_subscribed_courses`, `courses_completion_text` | `action_open_versions`, `action_open_courses` |  | `hr_skills_slides` |
| `hr_skills_slides.hr_employee_resume_view_form_inherit` | xpath | `hr_skills.hr_employee_view_form` | `course_url` |  |  | `hr_skills_slides` |
| `hr_timesheet.hr_employee_view_form_inherit_timesheet` | xpath | `hr_hourly_cost.view_employee_form` |  |  |  | `hr_timesheet` |
| `hr_timesheet.view_employee_tree_inherit_timesheet` | xpath | `hr.view_employee_tree` |  |  |  | `hr_timesheet` |
| `hr_timesheet.hr_employee_view_kanban_inherit_timesheet` | xpath | `hr.hr_kanban_view_employees` |  |  |  | `hr_timesheet` |
| `hr_work_entry.hr_employee_view_form` | button | `hr.view_employee_form` | `has_work_entries` | `action_open_versions`, `action_open_work_entries` |  | `hr_work_entry` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr.open_view_employee_list_my` | Employees | kanban,list,form,activity,graph,pivot | `[('company_id', 'in', allowed_company_ids)]` | `{'chat_icon': True, 'searchpanel_default_company_id': allowed_company_ids[0]}` |  | `hr` |
| `hr.open_view_employee_list` | Employees | form,list |  |  |  | `hr` |
| `hr.action_hr_employee_all_activities` | All activities | activity,list,kanban,form,graph,pivot | `[                 ('company_id', 'in', allowed_company_ids),                 ('activity_ids', '!=', False),             ]` | `{'chat_icon': True, 'searchpanel_default_company_id': allowed_company_ids[0]}` |  | `hr` |
| `hr_holidays.hr_employee_action_from_department` | Absent Employees | list,kanban,form |  | `{            'search_default_on_timeoff': 1,            'searchpanel_default_department_id': active_id,            'search_default_department_id': active_id,            'default_department_id': active_id}` |  | `hr_holidays` |
| `hr_org_chart.action_hr_employee_org_chart` | Org Chart | hierarchy,kanban,list,form,activity,graph,pivot | `[]` | `{'chat_icon': True}` |  | `hr_org_chart` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `hr_attendance.menu_hr_attendance_employee` | Employees | `menu_hr_attendance_overview` | `hr.open_view_employee_list_my` | 2 | `hr_attendance.group_hr_attendance_officer` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `hr.action_hr_employee_load_demo_data` | Load Sample Data | code |  | yes |
| `hr.action_hr_employee_create_users_confirmation` | Create User | code |  | yes |
| `hr.action_hr_employee_create_users` | Create User | code |  | yes |
| `hr_presence.action_hr_employee_presence_present` | Set Present | code |  | yes |
| `hr_presence.action_hr_employee_presence_absent` | Set Absent | code |  | yes |
| `hr_presence.action_hr_employee_presence_log` | Add a log note | code |  | yes |
| `hr_presence.action_hr_employee_presence_sms` | Send a SMS | code |  | yes |
| `hr_presence.action_hr_employee_presence_time_off` | Create a Time Off | code |  | yes |
| `hr_skills.action_print_employees_cv` | Print Resume | code | report | yes |
| `hr_timesheet.unlink_employee_action` | Delete | code |  | yes |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `hr.hr_employee_print_badge` | Print Badge | qweb-pdf | `hr.print_employee_badge` | `'Badge - %s' % (object.name).replace('/', '')` |  |
| `hr_skills.action_report_employee_cv` | Employee Resume | qweb-pdf | `hr_skills.report_employee_cv` | `'CV - %s' % (object.name)` |  |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `hr.ir_cron_data_employee_notify_expiring_contract_work_permit` | HR Employee: Notify Expiring Contract or Work Permit | 1 days | `notify_expiring_contract_work_permit` |  |
| `hr.ir_cron_data_employee_update_current_version` | HR Employee: Update Current Version | 1 days | `_cron_update_current_version_id` |  |
| `hr_presence.ir_cron_presence_control` | HR Presence: cron | 1 hours | `_check_presence` |  |
| `hr_skills.hr_job_skills_cron_add_certification_activity_to_employees` | Skills: Add an activity to employees with missing or expiring certifications | 1 days | `_add_certification_activity_to_employees` |  |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `hr_presence.mail_template_presence` | HR: Employee Absence email | Unexpected Absence |

Machine-readable definition: `../../../schemas/data/entities/hr.employee.json`; views: `../../../schemas/interfaces/views/hr.employee.json`.
