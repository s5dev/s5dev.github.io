# Site Revamp Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Migrate shivasurya.me from Jekyll to Hugo with a minimal, professional redesign, preserving all 46 blog post URLs, adding LLM discoverability, and targeting Lighthouse 100/100.

**Architecture:** Custom Hugo theme with single CSS file (Plus Jakarta Sans), CSS-only flip animation on homepage, semantic HTML throughout. All existing blog posts migrated with frontmatter conversion. Static pages for writing, projects, talks, research, books, about. LLM layer via static llms.txt and build-time generated llms-full.txt.

**Tech Stack:** Hugo (static site generator), Plus Jakarta Sans (Google Fonts), vanilla CSS, GitHub Actions for CI/CD to GitHub Pages.

**Spec:** `docs/superpowers/specs/2026-04-03-site-revamp-design.md`

---

## File Structure

```
s5dev.github.io/
├── hugo.toml                          # Hugo config (site metadata, permalinks, menus)
├── content/
│   ├── _index.md                      # Homepage content data (bio, tagline, projects list)
│   ├── posts/                         # Migrated blog posts (46 files)
│   │   └── YYYY-MM-DD-slug.md
│   ├── writing/_index.md              # Writing page (list override)
│   ├── projects.md                    # Projects page
│   ├── talks.md                       # Talks page
│   ├── research.md                    # Research page
│   ├── books.md                       # Books page
│   └── about.md                       # About page
├── layouts/
│   ├── _default/
│   │   ├── baseof.html                # Base template (doctype, head, nav, footer)
│   │   ├── list.html                  # Default list template
│   │   └── single.html               # Default single page template
│   ├── posts/
│   │   ├── list.html                  # Writing page (year-grouped, category pills)
│   │   └── single.html               # Blog post template
│   ├── partials/
│   │   ├── head.html                  # <head>: meta, fonts, schema, OG tags
│   │   ├── nav.html                   # Top navigation bar
│   │   ├── footer.html               # Footer (social links + love line)
│   │   ├── schema-person.html         # Person JSON-LD
│   │   ├── schema-post.html           # BlogPosting JSON-LD
│   │   └── hero.html                  # Homepage hero with flip animation
│   ├── index.html                     # Homepage template
│   └── _default/
│       └── llms-full.txt              # Template for /llms-full.txt output
├── static/
│   ├── llms.txt                       # LLM discoverability file
│   └── assets/media/                  # Existing images (copied from current)
├── assets/
│   └── css/
│       └── main.css                   # Single stylesheet
└── .github/
    └── workflows/
        └── hugo.yml                   # GitHub Actions deploy
```

---

## Task 1: Install Hugo and Initialize Project

**Files:**
- Create: `hugo.toml`

- [ ] **Step 1: Install Hugo**

```bash
# On Ubuntu/Debian
sudo snap install hugo
# Verify
hugo version
```

- [ ] **Step 2: Initialize Hugo project structure**

Do NOT run `hugo new site` — we're adding Hugo files to the existing repo. Create the config file manually.

```bash
cd /home/shivasurya/src/shivasurya/s5dev.github.io
```

Create `hugo.toml`:

```toml
baseURL = "https://shivasurya.me/"
languageCode = "en-us"
title = "Shivasurya"
description = "Software security engineer who believes open-source levels the playing field."

[params]
  author = "Shivasurya"
  email = "shiva@shivasurya.me"
  github = "shivasurya"
  twitter = "sshivasurya"
  linkedin = "shivasurya"
  scholar = "https://scholar.google.ca/citations?user=T1tx3TIAAAAJ"
  bio = "Senior Software Engineer, Security @ Carta | Prev @ Dropbox, Yelp"
  googleAnalytics = "G-1FY99CSBP2"

[permalinks]
  posts = "/:year/:month/:day/:slug/"

[markup]
  [markup.highlight]
    style = "github"
    lineNos = false
    noClasses = false
  [markup.goldmark]
    [markup.goldmark.renderer]
      unsafe = true

[menu]
  [[menu.main]]
    name = "writing"
    url = "/writing/"
    weight = 1
  [[menu.main]]
    name = "projects"
    url = "/projects/"
    weight = 2
  [[menu.main]]
    name = "talks"
    url = "/talks/"
    weight = 3
  [[menu.main]]
    name = "research"
    url = "/research/"
    weight = 4
  [[menu.main]]
    name = "books"
    url = "/books/"
    weight = 5
  [[menu.main]]
    name = "about"
    url = "/about/"
    weight = 6

[outputs]
  home = ["HTML", "RSS"]
  section = ["HTML", "RSS"]

[outputFormats]
  [outputFormats.RSS]
    mediaType = "application/rss+xml"
    baseName = "feed"
```

- [ ] **Step 3: Create required directories**

```bash
mkdir -p content/posts layouts/_default layouts/posts layouts/partials static/assets/media assets/css .github/workflows
```

- [ ] **Step 4: Add .superpowers/ to .gitignore**

```bash
echo ".superpowers/" >> .gitignore
echo "public/" >> .gitignore
echo "resources/" >> .gitignore
```

- [ ] **Step 5: Verify Hugo recognizes the project**

```bash
hugo config | head -5
```

Expected: Shows the baseURL and title from hugo.toml.

- [ ] **Step 6: Commit**

```bash
git add hugo.toml .gitignore
git commit -m "feat: initialize Hugo project with config"
```

---

## Task 2: Create Base Layout and CSS

**Files:**
- Create: `layouts/_default/baseof.html`
- Create: `layouts/partials/head.html`
- Create: `layouts/partials/nav.html`
- Create: `layouts/partials/footer.html`
- Create: `assets/css/main.css`

- [ ] **Step 1: Create `assets/css/main.css`**

The complete stylesheet for the site. Plus Jakarta Sans from Google Fonts, single-column centered layout, clean typography.

