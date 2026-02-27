# WordPress + Meta Pixel CAPI Setup Guide
## IPL Landing Page Deployment

---

## Step 1 — Replace Placeholders in index.html

Before deploying, find and replace these 3 values in `index.html`:

| Placeholder | Replace With | Where |
|---|---|---|
| `YOUR_PIXEL_ID` | Your Meta Pixel ID (e.g. `1234567890123456`) | 2 places in `<head>` |
| `https://t.me/YOUR_TELEGRAM_GROUP` | Your Telegram invite/channel link | `href` on the button |
| `ipl-players.png` | Your actual image filename | `<img src="">` |

---

## Step 2 — Upload Your IPL Image

1. Go to WordPress Admin → Media → Add New
2. Upload your IPL players image
3. Click the uploaded image → copy the filename (e.g. `ipl-players-2025.png`)
4. Update `index.html` → replace `ipl-players.png` with the exact filename
5. Note the full URL for later (e.g. `https://yoursite.com/wp-content/uploads/2025/02/ipl-players-2025.png`)
6. In `index.html`, update the `<img src="">` to use the full WordPress media URL

---

## Step 3 — Deploy to WordPress

Choose **one** of these 3 methods:

---

### Method A: Direct HTML Upload (Simplest — Recommended)

This bypasses WordPress entirely for the page itself — cleanest and fastest.

1. Connect to your server via FTP/cPanel File Manager
2. Navigate to `public_html/` (or your domain root)
3. Create a folder: `public_html/ipl/`
4. Upload `index.html` into it → your page is now live at `https://yoursite.com/ipl/`
5. PixelYourSite still runs on WordPress and sends CAPI events for all page visits

**Why this is best:** Zero WordPress overhead = faster page load = better Meta ad Quality Score.

---

### Method B: Elementor Full-Width Page

If you want the page managed inside WordPress:

