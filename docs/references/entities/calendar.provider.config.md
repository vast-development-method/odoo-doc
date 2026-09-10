# Calendar Provider Configuration Wizard (`calendar.provider.config`)

**Transport name:** `calendar.provider.config`  
**Storage name:** `calendar_provider_config`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `calendar`

Description: Calendar Provider Configuration Wizard

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `external_calendar_provider` | Choose an external calendar to configure | selection |  | default `google` |
| `cal_client_id` | Google Client_id | single line text |  | default computed dynamically (lambda self: self.env['ir.config_parameter'].get_param('google_calendar_client_id')) |
| `cal_client_secret` | Google Client_key | single line text |  | default computed dynamically (lambda self: self.env['ir.config_parameter'].get_param('google_calendar_client_secret')) |
| `cal_sync_paused` | Google Synchronization Paused | boolean |  | default computed dynamically (lambda self: str2bool(self.env['ir.config_parameter'].get_param('google_calendar_sync_paused'), default=False)) |
| `microsoft_outlook_client_identifier` | Outlook Client Id | single line text |  | default computed dynamically (lambda self: self.env['ir.config_parameter'].get_param('microsoft_calendar_client_id')) |
| `microsoft_outlook_client_secret` | Outlook Client Secret | single line text |  | default computed dynamically (lambda self: self.env['ir.config_parameter'].get_param('microsoft_calendar_client_secret')) |
| `microsoft_outlook_sync_paused` | Outlook Synchronization Paused | boolean |  | default computed dynamically (lambda self: str2bool(self.env['ir.config_parameter'].get_param('microsoft_calendar_sync_paused'), default=False)) |

## Selection values

### `external_calendar_provider` (Choose an external calendar to configure)

| Value | Label |
|---|---|
| `google` | Google |
| `microsoft` | Outlook |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_calendar_prepare_external_provider_sync` | user action | self | `calendar` |  | Called by the wizard to configure an external calendar provider without requiring users to access the general settings page. Make sure that the provider calendar module is installed or install it. Then, set the API keys into the applicable config parameters. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `calendar` |
| `base.group_system` | yes | yes | yes | yes | `calendar` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `calendar.calendar_provider_config_view_form` | form |  | `external_calendar_provider`, `cal_client_id`, `cal_client_secret`, `cal_sync_paused`, `microsoft_outlook_client_identifier`, `microsoft_outlook_client_secret`, `microsoft_outlook_sync_paused` | `Cancel` |  | `calendar` |

Machine-readable definition: `../../../schemas/data/entities/calendar.provider.config.json`; views: `../../../schemas/interfaces/views/calendar.provider.config.json`.
