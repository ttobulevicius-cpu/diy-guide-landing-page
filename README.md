# aivertises.com — Waitlist Landing Page

## Files

| File | Purpose |
|------|---------|
| `index.html` | Landing page with email capture form |
| `thank-you.html` | Confirmation + pre-order upsell page |
| `README.md` | This file |

---

## n8n Workflow

**Workflow name:** Waitlist Email Collection
**Workflow ID:** `Fd6y2K7StX51LmnU`
**Status:** Active
**Webhook URL:** `https://n8n.34.68.150.87.nip.io/webhook/waitlist`

When a visitor submits the form on `index.html`, the data is posted to this webhook. The workflow:
1. Receives the POST (email, name, source)
2. Normalizes the data and adds a timestamp
3. Appends a new row to the Google Sheet (see below)

**Google Sheet:** `1zZLs85nEHkC565iKzSSPY1a2Mc8ogUMjS0FAJy5lsME`
Make sure row 1 of the sheet has these exact headers: `Email | Name | Timestamp | Source`

---

## Deployment to aivertises.com

### Option A — cPanel / File Manager
1. Log in to your web hosting control panel
2. Open File Manager → navigate to your `public_html` folder
3. Upload `index.html` and `thank-you.html`
4. Visit `https://aivertises.com` to confirm the landing page loads

### Option B — FTP
```bash
ftp your-server.com
# Upload both HTML files to /public_html/
```

### Option C — Git / CI
If your host supports git deployment, commit both files and push.

**Note:** No server-side code is needed. Both files are pure HTML and can be hosted anywhere (Netlify, Vercel, GitHub Pages, shared hosting, etc.).

---

## Setting Up Stripe Payment Link

1. Log in to [https://dashboard.stripe.com](https://dashboard.stripe.com)
2. Go to **Products** → **Add product**
   - Name: `AI Phone Receptionist — Pre-Order`
   - Price: `€99` (one-time)
3. Go to **Payment Links** → **New** → select your product
4. Configure as needed (collect name/email, add your terms URL)
5. Click **Create link** — copy the URL (format: `https://buy.stripe.com/5kQeVfgsr6EIeBe6Uhcwg01`)

---

## Replacing the Stripe Placeholder

In `thank-you.html`, find this line:

```html
<a href="https://buy.stripe.com/5kQeVfgsr6EIeBe6Uhcwg01" class="btn-preorder">
```

Replace `https://buy.stripe.com/5kQeVfgsr6EIeBe6Uhcwg01` with your real Stripe Payment Link URL.

---

## Testing

### Quick test with curl (no browser needed)

```bash
curl -X POST https://n8n.34.68.150.87.nip.io/webhook/waitlist \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","name":"Test User","source":"aivertises.com"}'
```

Expected: HTTP 200 response. Then check the Google Sheet — a new row should appear.

### Full browser test

1. Open `index.html` in Chrome
2. Open DevTools → Network tab
3. Fill in the email form and submit
4. Confirm: the POST returns 200, no CORS errors in Console, page redirects to `thank-you.html`
5. Confirm: the email appears in the Google Sheet

### Edge case tests

- Submit with an invalid email (e.g. `notanemail`) — should show inline error, not submit
- Submit the form twice quickly — second click is blocked by the loading state
- View both pages on a mobile device at 375px width — no horizontal scroll

---

## Troubleshooting

**Form submits but Google Sheet has no new row**
- Check n8n Executions for the workflow (open n8n → Workflows → Waitlist Email Collection → Executions)
- Look for failed executions and inspect the error

**CORS error in browser console**
- The webhook node must have `allowedOrigins: *` configured
- This is already set in the workflow — if it reappears, check that the workflow is still active

**Webhook returns 404**
- Workflow may have been deactivated — log in to n8n and re-activate it
- Test mode URL (`/webhook-test/...`) only works when the canvas is open — use the production URL

**Redirect doesn't happen after submit**
- Open DevTools → Console for JavaScript errors
- Verify the webhook URL in `index.html` is `https://n8n.34.68.150.87.nip.io/webhook/waitlist`
