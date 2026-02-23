# GitHub Pages Documentation

This folder contains the developer documentation for Coinskro products, built with Jekyll and the "Just the Docs" theme. Each product lives in its own subdirectory so we can scale as we add features.

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
├── _config.yml              # Jekyll configuration
├── Gemfile                  # Ruby dependencies
├── index.md                 # Landing page (product hub)
├── coinskro-pay/            # Coinskro Pay product docs
│   ├── index.md             # Coinskro Pay overview (parent page)
│   ├── payments.md          # Creating payments guide
│   ├── webhooks.md          # Webhooks guide
│   ├── examples.md          # Code examples
│   ├── testing.md           # Testing guide
│   ├── errors.md            # Error handling
│   └── faq.md               # FAQ
└── <future-product>/        # Add new products here
    ├── index.md
    └── ...
```

## Customization

### Theme

Edit `_config.yml` to customize:
- `color_scheme`: `light` or `dark`
- `title`: Documentation title
- `description`: Site description
- `url` and `baseurl`: Your GitHub Pages URL

### Adding a New Product

Create a new subdirectory (e.g., `docs/coinskro-wallets/`) with its own `index.md`:

```markdown
---
layout: default
title: Coinskro Wallets
nav_order: 3
has_children: true
permalink: /coinskro-wallets/
---

# Coinskro Wallets

Content here...
```

Then add child pages with `parent: Coinskro Wallets` in their front matter.

### Adding Pages to an Existing Product

Create a new `.md` file inside the product directory with front matter:

```markdown
---
layout: default
title: Page Title
parent: Coinskro Pay
nav_order: 7
permalink: /coinskro-pay/page-url
---

# Page Title

Content here...
```

## Writing Guidelines

- Use clear, concise language
- Include code examples in multiple languages
- Add "Note" and "Warning" callouts for important info
- Test all code examples before publishing