```css
/* === Reset === */
*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

/* === Base === */
html {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

body {
  font-family: 'Plus Jakarta Sans', ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  font-size: 16px;
  line-height: 1.75;
  color: #444;
  background: #fff;
}

a {
  color: #000;
  text-decoration: underline;
  text-underline-offset: 3px;
  text-decoration-thickness: 1px;
}

a:hover {
  color: #000;
  text-decoration-thickness: 2px;
}

/* === Layout === */
.container {
  max-width: 640px;
  margin: 0 auto;
  padding: 48px 24px;
}

/* === Skip Link (Accessibility) === */
.skip-link {
  position: absolute;
  top: -100%;
  left: 0;
  padding: 8px 16px;
  background: #000;
  color: #fff;
  text-decoration: none;
  z-index: 100;
}

.skip-link:focus {
  top: 0;
}

/* === Navigation === */
.nav {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  margin-bottom: 56px;
  flex-wrap: wrap;
  gap: 12px;
}

.nav-name {
  font-weight: 700;
  font-size: 20px;
  color: #000;
  letter-spacing: -0.3px;
  text-decoration: none;
}

.nav-name:hover {
  text-decoration: none;
}

.nav-title {
  color: #999;
  font-size: 18px;
  font-weight: 400;
}

.nav-links {
  display: flex;
  gap: 24px;
  font-size: 14px;
  list-style: none;
}

.nav-links a {
  color: #555;
  font-weight: 450;
  text-decoration: none;
}

.nav-links a:hover {
  color: #000;
}

.nav-links a.active {
  font-weight: 600;
  color: #000;
}

/* === Hero / Homepage === */
.hero-static {
  font-size: 27px;
  font-weight: 700;
  color: #000;
  line-height: 1.35;
  letter-spacing: -0.5px;
  margin-bottom: 6px;
}

.hero-flip-container {
  height: 38px;
  overflow: hidden;
  margin-bottom: 24px;
}

.hero-flip-list {
  list-style: none;
  animation: flipDown 12s cubic-bezier(0.23, 1, 0.32, 1) infinite;
}

.hero-flip-list li {
  height: 38px;
  display: flex;
  align-items: center;
  font-size: 27px;
  font-weight: 700;
  letter-spacing: -0.5px;
  line-height: 1.35;
  color: #000;
}

@keyframes flipDown {
  0%, 20%   { transform: translateY(0); }
  25%, 45%  { transform: translateY(-38px); }
  50%, 70%  { transform: translateY(-76px); }
  75%, 95%  { transform: translateY(-114px); }
  100%      { transform: translateY(-152px); }
}

.bio {
  font-size: 15px;
  color: #555;
  line-height: 1.75;
  margin-bottom: 12px;
}

.bio a {
  color: #000;
}

.bio strong {
  font-weight: 600;
  color: #000;
}

.personal {
  font-size: 14px;
  color: #666;
  margin-bottom: 44px;
}

/* === Sections === */
.section-divider {
  border-top: 1px solid #eee;
  padding-top: 28px;
  margin-bottom: 36px;
}

.section-label {
  font-weight: 600;
  font-size: 12px;
  color: #000;
  margin-bottom: 20px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

/* === Post List (Writing Page + Homepage) === */
.post-item {
  margin-bottom: 16px;
}

.post-row {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
}

.post-title {
  font-size: 15px;
  color: #000;
  font-weight: 500;
  text-decoration: none;
}

.post-title:hover {
  text-decoration: underline;
  text-underline-offset: 3px;
}

.post-external-badge {
  font-size: 11px;
  color: #999;
  font-weight: 400;
  margin-left: 6px;
}

.post-date {
  font-size: 13px;
  color: #999;
  white-space: nowrap;
  margin-left: 20px;
}

.post-cat {
  font-size: 13px;
  color: #666;
  margin-top: 3px;
}

.post-desc {
  font-size: 13px;
  color: #888;
  font-style: italic;
  margin-top: 2px;
}

.year-label {
  font-weight: 600;
  font-size: 13px;
  color: #999;
  margin-bottom: 14px;
  margin-top: 28px;
}

.year-label:first-child {
  margin-top: 0;
}

.all-link {
  font-size: 14px;
  color: #555;
  margin-top: 16px;
}

.all-link a {
  color: #555;
}

/* === Category Pills === */
.pill-row {
  display: flex;
  gap: 8px;
  margin-bottom: 28px;
  flex-wrap: wrap;
}

.pill {
  padding: 4px 12px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: 500;
  text-decoration: none;
  cursor: pointer;
  border: none;
  background: #f3f3f3;
  color: #555;
}

.pill.active {
  background: #000;
  color: #fff;
}

/* === Project List === */
.project-row {
  margin-bottom: 14px;
  display: flex;
  gap: 20px;
}

.project-name {
  font-weight: 600;
  font-size: 14px;
  color: #000;
  min-width: 155px;
  text-decoration: none;
}

.project-name:hover {
  text-decoration: underline;
  text-underline-offset: 3px;
}

.project-desc {
  font-size: 14px;
  color: #666;
}

/* === Talks === */
.talk-item {
  margin-bottom: 20px;
  padding-bottom: 20px;
  border-bottom: 1px solid #f0f0f0;
}

.talk-item:last-child {
  border-bottom: none;
}

.talk-title {
  font-size: 15px;
  font-weight: 600;
  color: #000;
  margin-bottom: 4px;
}

.talk-meta {
  font-size: 13px;
  color: #666;
  margin-bottom: 6px;
}

.talk-links {
  display: flex;
  gap: 12px;
  font-size: 12px;
}

.talk-links a {
  color: #555;
}

/* === Research / Publications === */
.pub-item {
  margin-bottom: 20px;
  padding-bottom: 20px;
  border-bottom: 1px solid #f0f0f0;
}

.pub-item:last-child {
  border-bottom: none;
}

.pub-title {
  font-size: 15px;
  font-weight: 600;
  color: #000;
  margin-bottom: 4px;
}

.pub-authors {
  font-size: 13px;
  color: #666;
  margin-bottom: 4px;
}

.pub-venue {
  font-size: 13px;
  color: #888;
  font-style: italic;
}

.pub-links {
  display: flex;
  gap: 12px;
  font-size: 12px;
  margin-top: 6px;
}

.pub-links a {
  color: #555;
}

/* === About Page === */
.about-section {
  margin-bottom: 28px;
}

.about-section h3 {
  font-size: 14px;
  font-weight: 600;
  color: #000;
  margin-bottom: 10px;
}

.about-text {
  font-size: 15px;
  color: #444;
  line-height: 1.75;
}

.timeline-item {
  display: flex;
  gap: 16px;
  margin-bottom: 12px;
  font-size: 14px;
}

.timeline-date {
  color: #999;
  min-width: 110px;
  font-size: 13px;
}

.timeline-content {
  color: #444;
}

.timeline-content strong {
  color: #000;
  font-weight: 600;
}

/* === Books === */
.book-category {
  margin-bottom: 20px;
}

.book-category-label {
  font-weight: 600;
  font-size: 11px;
  color: #999;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 10px;
  margin-top: 16px;
}

.book-item {
  margin-bottom: 8px;
  font-size: 14px;
}

.book-title {
  color: #000;
  font-weight: 500;
}

.book-author {
  color: #888;
}

/* === Blog Post (Single) === */
.post-meta {
  font-size: 13px;
  color: #999;
  margin-bottom: 8px;
}

.post-header h1 {
  font-size: 28px;
  font-weight: 700;
  color: #000;
  line-height: 1.3;
  letter-spacing: -0.5px;
  margin: 0 0 24px 0;
}

.post-content {
  font-size: 15px;
  color: #444;
  line-height: 1.8;
}

.post-content h2 {
  font-size: 22px;
  font-weight: 700;
  color: #000;
  margin: 32px 0 16px;
}

.post-content h3 {
  font-size: 18px;
  font-weight: 600;
  color: #000;
  margin: 28px 0 12px;
}

.post-content h4 {
  font-size: 16px;
  font-weight: 600;
  color: #000;
  margin: 24px 0 8px;
}

.post-content p {
  margin-bottom: 16px;
}

.post-content ul, .post-content ol {
  margin-bottom: 16px;
  padding-left: 24px;
}

.post-content li {
  margin-bottom: 6px;
}

.post-content blockquote {
  border-left: 3px solid #e0e0e0;
  padding-left: 16px;
  margin: 16px 0;
  color: #666;
  font-style: italic;
}

.post-content img {
  max-width: 100%;
  height: auto;
  border-radius: 6px;
  margin: 16px 0;
}

.post-content pre {
  background: #f8f8f8;
  border: 1px solid #eee;
  border-radius: 6px;
  padding: 16px;
  overflow-x: auto;
  margin: 16px 0;
  font-size: 13px;
  line-height: 1.5;
}

.post-content code {
  font-family: 'JetBrains Mono', ui-monospace, 'Cascadia Code', 'Fira Code', monospace;
  font-size: 0.9em;
}

.post-content p code, .post-content li code {
  background: #f3f3f3;
  padding: 2px 6px;
  border-radius: 3px;
}

.post-content a {
  color: #000;
  text-decoration: underline;
  text-underline-offset: 3px;
}

.post-tags {
  margin-top: 32px;
  padding-top: 20px;
  border-top: 1px solid #eee;
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.post-tag {
  background: #f3f3f3;
  padding: 3px 10px;
  border-radius: 4px;
  font-size: 12px;
  color: #666;
  text-decoration: none;
}

.post-tag:hover {
  background: #e8e8e8;
  text-decoration: none;
}

/* === Footer === */
.footer {
  border-top: 1px solid #eee;
  padding-top: 20px;
  margin-top: 36px;
}

.footer-links {
  display: flex;
  gap: 24px;
  font-size: 13px;
  margin-bottom: 8px;
}

.footer-links a {
  color: #888;
  text-decoration: none;
}

.footer-links a:hover {
  color: #000;
}

.footer-love {
  font-size: 12px;
  color: #bbb;
}

/* === Responsive === */
@media (max-width: 640px) {
  .container {
    padding: 32px 20px;
  }

  .nav {
    margin-bottom: 40px;
  }

  .nav-links {
    gap: 16px;
    font-size: 13px;
  }

  .hero-static, .hero-flip-list li {
    font-size: 22px;
  }

  .hero-flip-container {
    height: 32px;
  }

  .hero-flip-list li {
    height: 32px;
  }

  @keyframes flipDown {
    0%, 20%   { transform: translateY(0); }
    25%, 45%  { transform: translateY(-32px); }
    50%, 70%  { transform: translateY(-64px); }
    75%, 95%  { transform: translateY(-96px); }
    100%      { transform: translateY(-128px); }
  }

  .post-row {
    flex-direction: column;
    gap: 2px;
  }

  .post-date {
    margin-left: 0;
  }

  .project-row {
    flex-direction: column;
    gap: 4px;
  }

  .timeline-item {
    flex-direction: column;
    gap: 2px;
  }

  .post-header h1 {
    font-size: 24px;
  }
}

/* === Focus Indicators (Accessibility) === */
a:focus-visible, button:focus-visible {
  outline: 2px solid #000;
  outline-offset: 2px;
}
```