1. WordPress Admin → Pages → Add New
2. Title: "IPL Landing Page" (won't be shown on page)
3. Click "Edit with Elementor"
4. In Elementor: drag a **"HTML" widget** onto the canvas
5. Paste the ENTIRE contents of `index.html` (including `<html>`, `<head>`, `<body>` tags)
   - Elementor will strip the outer tags automatically — this is fine
6. For the Meta Pixel `<head>` code: see Step 4 (WPCode plugin)
7. Set the page template to "Elementor Full Width" or "Canvas" (no header/footer)
8. Publish the page

---

### Method C: Custom PHP Page Template (Advanced)

1. In your theme folder (`wp-content/themes/your-theme/`), create: `page-ipl.php`
2. Paste this at the very top:
   ```php
   <?php
   /**
    * Template Name: IPL Landing Page
    */
   ?>
   ```
3. Below that, paste the full contents of `index.html`
4. In WordPress Admin → Pages → Add New → set Page Template to "IPL Landing Page"
5. Publish

---

## Step 4 — Add Meta Pixel to WordPress Head (for Methods B & C)

The Meta Pixel `<script>` in `index.html` works when you use Method A (direct HTML).

For Methods B & C, add the pixel via **WPCode plugin** (free):

1. Install: Plugins → Add New → search "WPCode Insert Headers and Footers" → Install → Activate
2. Go to: Code Snippets → Headers & Footers
3. In the "Header" box, paste this (with your real Pixel ID):

```html
<!-- Meta Pixel Code -->
<script>
  !function(f,b,e,v,n,t,s){
    if(f.fbq)return;
    n=f.fbq=function(){n.callMethod?
    n.callMethod.apply(n,arguments):n.queue.push(arguments)};
    if(!f._fbq)f._fbq=n;
    n.push=n;n.loaded=!0;n.version='2.0';
    n.queue=[];t=b.createElement(e);t.async=!0;
    t.src=v;s=b.getElementsByTagName(e)[0];
    s.parentNode.insertBefore(t,s)
  }(window,document,'script','https://connect.facebook.net/en_US/fbevents.js');
  fbq('init','YOUR_PIXEL_ID');
</script>
<noscript>
  <img height="1" width="1" style="display:none"
    src="https://www.facebook.com/tr?id=YOUR_PIXEL_ID&ev=PageView&noscript=1"/>
</noscript>
<!-- End Meta Pixel Code -->
```

4. Click Save

---

## Step 5 — Install PixelYourSite Pro (for CAPI)

> **Why CAPI?** Browser pixel alone tracks ~60-70% of users due to iOS 14+ and ad blockers. Browser pixel + CAPI together = ~90-95% tracking accuracy.

1. Purchase PixelYourSite Pro from pixelyoursite.com
2. WordPress Admin → Plugins → Add New → Upload Plugin → upload the `.zip` file → Install → Activate

---

## Step 6 — Connect Your Pixel in PixelYourSite

1. Go to: PixelYourSite → Facebook Pixel
2. Enter your **Pixel ID** in the first field
3. Click **Save Settings**

---

## Step 7 — Generate Meta System User Access Token

This token lets PixelYourSite send server-side events to Meta.

1. Go to [business.facebook.com](https://business.facebook.com) → Settings (gear icon)
2. Left sidebar → Users → **System Users**
3. Click **Add** → name it "WordPress CAPI" → Role: **Standard** → Create System User
4. Click the system user → click **Generate New Token**
5. Select your **Ad Account** and your **Pixel**
6. Required permissions:
   - `ads_management`
   - `ads_read`
   - `pages_read_engagement` (optional but recommended)
7. Click **Generate Token**
8. **Copy the token immediately** — it won't be shown again

---

## Step 8 — Enable CAPI in PixelYourSite

1. PixelYourSite → Facebook Pixel → **Conversions API** tab
2. Toggle **"Enable Conversions API"** → ON
3. Paste your Access Token from Step 7
4. Click **Save Settings**

---

## Step 9 — Enable Event ID Deduplication

This ensures that when both the browser pixel AND the server send the same event (e.g. `Lead`), Meta counts it as ONE conversion — not two.

1. PixelYourSite → **Advanced** tab
2. Find **"Event ID / Deduplication"** section
3. Enable it → Save

The landing page's JS already generates a UUID `eventID` for each `PageView` and `Lead` event and stores it in `sessionStorage`. PixelYourSite reads this and sends the matching ID server-side.

---

## Step 10 — Test Everything

### Browser Pixel Test:
1. Go to [business.facebook.com](https://business.facebook.com) → Events Manager → your Pixel → **Test Events** tab
2. Enter your landing page URL → click **"Open Website"**
3. The Event Manager should show:
   - `PageView` fires immediately on page load
   - `Lead` fires when you click JOIN TELEGRAM
4. Check the **green checkmark** next to each event

### CAPI Test:
1. In Events Manager → same **Test Events** tab
2. Switch to **"Server Events"** view
3. After clicking the Telegram button, you should see a `Lead` event arrive within 3-5 seconds
4. The event will show **"Deduplication: Yes"** or similar if deduplication is working

### Full Pixel Health Check:
1. Events Manager → your Pixel → **Overview** tab
2. Check **"Event Match Quality"** — aim for 6.0+ score
3. Check **"Redundancy"** — should show 0% or near-zero if deduplication is working

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Pixel not firing | Check Pixel ID is correct, no typos, no spaces |
| CAPI events not arriving | Check Access Token is valid (regenerate if expired) |
| Double-counting conversions | Verify deduplication is ON in PixelYourSite Advanced settings |
| Image not showing | Check image URL — use the full WordPress media URL, not just filename |
| Page loads slowly | Use Method A (direct HTML upload) instead of Elementor |
| Telegram button not working | Replace `YOUR_TELEGRAM_GROUP` with your actual t.me link |

---

## Final Checklist Before Running Ads

- [ ] `YOUR_PIXEL_ID` replaced with real Pixel ID (both in `<head>` script and `noscript` img)
- [ ] Telegram link updated to your real group link
- [ ] IPL image uploaded and URL updated in the HTML
- [ ] Page loads on mobile (test on real phone)
- [ ] PageView event confirmed in Meta Events Manager → Test Events
- [ ] Lead event fires on Telegram button click
- [ ] CAPI events arrive in Server Events tab
- [ ] Deduplication confirmed (no double counting)
- [ ] PixelYourSite → Event Match Quality shows 6.0+ score
