# shivasurya.me

Personal website and blog built with [Hugo](https://gohugo.io/).

## Local Development

```bash
# Install Hugo (https://gohugo.io/installation/)
brew install hugo  # macOS
# or
go install github.com/gohugoio/hugo@latest

# Run dev server
hugo server -D

# Build for production
hugo --gc --minify
```

## Structure

```
content/
  posts/       # Blog posts
  about.md     # About page
  books.md     # Reading list
  projects.md  # Open-source projects
  research.md  # Publications
  talks.md     # Conference talks
layouts/       # Hugo templates
assets/css/    # Stylesheet
static/        # Static files (favicon, llms.txt, robots.txt)
```

## Deployment

Deployed via GitHub Actions to GitHub Pages. Push to `master` triggers automatic build and deploy.

## License

Blog content is copyrighted. Site code is MIT licensed.