- [ ] **Step 2: Create `layouts/partials/head.html`**

```html
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>{{ if .IsHome }}{{ .Site.Title }}{{ else }}{{ .Title }} | {{ .Site.Title }}{{ end }}</title>
<meta name="description" content="{{ with .Description }}{{ . }}{{ else }}{{ .Site.Params.bio }}{{ end }}">
<meta name="author" content="{{ .Site.Params.author }}">
<link rel="canonical" href="{{ .Permalink }}">

<!-- Open Graph -->
<meta property="og:title" content="{{ if .IsHome }}{{ .Site.Title }}{{ else }}{{ .Title }}{{ end }}">
<meta property="og:description" content="{{ with .Description }}{{ . }}{{ else }}{{ .Site.Params.bio }}{{ end }}">
<meta property="og:url" content="{{ .Permalink }}">
<meta property="og:site_name" content="{{ .Site.Title }}">
<meta property="og:type" content="{{ if .IsPage }}article{{ else }}website{{ end }}">

<!-- Twitter Card -->
<meta name="twitter:card" content="summary">
<meta name="twitter:site" content="@{{ .Site.Params.twitter }}">
<meta name="twitter:title" content="{{ if .IsHome }}{{ .Site.Title }}{{ else }}{{ .Title }}{{ end }}">
<meta name="twitter:description" content="{{ with .Description }}{{ . }}{{ else }}{{ .Site.Params.bio }}{{ end }}">

<!-- Fonts -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700&display=swap" rel="stylesheet">

<!-- CSS -->
{{ $css := resources.Get "css/main.css" | minify | fingerprint }}
<link rel="stylesheet" href="{{ $css.RelPermalink }}" integrity="{{ $css.Data.Integrity }}">

<!-- Favicons -->
<link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png">
<link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png">
<link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png">
<link rel="manifest" href="/site.webmanifest">
<meta name="theme-color" content="#ffffff">

<!-- RSS -->
{{ range .AlternativeOutputFormats -}}
  {{ printf `<link rel="%s" type="%s" href="%s" title="%s" />` .Rel .MediaType.Type .Permalink (printf "%s - %s" $.Site.Title .Name) | safeHTML }}
{{ end -}}

<!-- Google Analytics -->
{{ if .Site.Params.googleAnalytics }}
<script async src="https://www.googletagmanager.com/gtag/js?id={{ .Site.Params.googleAnalytics }}"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', '{{ .Site.Params.googleAnalytics }}');
</script>
{{ end }}
```

- [ ] **Step 3: Create `layouts/partials/nav.html`**

```html
<nav class="nav" aria-label="Main navigation">
  <a href="/" class="nav-name">{{ .Site.Title }}</a>
  {{ if .Params.navTitle }}
  <span class="nav-title">— {{ .Params.navTitle }}</span>
  {{ end }}
  <ul class="nav-links">
    {{ range .Site.Menus.main }}
    <li>
      <a href="{{ .URL }}"{{ if eq $.RelPermalink .URL }} class="active" aria-current="page"{{ end }}>{{ .Name }}</a>
    </li>
    {{ end }}
  </ul>
</nav>
```

- [ ] **Step 4: Create `layouts/partials/footer.html`**

