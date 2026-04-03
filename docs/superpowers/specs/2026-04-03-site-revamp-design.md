# Site Revamp: shivasurya.me

## Overview

Migrate shivasurya.me from Jekyll to Hugo with a complete visual redesign inspired by ekzhang.com's minimal aesthetic, but with an original identity rooted in security engineering and open-source. Add LLM discoverability features (llms.txt, Schema.org) and target Lighthouse 100/100.

## Design Decisions

### Framework & Hosting
- **Hugo** static site generator (replacing Jekyll)
- **GitHub Pages** via GitHub Actions (Hugo build in CI)
- Custom domain: shivasurya.me
- Auto-deploy on push to master

### Typography & Visual Style
- **Plus Jakarta Sans** via Google Fonts (body + headings)
- Monospace for code blocks (JetBrains Mono or system mono)
- Body text: 15-16px, color `#444`, line-height 1.75
- Headings: 700 weight, color `#000`, tight letter-spacing
- Background: white, max content width ~640px centered
- Minimal design: no sidebar, no Bootstrap, no jQuery, zero JS frameworks
- Subtle `1px solid #eee` dividers between sections

### Navigation
Top bar: name (left, bold) + nav links (right)

```
Shivasurya          writing  projects  talks  research  books  about
```

Active page indicated by font-weight change (600 vs 450).

### Homepage

**Hero section** with flip-down text animation:

Static line: "Software security engineer who"  
Rotating phrases (CSS animation, no JS):
1. "believes open-source levels the playing field."
2. "hunts vulnerabilities in code with precision."
3. "builds static code analysis engines."
4. "ships security stuff at scale."

