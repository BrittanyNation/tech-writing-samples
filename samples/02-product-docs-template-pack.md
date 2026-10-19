# Email Template Pack: Product Documentation

Documentation for the **Responsive HTML Email Template Pack**, a set of three hand-coded, production-style email templates published at [github.com/BrittanyNation/email-template-pack](https://github.com/BrittanyNation/email-template-pack).

This document covers what the pack is, how to use the templates, what each template contains, how to customize them for your own brand and email service provider (ESP), and answers to common questions.

## Overview

The Email Template Pack provides three ready-to-use HTML email templates:

- **Welcome** — an onboarding email with a getting-started steps block
- **Newsletter** — a content email with a featured story and article blocks
- **Promotional** — an offer email with a hero image and a two-product grid

Every template is written by hand using table-based layout and inline CSS. There is no framework output and no drag-and-drop builder export. The templates are designed to render consistently in the clients that break email most often: Outlook on Windows, Gmail (web and mobile), Apple Mail, and mobile clients with dark mode enabled.

### Design principles

| Principle | What it means in practice |
| --- | --- |
| Table-based layout | All structure uses nested tables with `role="presentation"`, `cellpadding="0"`, and `cellspacing="0"`, the only layout method every major client renders the same way. |
| Inline CSS | Critical styling is written on the elements themselves. The `<style>` block carries only what cannot be inlined: media queries, dark-mode rules, and client resets. |
| Fixed 600px container | Content sits in a 600px container, held in place for Outlook by MSO conditional-comment "ghost tables." |
| Responsive by default | A 620px breakpoint stacks multi-column blocks full width and scales images fluidly on small screens. |
| Dark mode supported | `color-scheme` meta tags, `prefers-color-scheme` rules, and Outlook `[data-ogsc]` / `[data-ogsb]` overrides keep colors intentional in both modes. |

## Getting the templates

### Download

1. Go to [github.com/BrittanyNation/email-template-pack](https://github.com/BrittanyNation/email-template-pack).
2. Select **Code**, then **Download ZIP**, and unzip the file.

Or clone the repository:

```bash
git clone https://github.com/BrittanyNation/email-template-pack.git
```

### File structure

```
email-template-pack/
├── README.md
└── templates/
    ├── welcome.html
    ├── newsletter.html
    └── promotional.html
```

There is no build step and nothing to install. Each template is a single, self-contained HTML file. Open one in a browser to preview it, and open it in any code editor to modify it.

## Using a template

1. **Copy the file** for the email you want to send and rename the copy (for example, `spring-welcome.html`). Keep the originals unchanged so you always have a clean starting point.
2. **Replace the placeholder content**: brand name, logo, headlines, body copy, links, and footer address. See the [customization guide](#customization-guide) for the full checklist.
3. **Replace the placeholder images** with images hosted on your own CDN or your ESP's media library.
4. **Swap the footer links** for your ESP's merge tags (unsubscribe, preferences, view in browser).
5. **Test before sending.** Preview the email in a rendering tool (Litmus or Email on Acid) and send real tests to at least one desktop and one mobile inbox, in both light and dark mode.

## Template reference

### welcome.html

A welcome / onboarding email for new subscribers or customers.

| Section | Contents |
| --- | --- |
| Preheader | Hidden preview text with a spacer hack so no body copy leaks into the inbox preview |
| Hero | Brand block at the top of the email |
| Welcome message | Headline and introductory copy |
| Getting-started steps | A three-column steps block that stacks vertically on mobile |
| Call to action | A single padded-cell button built to hold its shape in Outlook |
| Footer | Unsubscribe, preferences, and view-in-browser link placeholders |

**Use it for:** signup confirmations, new-customer onboarding, and any first email where one clear next action matters.

### newsletter.html

A recurring content email.

| Section | Contents |
| --- | --- |
| Masthead | Newsletter name with an issue line (for example, "Issue 12 · October 2026") |
| Featured story | Large image with a headline and summary |
| Article blocks | Two-column story blocks that stack on mobile |
| Quick tips card | A numbered list of short items in a bordered card |
| Footer | Standard footer with unsubscribe and preferences placeholders |

**Use it for:** weekly or monthly digests, blog roundups, and company updates.

### promotional.html

A sales and offer email.

| Section | Contents |
| --- | --- |
| Announcement bar | A slim top bar for the headline offer (for example, a sale end date) |
| Hero | Full-width hero image with offer copy and a deadline line |
| Call to action | Primary button under the offer |
| Product grid | Two product cards with images, prices, and per-product buttons |
| Secondary band | A supporting content strip (for example, a guide or lookbook link) |
| Footer | Standard footer with unsubscribe and preferences placeholders |

**Use it for:** sales announcements, product launches, and seasonal campaigns.

## Customization guide

### Change the brand colors

Brand colors are set inline on individual elements, because inline styles are what email clients respect. To rebrand a template:

1. Search the file for the current hex values (the pack's default brand color is used for headings, buttons, and links).
2. Replace each occurrence with your brand's hex value.
3. Check the footer and button background colors as well; they are defined separately on their own cells.
4. Open the file in a browser to confirm nothing was missed, then test in dark mode, because dark-mode rules in the `<style>` block define their own background and text colors.

### Change the buttons

Buttons are padded table cells with a link inside. To change a button:

- **Text:** edit the link text.
- **Link:** replace the `href="#"` placeholder with your URL.
- **Color:** change the `background-color` on the `<td>`, not on the link.
- **Size:** adjust the `padding` on the link.

### Replace the images

Placeholder images come from placehold.co. For production sends:

1. Upload your images to your CDN or ESP media library.
2. Replace each placeholder URL with your hosted image URL.
3. Keep the `width` and `height` attributes accurate for the image's display size. Outlook uses these attributes to lay out the email before images load.
4. Write descriptive `alt` text on every image. Many readers see alt text first, with images blocked, and screen readers rely on it.

### Add your ESP's merge tags

The templates ship with placeholder links. Before sending, replace them with your provider's syntax. Common examples:

| Purpose | Mailchimp | Klaviyo / Customer.io style | Salesforce Marketing Cloud |
| --- | --- | --- | --- |
| Unsubscribe | `*\|UNSUB\|*` | `{{ unsubscribe_url }}` (check your provider's current syntax) | The account's unsubscribe personalization string |
| View in browser | `*\|ARCHIVE\|*` | Provider-specific tag | `%%view_email_url%%` |
| First name | `*\|FNAME\|*` | `{{ first_name }}` | `%%FirstName%%` |

Merge tag syntax changes between providers and occasionally within them. Always confirm the current syntax in your provider's documentation and verify with a live test send to a real contact record, not a preview, because previews don't always process personalization.

### Using the templates in Salesforce Marketing Cloud

The templates can be pasted into SFMC Content Builder as HTML content blocks or full templates. To make a template reusable across sends:

1. Move shared sections (header, footer) into their own content blocks.
2. Reference those blocks in the template with `%%=ContentBlockByName("...")=%%`, or the block reference AMPscript your account uses.
3. If non-developers will edit the email, replace static headlines and copy with named content areas.
4. Leave the inline CSS as it is. Content Builder preserves it, and inlining is what keeps rendering consistent once the email is assembled.

## FAQ

**Do I need to know how to code to use these templates?**
For small text changes, basic familiarity with HTML is enough: find the text, replace it, save. For layout changes, new sections, or color rebrands across a whole template, you'll want to be comfortable reading HTML tables and inline styles, or work with someone who is. Nothing in the pack requires JavaScript or a build tool.

**Why tables? Modern CSS exists.**
It does, and email clients largely ignore it. Outlook on Windows renders email with Microsoft Word's engine, which doesn't support flexbox or grid. Tables with inline CSS are the only approach that renders predictably in every major client today.

**Can I use these templates in my ESP's drag-and-drop builder?**
Not directly. These are code templates: use them through your ESP's HTML import or paste-in editor, or in SFMC Content Builder as described above. If your workflow is entirely drag-and-drop, the templates are still useful as a layout reference.

**Why is there a file size warning in every template's checklist?**
Gmail clips messages larger than about 102KB of HTML, hiding everything past the limit, including your unsubscribe link, behind a "view entire message" prompt. Each template in this pack is well under that limit as shipped. If your edits push a file toward it, remove comments and unused styles first.

**Do the templates support dark mode?**
Yes. Each template declares color-scheme support, includes `prefers-color-scheme` rules for clients that honor them, and carries Outlook's `[data-ogsc]` and `[data-ogsb]` overrides. Dark mode handling varies by client, so test on at least one Apple Mail and one Outlook client before sending.

**Can I use these templates for client or commercial work?**
The templates were written as portfolio work with a fictional brand ("Lumen & Co.") and sample copy, so they can be evaluated as layout and code. Replace all placeholder copy, images, and brand details with your own before any real send.