```html
<footer class="footer" role="contentinfo">
  <div class="footer-links">
    <a href="https://github.com/{{ .Site.Params.github }}" rel="me">GitHub</a>
    <a href="https://twitter.com/{{ .Site.Params.twitter }}" rel="me">Twitter</a>
    <a href="https://linkedin.com/in/{{ .Site.Params.linkedin }}" rel="me">LinkedIn</a>
    <a href="mailto:{{ .Site.Params.email }}">Email</a>
    <a href="{{ .Site.Params.scholar }}">Google Scholar</a>
  </div>
  <div class="footer-love">with ♥ from Waterloo, Canada</div>
</footer>
```

- [ ] **Step 5: Create `layouts/_default/baseof.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  {{ partial "head.html" . }}
  {{ block "schema" . }}{{ end }}
</head>
<body>
  <a href="#main-content" class="skip-link">Skip to content</a>
  <div class="container">
    {{ partial "nav.html" . }}
    <main id="main-content">
      {{ block "main" . }}{{ end }}
    </main>
    {{ partial "footer.html" . }}
  </div>
</body>
</html>
```

- [ ] **Step 6: Test with a blank homepage**

Create a minimal `layouts/index.html`:

```html
{{ define "main" }}
<p>Site is working.</p>
{{ end }}
```

Create a minimal `content/_index.md`:

```markdown
---
title: "Shivasurya"
---
```

Run:

```bash
hugo server -D
```

Expected: Site loads at localhost:1313 with "Site is working." text, Plus Jakarta Sans font loaded, nav bar showing all 6 links, footer with social links.

- [ ] **Step 7: Commit**

```bash
git add assets/css/main.css layouts/ content/_index.md
git commit -m "feat: add base layout, CSS, nav, and footer"
```

---

## Task 3: Build Homepage with Hero Animation

**Files:**
- Create: `layouts/partials/hero.html`
- Create: `layouts/partials/schema-person.html`
- Modify: `layouts/index.html`
- Modify: `content/_index.md`

- [ ] **Step 1: Create `layouts/partials/hero.html`**

```html
<section aria-label="Introduction">
  <div class="hero-static">Software security engineer who</div>
  <div class="hero-flip-container" aria-hidden="true">
    <ul class="hero-flip-list">
      <li>believes open-source levels the playing field.</li>
      <li>hunts vulnerabilities in code with precision.</li>
      <li>builds static code analysis engines.</li>
      <li>ships security stuff at scale.</li>
      <li>believes open-source levels the playing field.</li>
    </ul>
  </div>

  <p class="bio">
    Currently at <a href="https://carta.com">Carta</a>. Previously Dropbox, Yelp. Building <a href="https://codepathfinder.dev"><strong>code-pathfinder</strong></a> — an open-source static code analysis engine — and writing about finding vulnerabilities in the wild.
  </p>
  <p class="personal">Bikes long distances, reads on Kindle, and embraces value investing. Based in Waterloo, Canada.</p>
</section>
```

- [ ] **Step 2: Create `layouts/partials/schema-person.html`**

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Person",
  "name": "Shivasurya Sankarapandian",
  "alternateName": "Shivasurya",
  "url": "{{ .Site.BaseURL }}",
  "jobTitle": "Senior Software Engineer, Security",
  "worksFor": {
    "@type": "Organization",
    "name": "Carta"
  },
  "sameAs": [
    "https://github.com/{{ .Site.Params.github }}",
    "https://twitter.com/{{ .Site.Params.twitter }}",
    "https://linkedin.com/in/{{ .Site.Params.linkedin }}",
    "{{ .Site.Params.scholar }}"
  ],
  "knowsAbout": ["Software Security", "Static Code Analysis", "Application Security", "Software Engineering"]
}
</script>
```

- [ ] **Step 3: Update `layouts/index.html`**

```html
{{ define "schema" }}
{{ partial "schema-person.html" . }}
{{ end }}

{{ define "main" }}

{{ partial "hero.html" . }}

<!-- Recent Writing -->
<div class="section-divider">
  <div class="section-label">Recent Writing</div>
  {{ range first 5 (where .Site.RegularPages "Section" "posts") }}
  <div class="post-item">
    <div class="post-row">
      {{ if .Params.external_url }}
      <a href="{{ .Params.external_url }}" class="post-title" target="_blank" rel="noopener">
        {{ .Title }} <span class="post-external-badge">↗ {{ urls.Parse .Params.external_url | .Host }}</span>
      </a>
      {{ else }}
      <a href="{{ .RelPermalink }}" class="post-title">{{ .Title }}</a>
      {{ end }}
      <span class="post-date">{{ .Date.Format "Jan 2, 2006" }}</span>
    </div>
    {{ with .Params.categories }}
    <div class="post-cat">{{ delimit . " · " }}</div>
    {{ end }}
  </div>
  {{ end }}
  <div class="all-link"><a href="/writing/">All writing</a> →</div>
</div>

<!-- Notable Projects -->
<div class="section-divider">
  <div class="section-label">Notable Projects</div>
  <div class="project-row">
    <a href="https://codepathfinder.dev" class="project-name" target="_blank" rel="noopener">code-pathfinder ↗</a>
    <span class="project-desc">Static code analysis engine for modern security teams</span>
  </div>
  <div class="project-row">
    <a href="https://github.com/shivasurya/secureflow-cli" class="project-name" target="_blank" rel="noopener">SecureFlow CLI ↗</a>
    <span class="project-desc">AI-powered security scanning with 12+ model support</span>
  </div>
</div>

{{ end }}
```

- [ ] **Step 4: Run Hugo and verify homepage**

```bash
hugo server -D
```

Expected: Homepage shows hero with flip animation, empty "Recent Writing" section (no posts migrated yet), projects section, footer.

- [ ] **Step 5: Commit**

```bash
git add layouts/index.html layouts/partials/hero.html layouts/partials/schema-person.html content/_index.md
git commit -m "feat: build homepage with hero animation and schema"
```

---

## Task 4: Migrate All 46 Blog Posts

**Files:**
- Create: `content/posts/` (46 files migrated from `_posts/`)
- Create: migration script (temporary)

- [ ] **Step 1: Write migration script**

Create a temporary script `migrate.sh` that converts Jekyll posts to Hugo format:

```bash
#!/bin/bash
# Migrate Jekyll posts to Hugo
# Converts frontmatter: removes layout/comments, adds date from filename

SRC="_posts"
DEST="content/posts"

mkdir -p "$DEST"