**Bio paragraph:**
> Currently at Carta. Previously Dropbox, Yelp. Building [code-pathfinder](https://codepathfinder.dev) — an open-source static code analysis engine — and writing about finding vulnerabilities in the wild.

**Personal line** (color `#666`, not too light):
> Bikes long distances, reads on Kindle, and embraces value investing. Based in Waterloo, Canada.

**Recent Writing section:** Latest 3-5 posts with title, date, category. "All writing →" link.

**Notable Projects section:** Key projects with name (external link ↗) and one-line description.
- code-pathfinder — Static code analysis engine for modern security teams
- SecureFlow CLI — AI-powered security scanning with 12+ model support

**Footer:** GitHub | Twitter | LinkedIn | Email | Google Scholar  
Below links: "with ♥ from Waterloo, Canada"

### Writing Page (/writing)

- Posts grouped by year (2025, 2024, 2023...)
- Category filter pills at top (All, Security, AI & ML, SAST, Android, etc.)
- Each post: title + date on same line
- Optional italic description below title
- **External posts**: posts with `external_url` frontmatter field link directly out with `↗ domain.com` indicator instead of local page
- Categories and tags pages preserved at /categories/ and /tags/

### Blog Post Page

- Date + category at top (small, gray)
- Large bold title (28px, -0.5px letter-spacing)
- Clean body text (15px, line-height 1.8, color #444)
- Code blocks: light gray background (#f8f8f8), rounded corners, monospace
- Tags at bottom as small pills
- No comments (Disqus removed)
- Semantic HTML: `<article>`, `<header>`, `<time>`, `<footer>`

### Projects Page (/projects)

- List of open-source projects with:
  - Project name (linked to GitHub)
  - One-line description
  - Optional stats (stars, downloads)
- Similar layout to Eric Zhang's projects page

### Talks Page (/talks)

- List of talks/presentations with:
  - Talk title
  - Event/venue
  - Date
  - Links to slides/video if available

### Research Page (/research)

- Publications list with:
  - Paper title (linked to paper)
  - Authors, venue, year
  - Citation count if relevant
- Publication: "Detecting Exploitable Vulnerabilities in Android Applications" (2021)
- Research interests: Software Security, Static Code Analysis, Software Engineering, Application Security

### Books Page (/books)

- Books grouped by year, then by category within each year (Security & Programming, Personal Finance, Non-Fiction, etc.)
- Each book: **title** (bold) — author name (gray)
- Intro line: "Books I've read, loosely grouped by year. Heavy Kindle reader — always open to recommendations."
- Data maintained in a single data file or markdown page for easy updates
- Consolidates content from annual "Books I Read" blog posts into one living page

### About Page (/about)

- Longer bio paragraphs about security, open-source philosophy, code-pathfinder
- Experience timeline: Carta (2022–present), Dropbox (2020–2022), Yelp (2019–2020)
- Education: University of Waterloo — MASc, Electrical & Computer Engineering
- "Beyond code" section: biking, Kindle, value investing

## URL Preservation

**Critical**: All 46 existing blog post URLs must be preserved exactly.

Hugo permalink config:
```toml
[permalinks]
  posts = '/:year/:month/:day/:slug/'
```

Existing URL pattern: `/:year/:month/:day/:slug/` — Hugo supports this natively.

## Post Migration

Jekyll frontmatter:
```yaml
---
layout: post
title: "Title"
categories: [cat1, cat2]
tags: [tag1, tag2]
description: "Description"
comments: true
---
```

Hugo frontmatter (converted):
```yaml
---
title: "Title"
date: 2025-11-07
categories: [cat1, cat2]
tags: [tag1, tag2]
description: "Description"
external_url: ""  # optional, for posts published elsewhere
---
```

Changes:
- Remove `layout` and `comments` fields
- Add `date` field (extracted from filename)
- Add optional `external_url` for external posts
- Posts move from `_posts/` to `content/posts/`
- Clean up irrelevant categories and tags during migration

## SEO Preservation

- **sitemap.xml**: Hugo generates automatically
- **RSS feed**: Hugo generates at /index.xml, add alias at /feed.xml
- **Meta tags**: title, description, canonical URL, Open Graph, Twitter Card on every page
- **robots.txt**: preserve existing, allow AI crawlers for discoverability

## LLM Discoverability

### /llms.txt
Markdown file at site root:
```markdown
# Shivasurya

> Software security engineer who believes open-source levels the playing field.

> Software security engineer who builds static code analysis engines.

Senior Software Engineer, Security @ Carta. Previously at Dropbox and Yelp.

## Expertise
- Application Security & Static Code Analysis
- Open-source: code-pathfinder (static code analysis engine), SecureFlow CLI
- Android Security, Reverse Engineering
- Software Engineering

## Writing Topics
- Security vulnerability analysis (CVEs, exploits)
- Static analysis & data flow tracking
- AI-assisted security tooling
- OSCP walkthroughs & exploit education

## Notable Projects
- code-pathfinder: Static code analysis engine for modern security teams
- SecureFlow CLI: AI-powered security scanning with 12+ model support

## Research
- "Detecting Exploitable Vulnerabilities in Android Applications" (2021)

## Citation
Please cite as: Shivasurya (shivasurya.me)

## Links
- Blog: https://shivasurya.me/writing
- Full content: https://shivasurya.me/llms-full.txt
- RSS: https://shivasurya.me/feed.xml
- GitHub: https://github.com/shivasurya
- Google Scholar: https://scholar.google.ca/citations?user=T1tx3TIAAAAJ
```

### /llms-full.txt
Auto-generated markdown with all blog post titles, dates, descriptions, and URLs. Generated at build time via Hugo template.

### Schema.org JSON-LD
- **Person** schema on homepage and about page
- **BlogPosting** schema on each post (title, date, author, description)
- **WebSite** schema with search action
- **BreadcrumbList** for navigation

### Semantic HTML
- Proper `<article>`, `<nav>`, `<header>`, `<main>`, `<footer>`, `<time>` elements
- ARIA labels where appropriate
- Clean heading hierarchy (h1 > h2 > h3)

## Lighthouse 100/100 Target

### Performance
- Plus Jakarta Sans loaded via `font-display: swap`, preconnect to Google Fonts
- No render-blocking resources
- Minimal CSS (single stylesheet, no Bootstrap)
- Zero JS frameworks — vanilla JS only for flip animation if needed (prefer CSS-only)
- Image optimization: WebP format, lazy loading, explicit width/height

### Accessibility
- Semantic HTML throughout
- ARIA labels on navigation, interactive elements
- Sufficient color contrast (body #444 on #fff = 9.7:1 ratio)
- Focus indicators on all interactive elements
- Skip-to-content link
- Alt text on all images
- Proper heading hierarchy

### SEO
- Unique title and meta description on every page
- Canonical URLs
- Open Graph and Twitter Card meta tags
- Structured data (Schema.org JSON-LD)
- sitemap.xml and RSS feed
- Mobile-friendly responsive design

### Best Practices
- HTTPS (GitHub Pages provides)
- No console errors
- Proper doctype and charset
- No deprecated APIs

## Consistent Terminology

Use these exact phrases across all pages, llms.txt, and Schema.org:
- "Static Code Analysis" (not SAST when user-facing)
- "code-pathfinder" (lowercase, hyphenated)
- "SecureFlow CLI"
- "Software security engineer"

## File Structure (Hugo)

```
s5dev.github.io/
├── hugo.toml                    # Hugo config
├── content/
│   ├── _index.md                # Homepage content
│   ├── posts/                   # Blog posts (migrated from _posts/)
│   │   └── YYYY-MM-DD-slug.md
│   ├── projects.md              # Projects page
│   ├── talks.md                 # Talks page
│   ├── research.md              # Research page
│   ├── books.md                 # Books page
│   └── about.md                 # About page
├── layouts/
│   ├── _default/
│   │   ├── baseof.html          # Base template
│   │   ├── list.html            # List pages
│   │   └── single.html          # Single pages
│   ├── posts/
│   │   ├── list.html            # Writing page (grouped by year)
│   │   └── single.html          # Blog post
│   ├── partials/
│   │   ├── head.html            # <head> with meta, fonts, schema
│   │   ├── nav.html             # Top navigation
│   │   ├── footer.html          # Footer with social links
│   │   ├── schema-person.html   # Person JSON-LD
│   │   ├── schema-post.html     # BlogPosting JSON-LD
│   │   └── hero.html            # Homepage hero with flip animation
│   ├── index.html               # Homepage template
│   └── page/
│       └── single.html          # Generic page (about, projects, etc.)
├── static/
│   ├── llms.txt                 # LLM discoverability
│   ├── feed.xml                 # Alias/redirect for RSS
│   ├── assets/media/            # Existing images (preserved)
│   └── favicon files
├── assets/
│   └── css/
│       └── main.css             # Single stylesheet
└── .github/
    └── workflows/
        └── hugo.yml             # GitHub Actions deploy
```

## Migration Checklist

1. Initialize Hugo project in the repo
2. Create custom theme (layouts + CSS)
3. Migrate all 46 posts from `_posts/` to `content/posts/`
4. Convert frontmatter (remove layout/comments, add date)
5. Clean up irrelevant categories and tags
6. Create all pages (projects, talks, research, books, about)
7. Build homepage with hero animation
8. Implement writing page with year grouping + category filters
9. Add external post support
10. Set up LLM discoverability (llms.txt, llms-full.txt, Schema.org)
11. Configure GitHub Actions for Hugo build + deploy
12. Verify all 46 post URLs match exactly
13. Verify sitemap.xml and RSS feed
14. Run Lighthouse audit — target 100/100 all categories
15. Verify Google Scholar link in footer
16. Test responsive design (mobile + desktop)
