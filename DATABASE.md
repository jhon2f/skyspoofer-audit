# Database

Supabase PostgREST surface for project `dvybxmoxtmqbczgpqgmc`, reachable with the anon key.

## Tables — 17 total

Three are publicly readable. The other fourteen return `[]` to the anon key (RLS filters every row).

```
PUBLIC (readable):
  reseller_stores      full reseller roster + owner keys + business data
  site_updates         release history
  role_permissions     the RBAC map (59 rows)

RLS-LOCKED (return []):
  app_users            accounts
  user_subscriptions   subscriptions
  license_keys         license keys
  reseller_keys        reseller keys
  reseller_applications
  user_roles           admin/staff/moderator assignments
  referral_codes
  trial_redemptions
  downloadable_files
  support_tickets
  staff_profiles
  user_profiles        PII (readable only via the get-user-profile IDOR, see EXPLOITS.md)
  user_variables
  chat_messages
```

Write access: **none.** INSERT against all 17 tables returns 401. PATCH against `user_profiles` returns zero affected rows — the RLS `USING` clause filters every row, so nothing is ever written through PostgREST with the anon key.

## license_keys schema

Enumerated through PostgREST error messages (column-not-found hints and enum rejections), without ever reading a row:

```
id, key, status, note, created_at, duration_days, used_at, used_by, subscription_name
```

- `status` is an enum, `license_key_status`, with a valid value of `"unused"` (`"available"` is rejected).
- `note` likely holds the source/tier tag.
- `duration_days` + `subscription_name` determine what a key grants.

## role_permissions — the full RBAC map (public)

```
staff (10):
  tools.lookup, tools.subscriptions, tools.hwid, tools.trials, tools.password,
  tools.bans, tools.staff_suspend, tools.verify_codes, chat.moderate, admin.panel

moderator (12):
  staff set + tools.pause_subscription, admin.university

admin (37):
  moderator set + admin.roles, admin.permissions, admin.staff_management,
  admin.staff_applications, admin.university, admin.resellers, admin.referrals,
  admin.invoices, admin.bulk_grant, admin.files, admin.email_broadcast,
  admin.direct_email, admin.notifications, admin.site_updates, admin.spoofing_modes,
  admin.app_version, admin.audit_log, admin.driver_signing,
  tools.transfer_subscription, tools.delete_user, tools.user_variables,
  support.tickets_view, support.tickets_reply, support.tickets_manage,
  support.canned_responses, support.knowledge_base
```

This maps the internal permission taxonomy — every permission key that exists, and which role holds it — straight from a public table.

## site_updates (public)

```
v4.5.3  2026-10-20  kernel offsets from database, reworked data patching, HVCI/VBS beta
v4.5.2  2026-10-01  Windows version support, driver/loader compat
v4.5.0  2026-08-20  VBS spoofing, direct memory patching
v4.0.0  2026-07-28  ACPI spoofing + redesign
```