for file in "$SRC"/*.md; do
  filename=$(basename "$file")

  # Extract date from filename (YYYY-MM-DD)
  date=$(echo "$filename" | grep -oP '^\d{4}-\d{2}-\d{2}')

  # Extract slug (everything after the date, minus .md)
  slug=$(echo "$filename" | sed "s/^${date}-//" | sed 's/\.md$//')

  # Remove leading dashes from slug if any
  slug=$(echo "$slug" | sed 's/^-*//')

  # Read the file
  content=$(cat "$file")

  # Extract frontmatter (between --- markers)
  frontmatter=$(echo "$content" | sed -n '/^---$/,/^---$/p' | head -n -1 | tail -n +2)
  body=$(echo "$content" | sed -n '/^---$/,/^---$/!p' | tail -n +1)
  # More robust: get everything after the second ---
  body=$(awk 'BEGIN{c=0} /^---$/{c++; if(c==2){found=1; next}} found{print}' "$file")

  # Build new frontmatter
  {
    echo "---"
    # Keep title
    echo "$frontmatter" | grep "^title:"
    # Add date
    echo "date: ${date}"
    # Keep categories (if exists)
    echo "$frontmatter" | grep "^categories:" || true
    # Keep tags (if exists)
    echo "$frontmatter" | grep "^tags:" || true
    # Keep description (if exists)
    echo "$frontmatter" | grep "^description:" || true
    # Add slug
    echo "slug: \"${slug}\""
    echo "---"
    echo ""
    echo "$body"
  } > "$DEST/${filename}"

  echo "Migrated: $filename"
done

