# InfoSec & Dev Documentation Site 🚀

[![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-Deployed-brightgreen)](https://godack1905.github.io/)
[![MkDocs Material](https://img.shields.io/badge/MkDocs-Material-blue)](https://squidfunk.github.io/mkdocs-material/)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

Bilingual cybersecurity and development documentation site built with MkDocs Material. Available in English and Español.

## 🌐 Live Site
**https://godack1905.github.io/godack.github.io**

## 📁 Project Structure
```
.
├── docs/ # Documentation source files
│ ├── en/ # English documentation
│ │ ├── index.md # English homepage
│ │ ├── writeups/ # Cybersecurity writeups (EN)
│ │ └── notes/ # Technical notes (EN)
│ ├── es/ # Spanish documentation
│ │ ├── index.md # Spanish homepage
│ │ ├── writeups/ # Cybersecurity writeups (ES)
│ │ └── apuntes/ # Technical notes (ES)
│ └── images/ # Images and assets
├── mkdocs.yml # MkDocs configuration
├── .github/workflows/ # GitHub Actions workflows
│ └── deploy.yml # Auto-deployment to GitHub Pages
└── README.md # This file
```


## 🛠️ Local Development

### Prerequisites
- Python 3.8+
- pip (Python package manager)

### Installation
```
# Clone the repository
git clone https://github.com/godack1905/godack.github.io.git
cd godack.github.io

# Install MkDocs with Material theme
pip install mkdocs-material
```

### Running Locally
```
# Start the development server
mkdocs serve

# Open browser to: http://127.0.0.1:8000
```

### Building the Site
```
# Build static site to 'site/' directory
mkdocs build

# Build with verbose output
mkdocs build --verbose
```

## Deployment
Automatic Deployment
This site is automatically deployed to GitHub Pages when you push to the `main` branch:

1. Push changes to main branch
2. GitHub Actions builds and deploys automatically
3. Site updates in 1-2 minutes
