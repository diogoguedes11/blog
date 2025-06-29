---
title: "How to Deploy PaperMod Theme in Hugo"
date: 2025-06-29T12:00:00+00:00
tags: ["hugo", "papermod", "deployment", "tutorial", "static-site"]
author: "Diogo Guedes"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "A comprehensive guide on how to set up and deploy the PaperMod theme in Hugo for your static website"
disableHLJS: false
disableShare: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
cover:
    image: ""
    alt: "Hugo PaperMod Theme Deployment"
    caption: "Learn how to deploy PaperMod theme in Hugo"
    relative: false
    hidden: false
---

Hugo is one of the fastest static site generators available, and PaperMod is a clean, responsive theme that's perfect for blogs and documentation sites. In this guide, I'll walk you through the complete process of setting up and deploying a Hugo site with the PaperMod theme.

## Prerequisites

Before we begin, make sure you have the following installed:

- **Hugo Extended** (v0.112.0 or later)
- **Git** 
- **Go** (optional, for building from source)

You can verify your Hugo installation by running:
```bash
hugo version
```

## Step 1: Create a New Hugo Site

First, create a new Hugo site:

```bash
hugo new site my-papermod-blog
cd my-papermod-blog
```

This creates a basic Hugo directory structure with the necessary folders and configuration files.

## Step 2: Install PaperMod Theme

There are several ways to install PaperMod. I recommend using Git submodules for easier updates:

### Method 1: Git Submodule (Recommended)

```bash
git init
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

### Method 2: Direct Download

```bash
wget https://github.com/adityatelange/hugo-PaperMod/archive/master.zip
unzip master.zip
mv hugo-PaperMod-master themes/PaperMod
```

## Step 3: Configure Your Site

Create or update your `hugo.yaml` configuration file:

```yaml
baseURL: "https://yourdomain.com/"
title: "My Blog"
paginate: 5
theme: PaperMod

enableRobotsTXT: true
buildDrafts: false
buildFuture: false
buildExpired: false

minify:
  disableXML: true
  minifyOutput: true

params:
  env: production
  title: "My Blog"
  description: "Welcome to my blog powered by Hugo and PaperMod"
  keywords: [Blog, Portfolio, Hugo, PaperMod]
  author: "Your Name"
  
  DateFormat: "January 2, 2006"
  defaultTheme: auto
  disableThemeToggle: false

  ShowReadingTime: true
  ShowShareButtons: true
  ShowPostNavLinks: true
  ShowBreadCrumbs: true
  ShowCodeCopyButtons: true
  ShowWordCount: true
  ShowRssButtonInSectionTermList: true
  UseHugoToc: true
  disableSpecial1stPost: false
  disableScrollToTop: false
  comments: false
  hidemeta: false
  hideSummary: false
  showtoc: false
  tocopen: false

  homeInfoParams:
    Title: "Hi there 👋"
    Content: Welcome to my blog

  socialIcons:
    - name: github
      url: "https://github.com/yourusername"
    - name: linkedin
      url: "https://linkedin.com/in/yourusername"
    - name: x
      url: "https://x.com/yourusername"

menu:
  main:
    - identifier: categories
      name: categories
      url: /categories/
      weight: 10
    - identifier: tags
      name: tags
      url: /tags/
      weight: 20
    - identifier: archive
      name: archive
      url: /archives/
      weight: 30

pygmentsUseClasses: true
markup:
  highlight:
    noClasses: false
    style: github-dark
```

## Step 4: Create Your First Post

Create a new post in the `content/posts/` directory:

```bash
hugo new posts/my-first-post.md
```

Edit the post with proper front matter:

```markdown
---
title: "My First Post"
date: 2025-06-29T10:00:00+00:00
tags: ["hugo", "blog"]
author: "Your Name"
draft: false
---

Welcome to my new Hugo blog with PaperMod theme!
```

## Step 5: Test Locally

Run the Hugo development server:

```bash
hugo server -D
```

Visit `http://localhost:1313` to see your site in action.

## Step 6: Deployment Options

### Option 1: GitHub Pages with GitHub Actions

Create `.github/workflows/hugo.yml`:

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches: ["main"]
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
      HUGO_VERSION: 0.147.9
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb          
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
        run: |
          hugo \
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

### Option 2: Netlify

1. Push your code to GitHub
2. Connect your GitHub repo to Netlify
3. Set build command: `hugo --minify`
4. Set publish directory: `public`

### Option 3: Vercel

1. Install Vercel CLI: `npm i -g vercel`
2. Run `vercel` in your project directory
3. Follow the prompts

## Step 7: Customization Tips

### Custom CSS
Create `assets/css/extended/custom.css` for custom styles:

```css
:root {
    --custom-color: #your-color;
}

.custom-class {
    /* Your custom styles */
}
```

### Custom JavaScript
Create `assets/js/custom.js` for custom functionality.

### Custom Shortcodes
Create shortcodes in `layouts/shortcodes/` for reusable content blocks.

## Troubleshooting Common Issues

### Theme Not Loading
- Ensure the theme is in `themes/PaperMod/`
- Check that `theme: PaperMod` is set in your config
- Verify submodule is properly initialized

### Posts Not Showing
- Posts should be in `content/posts/` directory
- Ensure `draft: false` in front matter
- Check that date is not in the future

### Build Errors
- Update Hugo to the latest version
- Check for syntax errors in config file
- Ensure all required dependencies are installed

## Conclusion

You now have a fully functional Hugo site with the PaperMod theme! The combination of Hugo's speed and PaperMod's clean design makes for an excellent blogging platform. Remember to:

- Regularly update your theme and Hugo version
- Optimize images for better performance
- Use proper SEO practices in your content
- Test your site locally before deploying

Happy blogging! 🚀

## Useful Resources

- [Hugo Documentation](https://gohugo.io/documentation/)
- [PaperMod Theme Wiki](https://github.com/adityatelange/hugo-PaperMod/wiki)
- [Hugo Community Forum](https://discourse.gohugo.io/)
- [PaperMod GitHub Repository](https://github.com/adityatelange/hugo-PaperMod)
