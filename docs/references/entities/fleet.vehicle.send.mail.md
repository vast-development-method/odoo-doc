# Send mails to Drivers (`fleet.vehicle.send.mail`)

**Transport name:** `fleet.vehicle.send.mail`  
**Storage name:** `fleet_vehicle_send_mail`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `fleet`

Description: Send mails to Drivers

## Identity and behavior

- Mixins (classical inheritance): `mail.composer.mixin`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `vehicle_ids` | Vehicles | many to many | `fleet.vehicle` | required |
| `author_id` | Author | many to one | `res.partner` | required; default computed dynamically (lambda self: self.env.user.partner_id.id) |
| `template_id` | Template | many to one |  | restricted by domain `lambda self: [('model_id', '=', self.env['ir.model']._get('fleet.vehicle').id)]` |
| `attachment_ids` | Attachments | many to many | `ir.attachment` | association table `fleet_vehicle_mail_compose_message_ir_attachments_rel` |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_render_model` | computation | self | `fleet` | depends: `subject` |  |
| `_onchange_template_id` | on change | self | `fleet` | onchange: `template_id` |  |
| `action_send` | user action | self | `fleet` |  |  |
| `action_save_as_template` | user action | self | `fleet` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `fleet_group_manager` | yes | yes | yes | no | `fleet` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_send_mail_view_form` | form |  | `subject`, `body`, `attachment_ids`, `template_id` | `Send`, `Cancel`, `Save as new template` |  | `fleet` |

Machine-readable definition: `../../../schemas/data/entities/fleet.vehicle.send.mail.json`; views: `../../../schemas/interfaces/views/fleet.vehicle.send.mail.json`.
