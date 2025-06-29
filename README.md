# Diogo Guedes Blog

A personal blog built with Hugo and the PaperMod theme, demonstrating automated deployment to GitHub Pages using GitHub Actions.

This repository showcases a complete CI/CD pipeline for a static blog, automatically deploying content changes to GitHub Pages when updates are pushed to the main branch.

**Live Blog:** [https://diogoguedes11.github.io/gh-deployment-workflow-/](https://diogoguedes11.github.io/gh-deployment-workflow-/)

**Original Project Inspiration:** https://roadmap.sh/projects/github-actions-deployment-workflow

## What this blog features:
- **Hugo Static Site Generator** with PaperMod theme
- **Automated GitHub Actions deployment** to GitHub Pages
- **Modern, responsive design** with dark/light mode toggle
- **SEO optimized** with proper meta tags and structured data
- **Fast loading** with minified assets and optimized images
- **Blog posts** about web development, deployment, and tutorials

## Project Structure
```
├── DiogoGuedesBlog/           # Hugo blog source
│   ├── content/posts/         # Blog posts
│   ├── themes/PaperMod/       # PaperMod theme
│   ├── hugo.yaml             # Hugo configuration
│   └── ...
├── .github/workflows/         # GitHub Actions workflows
├── index.html                # Simple static page (original)
└── README.md                 # This file
```

## Technologies Used
- **Hugo** - Fast static site generator
- **PaperMod** - Clean, responsive Hugo theme
- **GitHub Actions** - CI/CD for automated deployment
- **GitHub Pages** - Free hosting for static sites