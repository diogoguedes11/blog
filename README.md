# GitHub Pages Deployment with GitHub Actions

A simple project demonstrating continuous integration and continuous deployment (CI/CD) using GitHub Actions to automatically deploy a static website to GitHub Pages.

## 🎯 Project Overview

This project showcases how to set up an automated deployment pipeline that triggers whenever changes are made to the `index.html` file. The workflow automatically deploys the updated website to GitHub Pages, making it accessible via a public URL.

**Live Demo:** `https://diogoguedes11.github.io/gh-deployment-workflow-/`

## 🚀 Features

- ✅ Automated deployment to GitHub Pages
- ✅ Triggered only when `index.html` is modified
- ✅ Simple static website with "Hello, GitHub Actions!" message
- ✅ Continuous Integration/Continuous Deployment (CI/CD) pipeline
- ✅ Zero-downtime deployments

## 📁 Project Structure

```
gh-deployment-workflow/
├── .github/
│   └── workflows/
│       └── deploy.yml          # GitHub Actions workflow
├── index.html                  # Main website file
└── README.md                   # Project documentation
```

## 🛠️ Setup Instructions

### 1. Repository Setup

1. Create a new GitHub repository named `gh-deployment-workflow`
2. Clone the repository to your local machine:
   ```bash
   git clone https://github.com/yourusername/gh-deployment-workflow.git
   cd gh-deployment-workflow
   ```

### 2. Enable GitHub Pages

1. Go to your repository on GitHub
2. Navigate to **Settings** → **Pages**
3. Under **Source**, select **GitHub Actions**
4. Save the configuration

### 3. Create the Workflow

Create the GitHub Actions workflow file at `.github/workflows/deploy.yml`:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]
    paths:
      - 'index.html'  # Only trigger when index.html changes

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    permissions:
      contents: read
      pages: write
      id-token: write
    
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      
      - name: Setup Pages
        uses: actions/configure-pages@v4
      
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: '.'
      
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### 4. Test the Deployment

1. Make changes to `index.html`
2. Commit and push to the main branch:
   ```bash
   git add .
   git commit -m "Update website content"
   git push origin main
   ```
3. Check the **Actions** tab in your GitHub repository to see the workflow running
4. Once complete, visit your GitHub Pages URL to see the deployed website

## 🔧 How It Works

### Workflow Triggers
- **Event:** Push to the `main` branch
- **Condition:** Only when `index.html` file is modified
- **Action:** Automatically deploy to GitHub Pages

### Deployment Process
1. **Checkout:** Retrieves the latest code from the repository
2. **Setup Pages:** Configures the GitHub Pages environment
3. **Upload Artifact:** Packages the website files
4. **Deploy:** Publishes the site to GitHub Pages

### Permissions
The workflow requires specific permissions:
- `contents: read` - To access repository contents
- `pages: write` - To deploy to GitHub Pages
- `id-token: write` - For secure authentication

## 📊 Benefits of This Approach

- **Automation:** No manual deployment steps required
- **Efficiency:** Only deploys when necessary (when index.html changes)
- **Reliability:** Consistent deployment process every time
- **Visibility:** Full deployment history in the Actions tab
- **Speed:** Fast deployment using GitHub's infrastructure

## 🔍 Monitoring Deployments

1. **Actions Tab:** View workflow runs and their status
2. **Environment:** Check deployment history under repository settings
3. **Pages Settings:** Monitor GitHub Pages configuration and status
4. **Commit History:** See which changes triggered deployments

## 🎨 Stretch Goals & Enhancements

Consider these improvements to make the project more advanced:

### Static Site Generators
- **Hugo:** Fast static site generator written in Go
- **Jekyll:** Ruby-based generator with GitHub Pages native support
- **Astro:** Modern framework for content-focused websites
- **Next.js:** React-based framework with static export capability

### Advanced Features
- Multiple environment deployments (staging/production)
- Custom domain configuration
- Performance optimization and caching
- Automated testing before deployment
- Slack/Discord notifications on deployment success/failure

### Example Enhancement with Jekyll

```yaml
# Enhanced workflow for Jekyll
name: Deploy Jekyll site to Pages

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.1'
          bundler-cache: true
      
      - name: Build with Jekyll
        run: bundle exec jekyll build
        
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./_site
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

## 📚 Learning Outcomes

After completing this project, you'll understand:

- **GitHub Actions:** Writing and configuring CI/CD workflows
- **GitHub Pages:** Hosting static websites for free
- **Automation:** Reducing manual deployment tasks
- **Version Control:** Triggering actions based on code changes
- **YAML Configuration:** Writing workflow files
- **DevOps Practices:** Implementing continuous deployment

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test the deployment workflow
5. Submit a pull request

## 📝 License

This project is open source and available under the [MIT License](LICENSE).

## 🔗 Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [YAML Syntax Reference](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html)
- [Roadmap.sh CI/CD Guide](https://roadmap.sh/devops)

---

**Happy Deploying! 🚀**