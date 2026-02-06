# GitHub Pages Documentation

This folder contains the API documentation for Coinskro Pay, built with Jekyll and the "Just the Docs" theme.

## Local Development

### Prerequisites

- Ruby 2.7+ 
- Bundler (`gem install bundler`)

### Setup

```bash
cd docs
bundle install
```

### Run Locally

```bash
bundle exec jekyll serve
```

Visit `http://localhost:4000/coinskro-pay/` in your browser.

## Deploy to GitHub Pages

### Option 1: Automatic (Recommended)

1. Go to your GitHub repository
2. Navigate to **Settings** → **Pages**
3. Under "Source", select **GitHub Actions**
4. The workflow file (`.github/workflows/docs.yml`) will automatically deploy

### Option 2: Manual

1. Go to **Settings** → **Pages**
2. Under "Source", select **Deploy from a branch**
3. Select **main** branch and **/docs** folder
4. Click **Save**

Your documentation will be available at:
```
https://YOUR_USERNAME.github.io/coinskro-pay/
```

## Documentation Structure

```
docs/
├── _config.yml          # Jekyll configuration
├── Gemfile              # Ruby dependencies
├── index.md             # Home page
├── payments.md          # Creating payments guide
├── webhooks.md          # Webhooks guide
├── examples.md          # Code examples
├── testing.md           # Testing guide
├── errors.md            # Error handling
└── faq.md               # FAQ
```

## Customization

### Theme

Edit `_config.yml` to customize:
- `color_scheme`: `light` or `dark`
- `title`: Documentation title
- `description`: Site description
- `url` and `baseurl`: Your GitHub Pages URL

### Adding Pages

Create a new `.md` file with front matter:

```markdown
---
layout: default
title: Page Title
nav_order: 8
permalink: /page-url
---

# Page Title

Content here...
```

## Writing Guidelines

- Use clear, concise language
- Include code examples in multiple languages
- Add "Note" and "Warning" callouts for important info
- Test all code examples before publishing
