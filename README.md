# miikkis Homepage

A personal link hub for miikkis with a minimal black-and-white design.

## Appearance

### Background
- **Soft radial spotlight glow** that drifts gently with mouse-parallax
- **Noise texture overlay** for added depth and visual interest

### Profile Section
- Live **Helsinki clock**, ticking every second
- Live **location line**, updated automatically when miikkis arrives somewhere new (via an iPhone Shortcuts → Apps Script pipeline), with a small (i) icon — click or hover for an explanation
- Smooth fade-in entrance animation

### Three-Column Layout
On wider screens the columns run horizontally in the order Links, Updates, Playlists; on mobile they stack in that order.

**Updates** — a self-hosted micro-feed of short posts, sourced live from a published Google Sheet (columns: `date`, `text`, `image`). Rendered as tweet-style cards, newest first, capped at 8. To post, add a row to the sheet — no code changes needed. Dates accept `YYYY-MM-DD` or `D.M.YYYY`. A leading `>` in the text renders the post as a styled blockquote, and literal newlines in the cell render as line breaks. `image` can carry an optional photo, shown below the post text.

**Links** — three flat, bordered link cards:
- Suno
- Spotify
- Instagram

Each card features the service's official logo in its original brand color, a sliding arrow reveal on hover, and staggered entrance animations.

**Playlists** — an embedded Spotify player, so visitors can preview and play tracks without leaving the page.

### Footer
Copyright text, hidden until hovered.

## Features

- **Minimal flat design** with subtle borders, no heavy blur or glow effects
- **Smooth animations** including entrance transitions and interactive hover effects
- **Responsive layout** adapting seamlessly to mobile, tablet, and desktop screens
- **Accessibility support** with keyboard focus states, reduced motion preferences (disables mouse-parallax too), and high contrast mode
- **Inter typography** for a clean, modern look