echo ""
echo "Done. Migrated $(ls "$DEST"/*.md | wc -l) posts."
```

- [ ] **Step 2: Run migration**

```bash
chmod +x migrate.sh
bash migrate.sh
```

Expected: "Done. Migrated 46 posts."

- [ ] **Step 3: Verify migration — spot check 3 posts**

Check a recent post, an old post, and one with special characters:

```bash
head -15 content/posts/2025-11-07-django-sql-injection-CVE-2025-64459.md
head -15 content/posts/2020-11-05-securing-express-server-part-1.md
head -15 content/posts/2025-12-20-2025-wrapped.md
```

Verify each has: title, date, categories, tags, description, slug — and NO layout or comments fields.

- [ ] **Step 4: Verify URL generation**

```bash
hugo list all 2>/dev/null | head -10
```

Check that URLs follow `/:year/:month/:day/:slug/` pattern.

- [ ] **Step 5: Copy media assets**

```bash
cp -r assets/media/* static/assets/media/
```

- [ ] **Step 6: Verify posts render**

```bash
hugo server -D
```

Navigate to a known URL like `/2025/11/07/django-sql-injection-CVE-2025-64459/` and verify it loads.

- [ ] **Step 7: Remove migration script and commit**

```bash
rm migrate.sh
git add content/posts/ static/assets/media/
git commit -m "feat: migrate all 46 blog posts from Jekyll to Hugo"
```

---

## Task 5: Create Blog Post Template

**Files:**
- Create: `layouts/posts/single.html`
- Create: `layouts/partials/schema-post.html`

- [ ] **Step 1: Create `layouts/partials/schema-post.html`**

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "BlogPosting",
  "headline": "{{ .Title }}",
  "datePublished": "{{ .Date.Format "2006-01-02T15:04:05Z07:00" }}",
  "dateModified": "{{ .Lastmod.Format "2006-01-02T15:04:05Z07:00" }}",
  "author": {
    "@type": "Person",
    "name": "Shivasurya Sankarapandian",
    "url": "{{ .Site.BaseURL }}"
  },
  "description": "{{ with .Description }}{{ . }}{{ else }}{{ .Summary | plainify | truncate 160 }}{{ end }}",
  "url": "{{ .Permalink }}",
  "mainEntityOfPage": {
    "@type": "WebPage",
    "@id": "{{ .Permalink }}"
  }
}
</script>
```

- [ ] **Step 2: Create `layouts/posts/single.html`**

```html
{{ define "schema" }}
{{ partial "schema-post.html" . }}
{{ end }}

{{ define "main" }}
<article>
  <header class="post-header">
    <div class="post-meta">
      <time datetime="{{ .Date.Format "2006-01-02" }}">{{ .Date.Format "January 2, 2006" }}</time>
      {{ with .Params.categories }}· {{ delimit . ", " }}{{ end }}
    </div>
    <h1>{{ .Title }}</h1>
  </header>

  <div class="post-content">
    {{ .Content }}
  </div>

  {{ with .Params.tags }}
  <div class="post-tags" aria-label="Tags">
    {{ range . }}
    <a href="/tags/{{ . | urlize }}/" class="post-tag">{{ . }}</a>
    {{ end }}
  </div>
  {{ end }}
</article>
{{ end }}
```

- [ ] **Step 3: Verify a blog post renders correctly**

```bash
hugo server -D
```

Navigate to `/2025/11/07/django-sql-injection-CVE-2025-64459/`. Verify:
- Date and category show at top
- Title is large and bold
- Content renders with proper formatting
- Tags appear as pills at bottom
- Schema JSON-LD in page source

- [ ] **Step 4: Commit**

```bash
git add layouts/posts/single.html layouts/partials/schema-post.html
git commit -m "feat: add blog post template with schema"
```

---

## Task 6: Create Writing Page (Post List)

**Files:**
- Create: `layouts/posts/list.html`
- Create: `content/writing/_index.md`

- [ ] **Step 1: Create `content/writing/_index.md`**

We need writing to be an alias for the posts section. Instead, we'll make writing redirect to the posts list. Actually, Hugo's section-based routing means posts live at `/posts/`. We need `/writing/` to show the post list. The simplest approach: use `content/writing/_index.md` as an alias and build a custom layout.

Actually, the cleanest Hugo approach: make the posts section use `writing` as its URL by adding to `hugo.toml`:

Add to `hugo.toml` under `[permalinks]`:

```toml
[permalinks]
  posts = "/:year/:month/:day/:slug/"

[permalinks.section]
  posts = "/writing/"
```

Wait — that won't work cleanly. Instead, let's just create a standalone writing page that queries posts:

Create `content/writing/_index.md`:

```markdown
---
title: "Writing"
navTitle: "Writing"
description: "Blog posts on security, static analysis, AI, and software engineering."
layout: "writing"
url: "/writing/"
---
```

- [ ] **Step 2: Create `layouts/_default/writing.html`**

```html
{{ define "main" }}

<div style="margin-bottom: 32px;">
  <p style="font-size: 15px; color: #555; line-height: 1.7;">
    Blog posts on security, static analysis, AI, and software engineering.
  </p>
</div>

<!-- Category filter pills — static links to category pages -->
<div class="pill-row">
  <a href="/writing/" class="pill active">All</a>
  {{ $categories := slice }}
  {{ range .Site.Taxonomies.categories }}
    {{ $categories = $categories | append .Page.Title }}
  {{ end }}
  {{ range sort (.Site.Taxonomies.categories) }}
  {{ end }}
  {{ range $name, $taxonomy := .Site.Taxonomies.categories }}
  <a href="/categories/{{ $name | urlize }}/" class="pill">{{ $name | humanize }}</a>
  {{ end }}
</div>

<!-- Posts grouped by year -->
{{ $posts := where .Site.RegularPages "Section" "posts" }}
{{ range $posts.GroupByDate "2006" }}
<div class="year-label">{{ .Key }}</div>
{{ range .Pages }}
<div class="post-item">
  <div class="post-row">
    {{ if .Params.external_url }}
    <a href="{{ .Params.external_url }}" class="post-title" target="_blank" rel="noopener">
      {{ .Title }} <span class="post-external-badge">↗ {{ with urls.Parse .Params.external_url }}{{ .Host }}{{ end }}</span>
    </a>
    {{ else }}
    <a href="{{ .RelPermalink }}" class="post-title">{{ .Title }}</a>
    {{ end }}
    <span class="post-date">{{ .Date.Format "Jan 2" }}</span>
  </div>
  {{ with .Description }}
  <div class="post-desc">{{ . }}</div>
  {{ end }}
</div>
{{ end }}
{{ end }}

{{ end }}
```

- [ ] **Step 3: Verify writing page**

```bash
hugo server -D
```

Navigate to `/writing/`. Verify:
- Posts grouped by year (2025, 2024, 2023...)
- Category pills at top
- Each post shows title and date
- Descriptions show in italic where available

- [ ] **Step 4: Commit**

```bash
git add content/writing/ layouts/_default/writing.html hugo.toml
git commit -m "feat: add writing page with year-grouped posts"
```

---

## Task 7: Create Static Pages (Projects, Talks, Research, Books, About)

**Files:**
- Create: `content/projects.md`
- Create: `content/talks.md`
- Create: `content/research.md`
- Create: `content/books.md`
- Create: `content/about.md`
- Create: `layouts/_default/single.html`

- [ ] **Step 1: Create `layouts/_default/single.html`**

```html
{{ define "main" }}
<article>
  <div class="post-content">
    {{ .Content }}
  </div>
</article>
{{ end }}
```

- [ ] **Step 2: Create `content/projects.md`**

Populate with real data from the existing site and GitHub repos:

```markdown
---
title: "Projects"
navTitle: "Projects"
description: "Open-source projects and tools."
url: "/projects/"
---

Open-source tools and projects I've built.

<div class="section-divider">
<div class="section-label">Security Tooling</div>

<div class="project-row">
  <a href="https://codepathfinder.dev" class="project-name" target="_blank" rel="noopener">code-pathfinder ↗</a>
  <span class="project-desc">Static code analysis engine for modern security teams</span>
</div>

<div class="project-row">
  <a href="https://github.com/shivasurya/secureflow-cli" class="project-name" target="_blank" rel="noopener">SecureFlow CLI ↗</a>
  <span class="project-desc">AI-powered security scanning with 12+ model support</span>
</div>
</div>
```

Note: The implementer should check `https://github.com/shivasurya` for the full list of repos and populate accordingly. The above is a starting template — pull real project names and descriptions from the user's GitHub profile.

- [ ] **Step 3: Create `content/talks.md`**

```markdown
---
title: "Talks"
navTitle: "Talks"
description: "Conference talks and presentations on software security and static analysis."
url: "/talks/"
---

Conference talks, meetups, and presentations on software security, static analysis, and open-source tooling.

*Interested in having me speak? [Get in touch](mailto:shiva@shivasurya.me).*
```

Note: The implementer should ask the user to provide talk data (title, venue, date, slide/video links) — this is a placeholder structure. Talks can be added as HTML within the markdown using the same `.talk-item` CSS classes.

- [ ] **Step 4: Create `content/research.md`**

```markdown
---
title: "Research"
navTitle: "Research"
description: "Research publications and interests in software security and static code analysis."
url: "/research/"
---

Research interests: Software Security, Static Code Analysis, Software Engineering, Application Security.

<div class="section-label">Publications</div>

<div class="pub-item">
  <div class="pub-title">Detecting Exploitable Vulnerabilities in Android Applications</div>
  <div class="pub-authors">S Sankarapandian</div>
  <div class="pub-venue">University of Waterloo, 2021 · Cited by 2</div>
  <div class="pub-links">
    <a href="https://scholar.google.ca/citations?user=T1tx3TIAAAAJ">Google Scholar</a>
  </div>
</div>

<div style="border-top:1px solid #eee; padding-top:24px; margin-top:12px;">
<div class="section-label">Research Areas</div>

- Static code analysis precision
- Source-sink vulnerability detection
- Inter-procedural data flow analysis
- AI-assisted security scanning
- Android application security
- Open-source security tooling
</div>
```

- [ ] **Step 5: Create `content/books.md`**

Pull data from the existing `_posts/2024-12-19-books-i-read-2024.md`:

```markdown
---
title: "Books"
navTitle: "Books"
description: "Books I've read, grouped by year."
url: "/books/"
---

Books I've read, loosely grouped by year. Heavy Kindle reader — always open to recommendations.

## 2024

<div class="book-category">
<div class="book-category-label">Security & Programming</div>

<div class="book-item"><span class="book-title">Building a Large Language Model from Scratch</span> <span class="book-author">— Sebastian Raschka</span></div>
<div class="book-item"><span class="book-title">Building Interpreter from Scratch</span> <span class="book-author">— Robert Nystrom</span></div>
<div class="book-item"><span class="book-title">Serious Cryptography</span> <span class="book-author">— Jean-Philippe Aumasson</span></div>
<div class="book-item"><span class="book-title">Crypto Dictionary</span> <span class="book-author">— Jean-Philippe Aumasson</span></div>
<div class="book-item"><span class="book-title">Crafting Interpreters</span> <span class="book-author">— Robert Nystrom</span></div>
<div class="book-item"><span class="book-title">LLVM Cookbook</span> <span class="book-author">— Mayur Pandey</span></div>
</div>

<div class="book-category">
<div class="book-category-label">Non-Fiction</div>

<div class="book-item"><span class="book-title">Orbiting the Giant Hairball</span> <span class="book-author">— Gordon MacKenzie</span></div>
<div class="book-item"><span class="book-title">How to Fail at Almost Everything and Still Win Big</span> <span class="book-author">— Scott Adams</span></div>
<div class="book-item"><span class="book-title">Ego is the Enemy</span> <span class="book-author">— Ryan Holiday</span></div>
<div class="book-item"><span class="book-title">Outlive: The Science and Art of Longevity</span> <span class="book-author">— Peter Attia</span></div>
</div>

<div class="book-category">
<div class="book-category-label">Personal Finance</div>

<div class="book-item"><span class="book-title">Mastering the Market Cycle</span> <span class="book-author">— Howard Marks</span></div>
<div class="book-item"><span class="book-title">The Most Important Thing</span> <span class="book-author">— Howard Marks</span></div>
<div class="book-item"><span class="book-title">Your Money or Your Life</span> <span class="book-author">— Vicki Robin</span></div>
<div class="book-item"><span class="book-title">Millionaire's Fastlane</span> <span class="book-author">— MJ DeMarco</span></div>
<div class="book-item"><span class="book-title">Dhandho Investor</span> <span class="book-author">— Mohnish Pabrai</span></div>
</div>
```

- [ ] **Step 6: Create `content/about.md`**

```markdown
---
title: "About"
navTitle: "About"
description: "About Shivasurya — software security engineer based in Waterloo, Canada."
url: "/about/"
---

<div class="about-section">
<div class="about-text">

I'm a software security engineer based in Waterloo, Canada. I work at [Carta](https://carta.com) where I focus on application security. Before that, I was at Dropbox and Yelp.

</div>
</div>

<div class="about-section">
<div class="about-text">

I believe open-source levels the playing field in security. That's why I build [code-pathfinder](https://codepathfinder.dev), a static code analysis engine designed for modern security teams. I also created SecureFlow CLI, an AI-powered tool for hunting vulnerabilities at scale.

</div>
</div>

<div class="about-section">
<div class="about-text">

I write about security vulnerabilities, static analysis, and the craft of building tools that find bugs before attackers do.

</div>
</div>

<div class="about-section">

### Experience

<div class="timeline-item">
  <span class="timeline-date">2022 – present</span>
  <span class="timeline-content"><strong>Carta</strong> — Senior Software Engineer, Security</span>
</div>
<div class="timeline-item">
  <span class="timeline-date">2020 – 2022</span>
  <span class="timeline-content"><strong>Dropbox</strong> — Software Engineer</span>
</div>
<div class="timeline-item">
  <span class="timeline-date">2019 – 2020</span>
  <span class="timeline-content"><strong>Yelp</strong> — Software Engineer</span>
</div>

</div>

<div class="about-section">

### Education

<div class="timeline-item">
  <span class="timeline-date">2018 – 2020</span>
  <span class="timeline-content"><strong>University of Waterloo</strong> — MASc, Electrical & Computer Engineering</span>
</div>

</div>

<div class="about-section">

### Beyond code

<div class="about-text">

Bikes long distances, reads on Kindle, and embraces value investing. Always looking for the next great trail or book recommendation.

</div>
</div>
```

- [ ] **Step 7: Verify all pages**

```bash
hugo server -D
```

Navigate to each: `/projects/`, `/talks/`, `/research/`, `/books/`, `/about/`. Verify content renders, nav highlights the active page, footer appears.

- [ ] **Step 8: Commit**

```bash
git add content/projects.md content/talks.md content/research.md content/books.md content/about.md layouts/_default/single.html
git commit -m "feat: add projects, talks, research, books, and about pages"
```

---

## Task 8: Add LLM Discoverability

**Files:**
- Create: `static/llms.txt`
- Create: `layouts/_default/llms-full.txt`
- Create: `content/llms-full.md`

- [ ] **Step 1: Create `static/llms.txt`**

```markdown
# Shivasurya

> Software security engineer who believes open-source levels the playing field.

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
- code-pathfinder: Static code analysis engine for modern security teams (https://codepathfinder.dev)
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

- [ ] **Step 2: Create auto-generated llms-full.txt**

Create `content/llms-full.md`:

```markdown
---
title: "LLMs Full"
url: "/llms-full.txt"
layout: "llms-full"
outputs:
  - html
---
```

Create `layouts/_default/llms-full.html`:

```
{{- /* Output as plain text */ -}}
# Shivasurya — Full Content Index

