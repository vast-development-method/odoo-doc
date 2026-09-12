# Configuration

Everything an installation ships for this domain, and everything an administrator can set. The document is exhaustive in both directions: where a category of configuration is empty — settings, sequences, scheduled jobs, message templates, activity types — that is stated and the reason is given, so that a rebuild does not go looking for something that is not there.

## 1. What this domain ships

| Category | Count | Where |
|---|---|---|
| Privilege | 1 | §2 |
| Access group | 1 | §2 |
| Access-right rows | 6 | §3 |
| Access groups borrowed from other packages | 5 platform-wide, plus the ten shipped audiences of §8 | §4, §8 |
| Record rules | 4 | §5 |
| Client actions | 1 | §6.1 |
| Window actions | 2, plus 5 from the conversation dashboards package | §6.2, §6.3 |
| Menus | 5, plus 5 from the conversation dashboards package | §6.4 |
| Views | 6 | §9 |
| Page templates | 1 | §9.5 |
| Dashboard groups | 7 | §7 |
| Dashboards | 14, shipped by twelve bridge packages | §8 |
| Session capability flags | 1 | §10 |
| Settings, system parameters | 0 | §11 |
| Sequences | 0 | §12 |
| Scheduled jobs | 0 | §13 |
| Message templates, activity types, subtypes | 0 | §14 |

## 2. The privilege and the administrator group

### 2.1 The privilege

| Property | Value |
|---|---|
| External identifier | `res_groups_privilege_dashboard` |
| Label | "Dashboard" |
| Sequence within its category | 30 |
| Category | the productivity category, external identifier `base.module_category_productivity` |

A privilege is the heading under which one or more access groups are offered on a user's form. This one carries exactly one group.

### 2.2 The dashboard administrator group

| Property | Value |
|---|---|
| External identifier | `spreadsheet_dashboard.group_dashboard_manager` |
| Label | "Admin" |
| Sequence within the privilege | 10 |
| Privilege | the one above |
| Implies | the internal-user group, external identifier `base.group_user` |
| Shipped members | the system user, external identifier `base.user_root`, and the administrator user, external identifier `base.user_admin` |

Holding this group is the whole of the administrative authority of this domain: it grants creation, modification and deletion of Spreadsheet Dashboards and Dashboard Groups (§3) and it widens the audience rule so that every dashboard is visible whatever its access groups (§5.3).

There is no second, weaker administrative group. A user either administers every dashboard or administers none.

## 3. Access rights

Six rows, all shipped by the dashboards package except the last, which is shipped by the personal-boards package.

| Entity | Group | Read | Update | Create | Delete |
|---|---|---|---|---|---|
| `spreadsheet.dashboard.group` — Dashboard Group | internal user | yes | no | no | no |
| `spreadsheet.dashboard` — Spreadsheet Dashboard | internal user | yes | no | no | no |
| `spreadsheet.dashboard.group` | dashboard administrator | yes | yes | yes | yes |
| `spreadsheet.dashboard` | dashboard administrator | yes | yes | yes | yes |
| `spreadsheet.dashboard.share` — Dashboard Share | internal user | yes | yes | yes | yes |
| `board.board` — Dashboard Board | internal user | yes | no | no | no |

Three consequences that a rebuild must reproduce:

