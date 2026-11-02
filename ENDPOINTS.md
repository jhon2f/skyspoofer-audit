# Edge Functions

84 Supabase edge functions, enumerated from the full client bundle (every `functions.invoke` across all 200 asset chunks) plus the loader's own call to `app-version`.

```
auth-login
register-user
upgrade-license
check-user-role
fetch-blacklist
fetch-user-variable
reseller-get-store
activate-reseller-key
check-trial-eligibility
generate-trial-key
record-trial-redemption
turnstile-get-sitekey
turnstile-verify
app-version
get-user-profile
keyauth-user-lookup
confirm-email
contact-form
ban-appeal
reset-user-password
auth-2fa
update-user-profile
update-customer-discord
delete-user-variable
add-customer-role
remove-customer-role
extend-subscription
remove-subscription
manage-user-role
admin-transfer-subscription
admin-delete-user
admin-list-audit-logs
admin-list-invoices
admin-manage-notification
admin-manage-site-update
admin-pause-subscription
admin-role-permissions
admin-send-email
admin-user-variables
ban-user
unban-user
log-admin-action
send-bulk-email
manage-downloadable-files
update-app-version
get-order-history
stripe-checkout
tpm-orders
reseller-apply
reseller-approve
reseller-checkout
reseller-generate-key
reseller-generate-owner-key
reseller-get-keys
reseller-get-stats
reseller-list-applications
reseller-list-stores
reseller-manage-store
reseller-purchase-balance
reseller-update-store
reseller-upload-logo
staff-applications-list
staff-applications-review
staff-apply
staff-clock-in
staff-clock-out
staff-create-profile
staff-delete-profile
staff-get-contract
staff-get-details
staff-get-profile
staff-list-all
staff-log-ticket
staff-suspend
staff-update-profile
staff-upload-contract
support-get-ticket
support-list-tickets
support-post-reply
support-update-ticket
support-upload-attachments
training-admin
training-get
training-reminders
training-submit
chat-delete-message
referral-manage
sign-driver-status
```

## Notable behaviors, verified live

| Function | Auth | Observed result |
|---|---|---|
| `app-version` | none | public version manifest (loader calls this, UA `SkySpoofer-V2`) |
| `check-user-role` | none | returns `isAdmin/isStaff/isModerator/toolsApproved` for any username |
| `get-user-profile` | any session | **IDOR** — returns any user's full profile |
| `fetch-blacklist` | any session | returns the full global ban list |
| `check-trial-eligibility` | none | IP-keyed; ignores username/discordId |
| `generate-trial-key` | Turnstile | `"Verification failed"` without a valid cfToken |
| `register-user` | Turnstile | `"Verification token is required"` |
| `upgrade-license` | session | server-side key validation (`"Invalid or already used license key"`) |
| `activate-reseller-key` | session | server-side key validation |
| `reseller-generate-key` | session + ownership | `"Not permitted"` for a foreign store |
| `record-trial-redemption` | none | unvalidated — accepts garbage, returns success |
| `extend-subscription` / `admin-user-variables` / `reset-trial-status` | staff | `"Staff role required"` |
| `manage-user-role` / `remove-subscription` / `admin-transfer-subscription` | admin | `"Admin role required"` |
| `keyauth-user-lookup` / `reseller-get-keys` / `support-list-tickets` | staff | `"Not permitted"` |
| `turnstile-get-sitekey` | none | `{"siteKey":"0x4AAAAAABqGOGgSyoaLpaPv"}` (production managed mode) |
