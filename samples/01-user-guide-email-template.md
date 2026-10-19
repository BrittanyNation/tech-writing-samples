# Getting Started: Build Your First Responsive HTML Email Template

This guide walks you through building a complete, responsive HTML email template from an empty file. By the end, you'll have a template with a header, a headline, body copy, a call-to-action button, and a footer that renders correctly in Gmail, Apple Mail, Outlook, and on mobile devices.

## Who this guide is for

Beginners who know basic HTML and CSS and want to build their first email template. If you've built a web page before, you know enough to start. Email HTML is stricter and older-fashioned than web HTML, and this guide points out every place where the rules change.

## What you'll build

A single-column email template, 600 pixels wide on desktop, that stacks and scales down cleanly on phones. It's the same structure used by most production welcome emails, and every technique in it (table layout, inline styles, Outlook fallbacks) carries over to more complex designs.

## Prerequisites

Before you begin, make sure you have:

- **A code editor.** Visual Studio Code is free and works well. Any plain-text editor will do.
- **A way to preview the file.** A web browser is enough for the first pass.
- **A test inbox.** A free Gmail account you can send test emails to. You'll need this in Step 8.
- **Basic HTML and CSS familiarity.** You should know what a tag, an attribute, and a style declaration are. You do not need JavaScript. Email doesn't use it.

> **A note before you write any code:** Email clients do not render HTML the way browsers do. Most strip out `<style>` blocks, ignore modern CSS layout (flexbox and grid are unreliable), and one major client, Outlook on Windows, renders email using Microsoft Word's engine. The techniques in this guide exist because of those limits, not in spite of them.

## Step 1: Create the file and the document shell

1. Create a new file named `my-first-email.html`.
2. Paste in the following shell:

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" lang="en">
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="color-scheme" content="light dark">
  <meta name="supported-color-schemes" content="light dark">
  <title>My First Email</title>
</head>
<body style="margin:0; padding:0; background-color:#f4f4f4;">
</body>
</html>
```

Three details in this shell matter:

- The **XHTML doctype** is the one email clients handle most predictably. Don't substitute the shorter HTML5 doctype.
- The **viewport meta tag** is what lets the email scale correctly on phones.
- The **color-scheme meta tags** tell clients your email supports both light and dark mode, which stops some clients from forcibly inverting your colors.

## Step 2: Add the layout table

Email layout is built with tables. Not because it's elegant, but because it's the only structure every client renders the same way.

Inside the `<body>`, add the outer table:

```html
<table role="presentation" width="100%" cellpadding="0" cellspacing="0" border="0" style="background-color:#f4f4f4;">
  <tr>
    <td align="center">
      <!-- Container table goes here -->
    </td>
  </tr>
</table>
```

Always include `role="presentation"`, `cellpadding="0"`, `cellspacing="0"`, and `border="0"` on layout tables. The attributes remove default spacing that otherwise appears as mystery gaps, and `role="presentation"` tells screen readers the table is layout, not data.

## Step 3: Add the 600px container

Inside the marked cell, add a second table. This is the container your content lives in:

```html
<table role="presentation" width="600" cellpadding="0" cellspacing="0" border="0" style="width:600px; max-width:600px; background-color:#ffffff;">
  <tr>
    <td style="padding:32px; font-family:Arial, sans-serif;">
      <!-- Email content goes here -->
    </td>
  </tr>
</table>
```

600 pixels is the long-standing desktop standard: wide enough to design in, narrow enough that no client needs to scale it down. The `max-width` in the inline style is what allows the container to shrink on smaller screens later.

> **Tip:** Outlook on Windows ignores `max-width`. Step 7 adds the conditional-comment fix that holds Outlook to 600px. Don't skip it if your audience includes corporate readers. It does.

## Step 4: Write the content with inline styles

Every style that matters must be written directly on the element it affects. Add this content inside the container cell:

```html
<h1 style="margin:0 0 16px 0; font-size:28px; line-height:34px; color:#222222;">Welcome aboard</h1>
<p style="margin:0 0 16px 0; font-size:16px; line-height:24px; color:#444444;">
  Thanks for joining us. Here's what to do first.
</p>
```

Rules to follow for all email content:

- **Set margins explicitly on every element.** Clients apply their own default margins; an unset margin is a surprise waiting in one client or another.
- **Use web-safe fonts or a declared stack.** `Arial, sans-serif` renders everywhere. Custom fonts work in some clients and fall back in others, so always name a safe fallback.
- **Style text on the text element itself**, not on a parent table cell. Outlook in particular does not inherit styles the way you'd expect.

## Step 5: Build a button that works in Outlook

A styled `<a>` tag is not a reliable button. Outlook ignores much of its padding, and the button collapses. Build buttons as a small table instead:

```html
<table role="presentation" cellpadding="0" cellspacing="0" border="0">
  <tr>
    <td align="center" style="background-color:#2b6cb0; border-radius:6px;">
      <a href="https://example.com/get-started" target="_blank"
         style="display:inline-block; padding:14px 28px; font-family:Arial, sans-serif; font-size:16px; color:#ffffff; text-decoration:none;">
        Get started
      </a>
    </td>
  </tr>