> Software security engineer who believes open-source levels the playing field.

## All Blog Posts

{{ range (where .Site.RegularPages "Section" "posts") }}
### {{ .Title }}
- Date: {{ .Date.Format "2006-01-02" }}
- URL: {{ .Permalink }}
{{ with .Description }}- Description: {{ . }}{{ end }}
{{ with .Params.categories }}- Categories: {{ delimit . ", " }}{{ end }}
{{ with .Params.tags }}- Tags: {{ delimit . ", " }}{{ end }}

{{ end }}
```

Note: This outputs as HTML but the content is markdown-formatted text. For a true .txt output, the implementer should configure a custom output format in hugo.toml:

Add to `hugo.toml`:

```toml
[mediaTypes]
  [mediaTypes."text/plain"]
    suffixes = ["txt"]

[outputFormats]
  [outputFormats.TXT]
    mediaType = "text/plain"
    baseName = "llms-full"
    isPlainText = true
```

Then update the layout to use `layouts/_default/llms-full.txt` and the content frontmatter to output `TXT`.

- [ ] **Step 3: Verify files**

```bash
hugo server -D
```

Check `http://localhost:1313/llms.txt` and `http://localhost:1313/llms-full.txt` load correctly.

- [ ] **Step 4: Commit**

```bash
git add static/llms.txt content/llms-full.md layouts/_default/llms-full.html
git commit -m "feat: add LLM discoverability (llms.txt, llms-full.txt)"
```

---

## Task 9: Add Categories and Tags Pages

**Files:**
- Create: `layouts/taxonomy/list.html` (or `layouts/_default/taxonomy.html`)
- Create: `layouts/taxonomy/term.html`

- [ ] **Step 1: Enable taxonomies in `hugo.toml`**

Add to `hugo.toml`:

```toml
[taxonomies]
  category = "categories"
  tag = "tags"
```

- [ ] **Step 2: Create `layouts/_default/taxonomy.html`**

This handles both `/categories/` and `/tags/` listing pages:

```html
{{ define "main" }}
<h2 style="font-size: 22px; font-weight: 700; color: #000; margin-bottom: 24px;">
  {{ .Title }}
</h2>

{{ range .Data.Terms.Alphabetical }}
<div style="margin-bottom: 20px;">
  <h3 style="font-size: 15px; font-weight: 600; color: #000; margin-bottom: 8px;">
    <a href="{{ .Page.RelPermalink }}" style="text-decoration: none; color: #000;">
      {{ .Page.Title }} <span style="color: #999; font-weight: 400;">({{ .Count }})</span>
    </a>
  </h3>
</div>
{{ end }}
{{ end }}
```

- [ ] **Step 3: Create `layouts/_default/term.html`**

This handles individual category/tag pages like `/categories/security/`:

