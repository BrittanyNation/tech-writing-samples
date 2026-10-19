# Troubleshooting Common Email Rendering Issues

An email that looks perfect in one inbox arrives broken in another. That is normal: every email client renders HTML differently, and a handful of failures account for most of the damage. This guide covers the five most common ones.

Each issue follows the same format: the **problem**, what you'll actually **see**, the **cause**, and the **fix**. Work through the fixes in order; most broken emails are fixed by the first one or two.

## Quick reference

| Symptom | Likely cause | Go to |
| --- | --- | --- |
| Layout stretched or extra space in Outlook | Missing Outlook fallbacks and default table spacing | [Issue 1](#issue-1-outlook-adds-spacing-and-stretches-the-layout) |
| Colors inverted or unreadable in dark mode | Client auto-inversion with no dark mode styles declared | [Issue 2](#issue-2-dark-mode-inverts-or-washes-out-colors) |
| Footer or bottom content missing in Gmail | HTML over Gmail's ~102KB clipping limit | [Issue 3](#issue-3-gmail-clips-the-message) |
| Columns squeezed instead of stacking on phones | No media query, or a client that ignores them | [Issue 4](#issue-4-columns-dont-stack-on-mobile) |
| Email looks empty or content is missing | Images blocked by default and no fallback text | [Issue 5](#issue-5-images-are-blocked) |

---

## Issue 1: Outlook adds spacing and stretches the layout

**Problem**
The email renders correctly in Gmail and Apple Mail, but in Outlook on Windows the layout is stretched full width, gaps appear between sections, or images are the wrong size.

**Symptom**

- The email fills the whole preview pane instead of sitting in a centered 600px column.
- Thin white gaps appear between stacked images or table rows.
- Images display larger or smaller than designed.

**Cause**
Outlook on Windows renders email with Microsoft Word's engine, not a web browser engine. That engine ignores `max-width` and most modern CSS, applies its own defaults to table spacing, and scales images using its own DPI assumptions. An email built only to web standards will break there first.

**Fix**

1. **Hold the width with a ghost table.** Wrap your container in MSO conditional comments that only Outlook reads:

   ```html
   <!--[if mso]>
   <table role="presentation" width="600" cellpadding="0" cellspacing="0" border="0" align="center"><tr><td>
   <![endif]-->
     <!-- your container table -->
   <!--[if mso]>
   </td></tr></table>
   <![endif]-->
   ```

2. **Zero out table spacing everywhere.** Every layout table needs `cellpadding="0"`, `cellspacing="0"`, and `border="0"`, plus `border-collapse: collapse` in its inline style if gaps persist.
3. **Set explicit image dimensions.** Give every image accurate `width` and `height` attributes. Outlook lays out using these attributes, not CSS.
4. **Add the DPI setting.** Include the OfficeDocumentSettings block in the head so Outlook renders images at 96 DPI instead of rescaling them:

   ```html
   <!--[if mso]>
   <noscript><xml><o:OfficeDocumentSettings><o:PixelsPerInch>96</o:PixelsPerInch></o:OfficeDocumentSettings></xml></noscript>
   <![endif]-->
   ```

5. Retest in a real Outlook install or a rendering tool (Litmus, Email on Acid) before sending.

---

## Issue 2: Dark mode inverts or washes out colors

**Problem**
The email is readable in light mode, but in dark mode the colors are inverted, the logo disappears, or text becomes unreadable against its background.

**Symptom**

- Light backgrounds turn dark while text stays dark, making copy invisible.
- A dark logo vanishes against a newly dark background.
- Brand colors shift to unintended shades.

**Cause**
Many clients (Gmail's apps, Outlook's apps, and others) automatically invert colors in emails that don't declare dark mode support. Each client inverts differently: some do a full inversion, some a partial one, and some leave the email alone, which is why the same email can look fine in one dark inbox and broken in another.

**Fix**

1. **Declare support so clients stop guessing.** Add both meta tags to the head:

   ```html
   <meta name="color-scheme" content="light dark">
   <meta name="supported-color-schemes" content="light dark">
   ```

2. **Write explicit dark styles.** In a `<style>` block, define how the email should look in dark mode for clients that honor media queries:

   ```css
   @media (prefers-color-scheme: dark) {
     .body-bg { background-color: #1a1a1a !important; }
     .text { color: #f2f2f2 !important; }
   }
   ```

3. **Handle Outlook's dark mode separately.** Outlook apps use their own override attributes instead of the media query. Add rules using `[data-ogsc]` (text color) and `[data-ogsb]` (background color) selectors for the elements that matter most.
4. **Protect images that must not change.** Give logos a light background by design (a logo on a white rounded rectangle looks correct in both modes), or outline transparent images so they stay visible. Never ship a dark, transparent logo with no fallback.
5. **Test in at least two dark clients.** Apple Mail and one Outlook app at minimum. Their dark modes behave differently, and one passing proves nothing about the other.

---

## Issue 3: Gmail clips the message

**Problem**
The bottom of the email is missing in Gmail. The footer, the unsubscribe link, and sometimes entire final sections don't appear. Instead, Gmail shows a "[Message clipped] View entire message" notice.

**Symptom**

- Content ends abruptly partway through the email.
- The unsubscribe link is missing, which is also a compliance risk.
- The email looks complete in other clients.

**Cause**
Gmail clips any message whose HTML exceeds roughly 102KB. Everything after the limit is hidden behind the "view entire message" link, and tracking and footer content past the cut are effectively gone for most readers.

**Fix**

1. **Check the file size.** Save the final HTML and check its size on disk. If it's near or over 100KB, it needs to shrink before sending.
2. **Remove what's invisible but heavy:**
   - HTML comments (developer notes add up fast).
   - Unused CSS from a framework export or an edited-down template.
   - Repeated inline styles that can be shortened without changing rendering.
   - Excess whitespace and indentation.
3. **Cut content before cutting code.** If the email is still too large, it is probably trying to say too much. Move secondary content to a landing page and link to it.
4. **Retest by sending to a real Gmail account**, not a preview. Clipping happens on delivery, and previews don't always reflect it.

---

## Issue 4: Columns don't stack on mobile

**Problem**
A two- or three-column layout that looks right on desktop is squeezed, tiny, or overlapping on phones instead of stacking into a single column.

**Symptom**

- Columns shrink to unreadable widths on a phone screen.
- Text in side-by-side blocks becomes a few words per line.
- In some apps the layout holds; in others it collapses entirely.

**Cause**
Stacking depends on media queries in a `<style>` block, and not every client supports them. Gmail supports media queries in most contexts; some Outlook mobile apps and older Android clients ignore them completely. A layout that *requires* stacking to be readable will fail somewhere.

**Fix**

1. **Add the stacking media query.** Target screens under your container width and force columns full width:

   ```css
   @media only screen and (max-width: 620px) {
     .stack { display: block !important; width: 100% !important; max-width: 100% !important; }
   }
   ```

   Apply the `stack` class to each column cell.

2. **Design columns that degrade gracefully.** Order content so the email still reads correctly left-to-right, top-to-bottom if the columns stay side by side in a client that ignores the query.
3. **Use a hybrid approach for critical layouts.** Fluid tables with `max-width` on columns (sometimes called hybrid or spongy coding) narrow and wrap without depending on media queries at all. It takes more code and is worth it for emails that must work everywhere.
4. **Set images to scale.** `width:100%; height:auto` with a `max-width` keeps images inside a stacked column instead of forcing it wider than the screen.
5. **Test on a physical phone.** Emulators get close; a real Gmail app on a real device is the test that counts.

---

## Issue 5: Images are blocked

**Problem**
The email arrives with empty boxes where the images should be. If the design leans on those images, the email looks broken or says almost nothing.

**Symptom**

- Large blank areas, sometimes with a broken-image icon.
- Buttons or headlines that were images are simply absent.
- The layout shifts or collapses around the missing images.

**Cause**
Many clients and corporate mail systems block images by default until the reader allows them. Outlook on Windows does this out of the box, and plenty of readers never turn images on. An email that carries its message inside images has no message at all for those readers.

**Fix**

1. **Never put essential content only in an image.** Headlines, offers, prices, and calls to action belong in real text. Images support the message; they can't be the message.
2. **Write descriptive alt text on every image.** `alt="Spring sale: 25% off all planters through Sunday"` carries the content. `alt="image"` carries nothing.
3. **Style the alt text.** Set font size, color, and alignment inline on the image tag so the fallback text looks intentional, not like an error.
4. **Give images fixed dimensions.** Accurate `width` and `height` attributes let clients lay out the email correctly before (or without) loading images, so the design holds its shape.
5. **Test with images off.** Every pre-send check should include one pass with images disabled in a real client. If the email still communicates, it passes.

---

## Before you send: a five-minute checklist

Run through this list whenever an email looks broken and again before any important send:

1. Is every critical style inlined on its element?
2. Does the HTML stay under ~102KB?
3. Did you open the test send with images blocked?
4. Did you check dark mode in at least two clients?
5. Did you send a real test to a physical phone?

Most rendering problems are caught by one of these five. The rest are caught by a rendering tool, which is why a final Litmus or Email on Acid pass is standard practice before a campaign goes out.
