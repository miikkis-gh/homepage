# miikkis Homepage

A personal link hub for miikkis with a minimal black-and-white design.

## Appearance

### Background
- **Soft radial spotlight glow** that drifts gently with mouse-parallax
- **Noise texture overlay** for added depth and visual interest

### Profile Section
- "miikkis" title set in a custom Derbyshire Bold display font, dimmed to 75% opacity with a subtle gradient
- Smooth fade-in entrance animation

### Three-Column Layout
On wider screens, updates, links, and playlists sit side by side; on mobile they stack in that order.

**Updates** — a self-hosted micro-feed of short posts, sourced live from a published Google Sheet (columns: `date`, `text`). Rendered as tweet-style cards, newest first, capped at 8. To post, add a row to the sheet — no code changes needed. Date values should use `YYYY-MM-DD` so sort order stays correct.

**Links** — three flat, bordered link cards:
- Suno
- Spotify
- Instagram

Each card features the service's official logo in its original brand color, a sliding arrow reveal on hover, and staggered entrance animations.

**Playlists** — embedded Spotify players for three public playlists, so visitors can preview and play tracks without leaving the page.

### Footer
Copyright text, hidden until hovered.

## Features

- **Minimal flat design** with subtle borders, no heavy blur or glow effects
- **Smooth animations** including entrance transitions and interactive hover effects
- **Responsive layout** adapting seamlessly to mobile, tablet, and desktop screens
- **Accessibility support** with keyboard focus states, reduced motion preferences (disables mouse-parallax too), and high contrast mode
- **Inter typography** for a clean, modern look