1. **Every internal user may create a share.** Sharing is not an administrative act; the record rule of §5.4 is what keeps one user's shares out of another user's sight.
2. **No group at all may write a dashboard other than the administrator group.** The favourite mark is written with elevated rights for exactly that reason; see [`business-rules.md`](business-rules.md#sd-019).
3. **The board grants read only**, and the operation that would create a board record stores nothing, so the absence of a create right is never reached.

The Spreadsheet Document mixin has no rights of its own; it is abstract and every check falls on the entity that carries it.

## 4. Access groups borrowed from other packages

The domain does not define these; it reads them. Each is named by its external identifier because that identifier is the contract.

| External identifier | Full name | What this domain uses it for |
|---|---|---|
| `base.group_user` | Internal user | The baseline audience of a new dashboard; the group every access-right row above is granted to; the group the share record rule applies to |
| `base.group_system` | Technical settings | Shows the drag handle that reorders dashboards and dashboard groups in the configuration lists |
| `base.group_no_one` | Extended visibility | Shows the raw workbook file control in the dashboard configuration list |
| `base.group_multi_company` | Multiple companies | Shows the companies control on the dashboard form and list |
| `base.group_allow_export` | Data export | Required to download a shared dashboard's workbook file (rule [SD-023](business-rules.md#sd-023)) and to download a workbook from inside the application (rule [SD-066](business-rules.md#sd-066)) |

Three further groups exist only as the shipped audiences of shipped dashboards and are listed with them in §8.

## 5. Record rules

Four rules. Two apply to Spreadsheet Dashboard, one to Dashboard Share, and none to Dashboard Group or Dashboard Board. A record rule narrows what a reader sees; it never produces a message, so a record excluded by one of these is simply absent.

### 5.1 Audience

| Property | Value |
|---|---|
| External identifier | `ir_rule_spreadsheet_dashboard` |
| Label | "Spreadsheet dashboard: groups" |
| Entity | Spreadsheet Dashboard |
| Applies to | the internal-user group |
| Selection | the dashboard's access groups intersect the reader's own groups, direct and implied |

### 5.2 Company

| Property | Value |
|---|---|
| External identifier | `spreadsheet_dashboard_rule_company` |
| Label | "Dashboard multi-company" |
| Entity | Spreadsheet Dashboard |
| Applies to | every group, because the rule names none |
| Selection | the dashboard's companies intersect the reader's currently active companies, or the dashboard has no company at all |

Because the rule names no group, it is a **global** rule: it is joined by conjunction with whatever any group rule allows. A dashboard administrator is therefore still confined to the active companies.

### 5.3 Administrator widening

| Property | Value |
|---|---|
| External identifier | `spreadsheet_dashboard_rule_manager` |
| Label | "Spreadsheet dashboard: manager" |
| Entity | Spreadsheet Dashboard |
| Applies to | the dashboard administrator group |
| Selection | every record |

Group rules are joined by disjunction with each other. An administrator holds both the internal-user group and the administrator group, so the audience rule and this one are joined by disjunction and this one wins: every dashboard matches. The company rule of §5.2 still narrows the result, because it is global.

### 5.4 A share belongs to its creator

| Property | Value |
|---|---|
| External identifier | `spreadsheet_dashboard_share_create_uid_rule` |
| Label | "spreadsheet.dashboard.share: create uid" |
| Entity | Dashboard Share |
| Applies to | the internal-user group |
| Selection | the share's creating user is the reader |

Every internal user may create, read, update and delete shares; this rule confines each of them to their own. Reading another user's share — including reading its token — is refused with the platform's access refusal.

### 5.5 The evaluation order in one sentence

For Spreadsheet Dashboard: a reader sees a dashboard when ( the audience rule matches **or** the reader is a dashboard administrator ) **and** the company rule matches. For Dashboard Share: a reader sees a share when they created it. For Dashboard Group and Dashboard Board: no rule, so every record the access rights allow.

## 6. Actions and menus

### 6.1 The dashboard workspace action

| Property | Value |
|---|---|
| External identifier | `ir_actions_dashboard_action` |
| Kind | client action |
| Label | "Dashboards" |
| Address path | `dashboards` |
| Client handler | `action_spreadsheet_dashboard` |

The address path makes the workspace reachable directly, and the workspace writes the active dashboard's identifier into the address as it goes, under the key `dashboard_id`. The action accepts one parameter, `dashboard_id`, which selects the dashboard to open.

### 6.2 The dashboards configuration action

| Property | Value |
|---|---|
| External identifier | `spreadsheet_dashboard_action_configuration_dashboards` |
| Kind | window action |
| Label | "Dashboards" |
| Entity | Dashboard Group |
| Presentations | list, then form |

### 6.3 The personal board action

| Property | Value |
|---|---|
| External identifier | `open_board_my_dash_action` |
| Kind | window action |
| Label | "My Dashboard" |
| Entity | Dashboard Board |
| Presentations | form |
| Reading context | the toolbar is disabled, under the key `disable_toolbar` set to true |
| Usage | `usage` set to `menu`, which marks it as a menu landing rather than a record action |
| Presentation used | the shipped board form of §9.4 |

The action and its view are shipped with the no-update mark, so that an upgrade of the package does not overwrite an installation that changed them.

### 6.4 Menus

| External identifier | Label | Parent | Action | Sequence |
|---|---|---|---|---|
| `spreadsheet_dashboard_menu_root` | "Dashboards" | none; a root menu | the workspace action | 37 |
| `spreadsheet_dashboard_menu_dashboard` | "Dashboards" | the root | the workspace action | 1 |
| `menu_board_my_dash` | "My Dashboard" | the root | the personal board action | 100 |
| `spreadsheet_dashboard_menu_configuration` | "Configuration" | the root | none; a container | 150 |
| `spreadsheet_dashboard_menu_configuration_dashboards` | "Dashboards" | the configuration menu | the configuration action | 10 |

The root menu carries the application icon shipped with the dashboards package. The personal-board menu is contributed by the personal-boards package but hangs under the dashboards root, which is why the personal-boards package depends on the dashboards package rather than the reverse.

Because the root menu and its first child both point at the same action, a reader who activates the root lands on the workspace directly, and the child exists so that the workspace still has a menu entry of its own beside "My Dashboard" and "Configuration".

### 6.5 Menus and actions shipped by the conversation dashboards package

The conversation dashboards package ships five window actions and five menus so that its charts have something to link to. Every one of the five actions:

- is a window action over the conversation-channel entity, transport name `discuss.channel`;
- is labelled "Sessions";
- offers a list and a form, in that order, using the conversation package's own list and form presentations and its own search presentation;
- is restricted to the conversation manager group, external identifier `im_livechat.im_livechat_group_manager`;
- hangs under the conversation package's technical menu.

They differ only in the search filters they pre-select and in their menu labels:

| Action external identifier | Menu external identifier | Menu label | Pre-selected filters | Extra record selection |
|---|---|---|---|---|
| `ongoing_sessions_all_action` | `ongoing_session_all_menu` | "Ongoing Sessions" | ongoing | the channel kind is `livechat` |
| `ongoing_sessions_escalated_action` | `ongoing_sessions_escalated_menu` | "Escalated Sessions" | ongoing, escalated | none |
| `ongoing_sessions_agents_in_call_action` | `ongoing_sessions_agents_in_call_menu` | "Ongoing Call Sessions" | ongoing, in call | none |
| `ongoing_sessions_handle_by_agent_action` | `ongoing_sessions_handle_by_agent_menu` | "Sessions Handled by Agent" | ongoing, handled by an agent | none |
| `ongoing_sessions_handle_by_bot_action` | `ongoing_sessions_handle_by_bot_menu` | "Sessions Handled by Bot" | ongoing, handled by an automated responder | none |

These exist for this domain's sake — the ongoing-sessions dashboard links its charts to them — but the records they show belong to [`../messaging-and-activities/`](../messaging-and-activities/).

## 7. Shipped dashboard groups

Seven groups, all shipped by the dashboards package, all with an external identifier, and therefore all protected from deletion by rule [SD-009](business-rules.md#sd-009).

| External identifier | Name | Sequence |
|---|---|---|
| `spreadsheet_dashboard_group_sales` | "Sales" | 100 |
| `spreadsheet_dashboard_group_finance` | "Finance" | 300 |
| `spreadsheet_dashboard_group_logistics` | "Logistics" | 400 |
| `spreadsheet_dashboard_group_project` | "Services" | 500 |
| `spreadsheet_dashboard_group_marketing` | "Marketing" | 600 |
| `spreadsheet_dashboard_group_website` | "Website" | 700 |
| `spreadsheet_dashboard_group_hr` | "Human Resources" | 800 |

The sidebar order follows the sequence, so the shipped order is Sales, Finance, Logistics, Services, Marketing, Website, Human Resources. The gaps of one hundred leave room for an installation to insert its own groups between the shipped ones.

Two of the seven — "Marketing" and "Human Resources" — receive no dashboard from any package of this domain. They are shipped so that a package outside this domain has a group to file its dashboards under, and because a group with no published dashboard simply does not appear in the sidebar, an installation that adds nothing to them sees nothing.

## 8. Shipped dashboards

Fourteen dashboards shipped by twelve bridge packages, each installed only when both the dashboards package and the package the bridge depends on are installed.

### 8.1 The table

| External identifier | Name | Shipped by | Group | Sequence | Audience |
|---|---|---|---|---|---|
| `spreadsheet_dashboard_sales` | "Sales" | the sales bridge | Sales | 100 | `sales_team.group_sale_manager` |
| `spreadsheet_dashboard_product` | "Product" | the sales bridge | Sales | 200 | `sales_team.group_sale_manager` |
| `spreadsheet_dashboard_pos` | "Point of Sale" | the counter-sale bridge | Sales | 300 | `point_of_sale.group_pos_manager` |
| `spreadsheet_dashboard_pos_restaurant` | `POS - Restaurant` | the restaurant bridge | Sales | 350 | `point_of_sale.group_pos_manager` |
| `dashboard_invoicing` | "Invoicing" | the accounting bridge | Finance | 20 | `account.group_account_readonly` and `account.group_account_invoice` |
| `spreadsheet_dashboard_expense` | "Expenses" | the expense bridge | Finance | 40 | `hr_expense.group_hr_expense_manager` |
| `spreadsheet_dashboard_warehouse_metrics` | "Warehouse Metrics" | the warehouse bridge | Logistics | 300 | `stock.group_stock_manager` |
| `spreadsheet_dashboard_tasks` | "Project" | the project-time bridge | Services | 100 | `hr_timesheet.group_hr_timesheet_approver` |
| `spreadsheet_dashboard_timesheet` | "Timesheets" | the billable-time bridge | Services | 200 | `hr_timesheet.group_hr_timesheet_approver` |
| `spreadsheet_dashboard_events` | "Events" | the event bridge | Marketing | 60 | `event.group_event_manager` |
| `spreadsheet_dashboard_livechat` | "Live Chat" | the conversation bridge | Website | 100 | `im_livechat.im_livechat_group_manager` |
| `spreadsheet_dashboard_livechat_ongoing` | "Live Chat - Ongoing Sessions" | the conversation bridge | Website | 125 | `im_livechat.im_livechat_group_manager` |
| `spreadsheet_dashboard_ecommerce` | "eCommerce" | the storefront bridge | Website | 200 | `sales_team.group_sale_manager` |
| `spreadsheet_dashboard_elearning` | "eLearning" | the course bridge | Website | 200 | `website_slides.group_website_slides_manager` |

The name `POS - Restaurant` is reproduced as stored; it is the only shipped name that carries an abbreviation, and it is stored text rather than authored prose.

The table holds fourteen rows because the sales bridge and the conversation bridge ship two dashboards each; twelve packages ship them.

Every one of the fourteen is shipped published — `is_published` true — and every one carries no company, so every one is visible in every company until an administrator restricts it.

Two sequences collide: the storefront dashboard and the course dashboard are both 200 in the Website group. Order between two dashboards with the same sequence falls back to the identifier, so the one installed first is listed first. This is recorded as a **compatibility finding**: a corrected behaviour would give the two distinct sequences, so that the sidebar order does not depend on installation order.

### 8.2 Measured models and samples

Each dashboard may name the entities it measures. The reading route uses them to decide whether to show a sample instead of the real workbook; the rule is [`state-machines.md`](state-machines.md) §5.2.

| Dashboard | Measured entities | Sample document shipped |
|---|---|---|
| "Sales" | Sales Order, `sale.order` | yes |
| "Product" | Sales Order, `sale.order` | yes |
| "Point of Sale" | the counter-sale order report, `report.pos.order`, and Counter Sale Order, `pos.order` | yes |
| `POS - Restaurant` | Counter Sale Order, `pos.order` | yes |
| "Invoicing" | Journal Entry, `account.move` | yes |
| "Expenses" | Expense, `hr.expense` | yes |
| "Warehouse Metrics" | Quantity On Hand, `stock.quant` | yes |
| "Project" | none | no |
| "Timesheets" | Analytic Line, `account.analytic.line`, Project, `project.project`, and Sales Order, `sale.order` | yes |
| "Events" | Event, `event.event` | yes |
| "Live Chat" | the conversation report, `im_livechat.report.channel` | yes |
| "Live Chat - Ongoing Sessions" | the conversation report, `im_livechat.report.channel` | yes |
| "eCommerce" | Sales Order, `sale.order` | yes |
| "eLearning" | Sales Order, `sale.order` | yes |

The project dashboard is the one exception on both counts: it names no measured entity and ships no sample, so it always shows its real workbook, empty or not.

`sample_dashboard_file_path` holds the location of the sample document inside the capability package that ships it. The stored value is a path; the domain places no constraint on its shape beyond the file being readable and parsing as a structured document. A missing or unreadable file simply turns the sample off, without a message; see [`workflows.md`](workflows.md) §5 step 3.

### 8.3 What an administrator may change on a shipped dashboard

Everything except the external identifier: the name, the group, the sequence, the audience, the companies, the publication mark, the measured entities, the sample path and the workbook itself. A shipped dashboard is an ordinary record that happens to carry an external identifier.

The external identifier has exactly two consequences: the record is recognised on upgrade, and — for a Dashboard Group, not for a dashboard — deletion is refused by rule [SD-009](business-rules.md#sd-009). A shipped *dashboard* may be deleted; only a shipped *group* may not.

### 8.4 Adding a dashboard from another package

A package outside this domain ships a dashboard by creating one Spreadsheet Dashboard record with: a name, a dashboard group — one of the seven of §7, or its own — a workbook, an audience, a sequence, and optionally the measured entities and a sample path. Nothing else is required, and nothing in this domain has to be changed to accept it. The workbook may be supplied as an already-encoded file.

## 9. Views

Six views, plus one page template.

### 9.1 The dashboard configuration list

| Property | Value |
|---|---|
| External identifier | `spreadsheet_dashboard_view_list` |
| Entity | Spreadsheet Dashboard |
| Kind | list, editable at the bottom, with creation from the list disabled |

Columns, in order:

| Column | Control | Visible to |
|---|---|---|
| `sequence` — sequence | a drag handle | the technical-settings group only |
| `name` — name | plain text | everybody |
| `group_ids` — access groups | tags, marked required | everybody |
| `company_ids` — companies | tags, creation of a company from the control forbidden, placeholder "Visible to all" | the multiple-companies group only |
| `spreadsheet_binary_data` — the workbook, labelled "Data" | the workbook file control, using `spreadsheet_file_name` as the download name | the extended-visibility group only |
| `is_published` — publication | a toggle | everybody |
| `dashboard_group_id` — dashboard group | plain reference, optional and hidden by default | everybody |

Creation from the list is disabled because this list is embedded in the group form, where a row's group is implied by the form.

### 9.2 The dashboard form

| Property | Value |
|---|---|
| External identifier | `spreadsheet_dashboard_view_form` |
| Entity | Spreadsheet Dashboard |
| Kind | form |

One group of fields: the name, the dashboard group, the companies (tags, no creation, shown to the multiple-companies group only, placeholder "Visible to all"), the access groups (tags), and the workbook.

### 9.3 The dashboard card presentation

| Property | Value |
|---|---|
| External identifier | `spreadsheet_dashboard_view_kanban` |
| Entity | Spreadsheet Dashboard |
| Kind | card |
| Content | one card per dashboard showing its name, laid out for a small screen |

### 9.4 The group list and form

| External identifier | Entity | Kind | Content |
|---|---|---|---|
| `spreadsheet_dashboard_container_view_list` | Dashboard Group | list, titled "Dashboards" | a drag handle for the sequence, shown to the technical-settings group only, then the name |
| `spreadsheet_dashboard_container_view_form` | Dashboard Group | form | the name as the heading, then one page named "Spreadsheets" holding the group's dashboards through the list of §9.1 |

### 9.5 The personal board form and the public page

| External identifier | Entity | Kind | Content |
|---|---|---|---|
| `board_my_dash_view` | Dashboard Board | form, titled "My Dashboard" | a board element whose layout is `2-1`, holding one empty column |
| `spreadsheet.public_spreadsheet_layout` | none; a page template | page | The public page of a shared dashboard: the dashboard's name, the sentence "Frozen and copied on" followed by the share's creation moment, a download control when one was supplied, the portal identity controls, and the mount point for the workbook |

The five layouts a reader may choose for the personal board are the reproduced values `1`, `1-1`, `1-1-1`, `1-2` and `2-1`; the shipped one is `2-1`, meaning two columns of which the first is twice the width of the second.

## 10. The session capability flag

The domain contributes one key to the description a client receives when it starts a session:

| Key | Value shipped by this domain | Meaning |
|---|---|---|
| `can_insert_in_spreadsheet` | false | Whether the client offers the control that inserts the current view into a workbook |

The key is always present once the workbook engine is installed, and this domain always sets it to false. It exists as an extension point: a package that adds workbook authoring sets it to true, and every client control that offers insertion is drawn only when it is true. A rebuild must ship the key with the value false, because a client that does not find the key at all behaves differently from one that finds it false.

## 11. Settings and system parameters

**None.** The domain has no settings page, contributes no field to any settings entity, and reads no system parameter.

Everything an administrator can change is a record: a dashboard, a dashboard group, a group membership, a company. The design decision behind this is worth recording, because a rebuild is likely to want a setting: the audience of a dashboard is per dashboard, not per installation, so there is nothing installation-wide to set.

The one installation-wide value the domain depends on is the base web address, which it reads when it composes a shared address (see [`entities.md`](entities.md) §5.3). That value belongs to the platform, not to this domain.

## 12. Sequences

**None.** No entity of the domain carries a generated reference.

- A Spreadsheet Dashboard is identified by its name and ordered by its `sequence` integer, which is a position, not a number drawn from a sequence.
- A Dashboard Group likewise.
- A Dashboard Share is identified by its record identifier and its token; the token is a freshly generated universally unique identifier, not a sequence value.

## 13. Scheduled jobs

**None.** Nothing in this domain runs on a timer.

The consequences, each of which a rebuild must reproduce:

| Expectation a reader might have | What actually happens |
|---|---|
| Dashboards are refreshed on a schedule | They are not. A dashboard's data is fetched when a reader opens it, and again when the reader asks for a refresh of all data |
| Shares expire | They do not. A Dashboard Share lives until its creator deletes it or its dashboard is deleted. Its reachability can change without the record changing, by the sharing user's rights changing; see [`state-machines.md`](state-machines.md) §8 |
| Old shares are cleaned up | They are not |
| The export log is rotated | It is not by this domain; the entries go to the application log and their retention is a platform matter |

## 14. Message templates, activity types, subtypes

**None of the three.** No entity of this domain carries a message thread, an activity or a follower list, so there is nothing for a template, an activity type or a subtype to attach to. Sharing a dashboard produces an address, not a message: the reader copies it and sends it by whatever means they choose. Nothing in this domain sends mail.

## 15. Default values that apply without configuration

| Entity and field | Default | Where it is specified |
|---|---|---|
| Spreadsheet Dashboard, `group_ids` | the internal-user group | [`entities.md`](entities.md) §3.2 |
| Spreadsheet Dashboard, `is_published` | true | [`state-machines.md`](state-machines.md) §2.1 |
| Spreadsheet Dashboard, `company_ids` | empty, which means every company | [`entities.md`](entities.md) §3.6 |
| Spreadsheet Dashboard, `sequence` | zero | [`entities.md`](entities.md) §3.4 |
| Any record carrying the mixin, `spreadsheet_binary_data` | the empty workbook | [`document-format.md`](document-format.md) §6 |
| Dashboard Share, `access_token` | a freshly generated universally unique identifier | [`entities.md`](entities.md) §5.2 |
| Spreadsheet Dashboard, `favorite_user_ids` | empty; the control that edits it offers only the acting reader | [`entities.md`](entities.md) §3.2 |

## 16. Extension points an installation may use

| Point | How a package uses it |
|---|---|
| A new dashboard | Ship a Spreadsheet Dashboard record; §8.4 |
| A new dashboard group | Ship a Dashboard Group record; it will be protected from deletion by rule [SD-009](business-rules.md#sd-009) |
| A new workbook-carrying entity | Declare the Spreadsheet Document mixin on it; the entity gains the four fields, the validation, the empty-workbook default and the file-name rule of [`entities.md`](entities.md) §2 |
| A control beside each dashboard in the sidebar | Register a component in the dashboard-action registry; the workspace draws the first registered one beside every dashboard entry and hands it the dashboard's identifier, its data and a callback that opens the dashboard for editing |
| Workbook authoring | Set the session capability flag of §10 to true and provide the authoring action; every insertion control in every client becomes visible |
| A new business formula | Register the formula under the category `Odoo`, which is the reproduced category name under which every formula of this domain is filed, so that it appears with the others in the formula list |
| A new data-bound chart kind | Register a chart kind whose stored value begins with `odoo_`; the freeze algorithm turns any such chart into a picture without further work |
