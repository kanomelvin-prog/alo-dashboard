# Alo Therapist Dashboard

Clinical dashboard for therapists to monitor client activity with Alo.

**Live at:** (your dashboard URL)

---

## Current Version

`index.html` - Working dashboard with:
- Client list with status indicators
- Alerts for crisis conversations
- Triage for clients needing attention
- Individual client activity view
- Homework assignment system
- Session notes with EHR export
- Supabase integration

---

## Version History

### v1.0.0 Baseline (Dec 29, 2025)
- **File:** `versions/baseline_20251229.html`
- Initial stable version
- Features:
  - Client management (add/edit/remove)
  - Crisis detection & alerts
  - Engagement trend monitoring
  - Homework tracking
  - Session notes
  - 3-tab client view (Activity, Homework, Notes)

---

## Supabase Configuration

**URL:** `https://wekvrjnbjpbvvpfuwzvz.supabase.co`

**Tables Used:**
- `therapist_client_links` - Client roster
- `conversation_turns` - Alo chat history
- `homework` - Assigned exercises
- `session_notes` - Clinical documentation

**Therapist ID:** `THERAPIST_001` (hardcoded for demo)

---

## Making Changes

**Before editing `index.html`:**

1. **Create backup:**
   - Go to "Add file" → "Create new file"
   - Name: `versions/backup_YYYYMMDD_HHMM.html`
   - Paste current index.html code
   - Commit

2. **Make your changes to `index.html`**

3. **If it works:**
   - Create: `versions/feature-name_working_YYYYMMDD.html`
   - Copy the working code
   - Commit

4. **If it breaks:**
   - Go to `versions/baseline_20251229.html`
   - Copy the code
   - Replace `index.html`
   - Commit with message: "Rollback to baseline"

---

## Known Issues

None currently - this is the baseline stable version.

---

## Planned Improvements

(Track these in a separate `IMPROVEMENTS.md` file or in issues)

---

## Testing

1. **Add a client:** Click "+ Add Client"
2. **View client:** Click client name in sidebar
3. **Assign homework:** Go to Homework tab
4. **Create session note:** Go to Session Notes tab
5. **Check alerts:** Crisis conversations trigger red alerts

---

## Support

For issues or questions, contact Kano.