</table>
```

The padding lives on the link, the background color lives on the cell, and the whole cell becomes the tap target. This pattern is often called a bulletproof button because it survives every major client, including Outlook.

> **Tip:** Make the entire cell easy to tap. On mobile, a target smaller than about 44 pixels tall frustrates readers and costs clicks.

## Step 6: Add the footer

Every production email needs a footer with your physical address and an unsubscribe path. Add a final row to the container table:

```html
<tr>
  <td style="padding:24px 32px; font-family:Arial, sans-serif; font-size:12px; line-height:18px; color:#888888; text-align:center;">
    Your Company, 123 Main Street, Springfield, IL 62701<br>
    <a href="#" style="color:#888888;">Unsubscribe</a> · <a href="#" style="color:#888888;">View in browser</a>
  </td>
</tr>
```

When you send through an email service provider (ESP), replace the `#` placeholders with that provider's merge tags for unsubscribe and view-in-browser. The unsubscribe link is a legal requirement for marketing email in the United States, not a design choice.

## Step 7: Add the Outlook container fix

Outlook on Windows ignores `max-width` and will stretch your container to the full window width. The fix is a pair of conditional comments that only Outlook can see. Wrap the container table from Step 3 like this:

```html
<!--[if mso]>
<table role="presentation" width="600" cellpadding="0" cellspacing="0" border="0" align="center"><tr><td>
<![endif]-->

  <!-- your 600px container table here -->

<!--[if mso]>
</td></tr></table>
<![endif]-->
```

Every client except Outlook treats these as ordinary comments and ignores them. Outlook reads them as a real fixed-width table wrapped around your content, which holds the layout at 600px.

## Step 8: Test before you send anything

A template that looks right in your browser has passed the easiest test, not the real one. Test in this order:

1. **Open the file in a browser.** Check alignment, spacing, and that nothing overflows.
2. **Send a real test email to yourself.** Use your ESP's test send, or a free tool that delivers the HTML as an email. Open it in Gmail and in a mobile mail app.
3. **Turn images off and read it again.** Many readers see your email this way first. The layout and message must survive on text and alt text alone.
4. **Check the file size.** If the HTML is over roughly 102KB, Gmail clips the message and hides your footer behind a "view entire message" link. Trim comments and unused styles to get under the limit.

If you have access to a rendering tool such as Litmus or Email on Acid, run the template through it before your first real send. It shows screenshots from dozens of clients at once and catches problems you can't see from your own devices.

## Tips for your next template

- **Add a hidden preheader.** A short line of preview text at the top of the body (styled with `display:none` so it doesn't show in the email itself) controls what appears next to the subject line in the inbox. It's the highest-value 30 characters in the email.
- **Stack columns, don't shrink them.** Two-column layouts should become one column on mobile, using a media query in a `<style>` block for the clients that support it. Some clients ignore media queries, so design columns that still make sense side by side if the stacking fails.
- **Keep a personal template library.** Save each working pattern (button, header, footer, product block) as its own snippet. Building emails gets much faster when you're assembling tested parts instead of starting over.

## Troubleshooting

**The email looks perfect in my browser but broken in Outlook.**
This is the most common email problem there is. Work through this checklist: Is every style inlined? Did you include the conditional comments from Step 7? Are all layout tables carrying `cellpadding="0"` and `cellspacing="0"`? One of those three fixes the majority of Outlook breakage.

**There are white gaps between images.**
Email clients add space below images by default. Add `display:block` to the image's inline style, and set `border="0"` on the image tag.

**The text is tiny on phones.**
The viewport meta tag from Step 1 is missing or was removed. Without it, mobile clients render the email at desktop width and shrink it to fit.

**Gmail cut off the bottom of my email.**
Your file is over Gmail's ~102KB clipping limit. Remove HTML comments, collapse whitespace, and delete styles you're not using, then check the size again.

**My styles in the `<style>` block aren't showing up.**
Several clients, including some versions of Gmail and most Outlook versions, strip or ignore `<style>` blocks. Move the style onto the element itself. Inline styles are the baseline; treat anything in a `<style>` block as an enhancement that some readers won't get.

## Where to go next

You now have a working, tested, single-column template. From here, the natural next steps are a styled header with a logo image, a two-column product block, and dark mode styles using the `prefers-color-scheme` media query. Each builds directly on the structure in this guide.