```html
{{ define "main" }}
<h2 style="font-size: 22px; font-weight: 700; color: #000; margin-bottom: 24px;">
  {{ .Title }}
</h2>

{{ range .Pages }}
<div class="post-item">
  <div class="post-row">
    <a href="{{ .RelPermalink }}" class="post-title">{{ .Title }}</a>
    <span class="post-date">{{ .Date.Format "Jan 2, 2006" }}</span>
  </div>
  {{ with .Description }}
  <div class="post-desc">{{ . }}</div>
  {{ end }}
</div>
{{ end }}
{{ end }}
```

- [ ] **Step 4: Verify taxonomy pages**

```bash
hugo server -D
```

Navigate to `/categories/`, `/tags/`, `/categories/security/`, `/tags/django/`. Verify they list posts correctly.

- [ ] **Step 5: Commit**

```bash
git add layouts/_default/taxonomy.html layouts/_default/term.html hugo.toml
git commit -m "feat: add categories and tags pages"
```

---

## Task 10: Set Up GitHub Actions for Deployment

**Files:**
- Create: `.github/workflows/hugo.yml`

- [ ] **Step 1: Create `.github/workflows/hugo.yml`**

```yaml
name: Deploy Hugo site to GitHub Pages

on:
  push:
    branches: ["master"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.147.0
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb

      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5

      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
          TZ: America/Toronto
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

- [ ] **Step 2: Verify the workflow file is valid**

```bash
cat .github/workflows/hugo.yml | head -5
```

- [ ] **Step 3: Commit**

```bash
git add .github/workflows/hugo.yml
git commit -m "feat: add GitHub Actions workflow for Hugo deployment"
```

---

## Task 11: Clean Up Old Jekyll Files

**Files:**
- Remove old Jekyll files that are no longer needed

- [ ] **Step 1: Identify Jekyll files to remove**

These files are Jekyll-specific and replaced by Hugo equivalents:

```
_config.yml          → hugo.toml
_includes/           → layouts/partials/
_layouts/            → layouts/
_posts/              → content/posts/ (already migrated)
index.html           → layouts/index.html
categories.html      → auto-generated by Hugo taxonomies
tags.html            → auto-generated by Hugo taxonomies
feed.xml             → auto-generated by Hugo RSS
Gemfile              → not needed (no Ruby)
Gemfile.lock         → not needed
assets/css/style.css → assets/css/main.css
assets/js/app.js     → not needed
assets/resources/    → not needed (no Bootstrap/jQuery/Font Awesome)
```

- [ ] **Step 2: Remove old files**

```bash
git rm _config.yml Gemfile Gemfile.lock index.html categories.html tags.html feed.xml
git rm -r _includes/ _layouts/ _posts/
git rm -r assets/resources/ assets/js/ assets/css/style.css
git rm -r assets/ico/ 2>/dev/null || true
```

Keep: `assets/media/` images are now in `static/assets/media/`.

- [ ] **Step 3: Verify Hugo still builds**

```bash
hugo --gc --minify
```

Expected: Build succeeds with no errors.

- [ ] **Step 4: Commit**

```bash
git add -A
git commit -m "chore: remove old Jekyll files after Hugo migration"
```

---

## Task 12: Verify URL Preservation and SEO

**Files:** None (verification only)

- [ ] **Step 1: Build the site and list all URLs**

```bash
hugo --gc --minify
find public -name "index.html" | sed 's|public||;s|/index.html||' | sort > /tmp/hugo-urls.txt
head -30 /tmp/hugo-urls.txt
```

- [ ] **Step 2: Verify all 46 post URLs match**

Check a sample of known URLs exist in the output:

```bash
grep "2025/11/07/django-sql-injection-CVE-2025-64459" /tmp/hugo-urls.txt
grep "2020/11/05/securing-express-server-part-1" /tmp/hugo-urls.txt
grep "2023/01/06/exploit-education-lab-setup" /tmp/hugo-urls.txt
grep "2025/12/20/2025-wrapped" /tmp/hugo-urls.txt
grep "2020/12/07/leetcode-xss" /tmp/hugo-urls.txt
```

All 5 should match. If any are missing, check the slug in the post frontmatter.

- [ ] **Step 3: Verify sitemap.xml**

```bash
cat public/sitemap.xml | head -30
```

Verify it contains post URLs in the correct format.

- [ ] **Step 4: Verify RSS feed**

```bash
cat public/feed.xml | head -20
```

Verify it's valid RSS with recent posts.

- [ ] **Step 5: Verify llms.txt**

```bash
cat public/llms.txt | head -10
```

- [ ] **Step 6: Spot-check meta tags on a post**

```bash
grep -A2 "og:title\|og:url\|canonical\|description" public/2025/11/07/django-sql-injection-CVE-2025-64459/index.html | head -20
```

Verify OG tags, canonical URL, and description are present and correct.

- [ ] **Step 7: No commit needed — this is verification only**

---

## Task 13: Run Lighthouse Audit

- [ ] **Step 1: Start local server**

```bash
hugo server --gc --minify &
```

- [ ] **Step 2: Run Lighthouse via Chrome DevTools or CLI**

```bash
# If lighthouse CLI is installed:
npx lighthouse http://localhost:1313 --output html --output-path ./lighthouse-report.html
npx lighthouse http://localhost:1313/writing/ --output html --output-path ./lighthouse-writing.html
npx lighthouse http://localhost:1313/2025/11/07/django-sql-injection-CVE-2025-64459/ --output html --output-path ./lighthouse-post.html
```

Target: 100/100 on Performance, Accessibility, Best Practices, SEO for all pages.

- [ ] **Step 3: Fix any issues found**

Common fixes:
- Missing alt text on images → add alt attributes
- Color contrast issues → adjust colors
- Missing lang attribute → already in baseof.html
- Render-blocking resources → fonts use `display=swap`
- Missing meta description → already in head.html

- [ ] **Step 4: Clean up and commit any fixes**

```bash
git add -A
git commit -m "fix: address Lighthouse audit issues"
```

---

## Summary

| Task | Description | Key files |
|------|-------------|-----------|
| 1 | Install Hugo, config | `hugo.toml` |
| 2 | Base layout, CSS, nav, footer | `baseof.html`, `main.css`, partials |
| 3 | Homepage with hero animation | `index.html`, `hero.html`, schema |
| 4 | Migrate 46 blog posts | `content/posts/*.md` |
| 5 | Blog post template | `posts/single.html`, schema |
| 6 | Writing page (year-grouped) | `writing.html`, category pills |
| 7 | Static pages (5 pages) | projects, talks, research, books, about |
| 8 | LLM discoverability | `llms.txt`, `llms-full.txt` |
| 9 | Categories + tags pages | taxonomy templates |
| 10 | GitHub Actions deploy | `hugo.yml` |
| 11 | Clean up Jekyll files | Remove old files |
| 12 | Verify URLs + SEO | Spot checks |
| 13 | Lighthouse audit | Fix to 100/100 |
