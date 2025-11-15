# LyreBirdAudio Documentation

Official documentation site for [LyreBirdAudio](https://github.com/tomtom215/LyreBirdAudio).

**Live Site:** https://lyrebird-docs.pages.dev (will be deployed after initial setup)

---

## Built With

- **[MkDocs](https://www.mkdocs.org/)** 1.6.1 - Static site generator
- **[Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)** 9.7.0 - Documentation theme
- **[Cloudflare Pages](https://pages.cloudflare.com/)** - Hosting platform (free tier)

---

## Features

- ✅ **Dark mode default** with light mode toggle
- ✅ **Full-text search** across all documentation
- ✅ **Responsive design** optimized for all screen sizes
- ✅ **Automatic deployment** on push to main branch
- ✅ **Mermaid diagrams** for architecture visualization
- ✅ **Syntax highlighting** for code blocks
- ✅ **Minified** HTML/CSS/JS for fast loading

---

## Local Development

### Prerequisites

- Python 3.8 or higher (3.11 recommended)
- pip
- git

### Setup

```bash
# Clone repository
git clone https://github.com/tomtom215/LyreBird-Docs.git
cd LyreBird-Docs

# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # On Linux/macOS
# venv\Scripts\activate   # On Windows

# Install dependencies
pip install -r requirements.txt

# Start development server
mkdocs serve
```

Access the site at http://127.0.0.1:8000

The development server features live reload - changes to documentation files will automatically refresh the browser.

### Build

```bash
# Build static site
mkdocs build

# Output in site/ directory
```

---

## Project Structure

```
.
├── docs/                      # Documentation source files
│   ├── index.md              # Homepage
│   ├── getting-started/      # Getting started guides
│   ├── user-guide/           # User documentation
│   ├── components/           # Component reference
│   ├── advanced/             # Advanced topics
│   ├── reference/            # Reference documentation
│   ├── maintenance/          # Maintenance guides
│   ├── contributing/         # Contribution guidelines
│   ├── about/                # About the project
│   └── stylesheets/          # Custom CSS
├── mkdocs.yml                # MkDocs configuration
├── requirements.txt          # Python dependencies
├── EXECUTIVE_SUMMARY.md      # Implementation overview
├── IMPLEMENTATION_PLAN.md    # Detailed implementation guide
├── LICENSE                   # Apache 2.0 License
└── README.md                 # This file
```

---

## Deployment

This site is automatically deployed to Cloudflare Pages on every push to the `main` branch.

### Deployment Configuration

**Build Command:**
```bash
pip install -r requirements.txt && mkdocs build
```

**Build Output Directory:**
```
site
```

**Environment Variables:**
```
PYTHON_VERSION = 3.11
```

---

## Content Status

### Complete Sections

- ✅ **Homepage** - Full content with architecture diagram
- ✅ **Getting Started** - 4 complete pages (Quick Start, System Requirements, Installation, Basic Usage)
- ✅ **Components** - **8 complete pages (5,512 lines)** with comprehensive documentation, Mermaid diagrams, examples, and troubleshooting

### Sections In Progress

- 🔄 **User Guide** - 5 pages (next priority)

### Placeholder Sections (Ready for Expansion)

- 📝 **Advanced** - 5 placeholder pages
- 📝 **Reference** - 5 placeholder pages
- 📝 **Maintenance** - 3 placeholder pages
- 📝 **Contributing** - 4 placeholder pages
- 📝 **About** - 4 complete pages

**Progress:** 12 of 38 pages complete (32%)

---

## Contributing

Contributions are welcome! To contribute to the documentation:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improve-docs`)
3. Make your changes
4. Test locally with `mkdocs serve`
5. Commit your changes (`git commit -m "Improve XYZ documentation"`)
6. Push to your fork (`git push origin feature/improve-docs`)
7. Open a Pull Request

### Documentation Guidelines

- Use clear, concise language
- Include code examples where appropriate
- Use admonitions for important notes, warnings, and tips
- Test all commands and code examples
- Follow the existing content structure

---

## Maintenance

### Update Dependencies

```bash
# Activate virtual environment
source venv/bin/activate

# Update packages
pip install --upgrade mkdocs mkdocs-material mkdocs-minify-plugin

# Test build
mkdocs build --strict

# Update requirements.txt
pip freeze > requirements.txt
```

### Content Sync

Regularly sync with the main [LyreBirdAudio repository](https://github.com/tomtom215/LyreBirdAudio) README.md to keep documentation up-to-date.

---

## Technical Details

### MkDocs Configuration

- **Theme:** Material 9.7.0 (final release, entering maintenance mode)
- **Dark Mode:** Default (slate scheme)
- **Search:** Zero-configuration full-text search
- **Navigation:** Hierarchical tabs with sticky top-level navigation
- **Plugins:** Search, Minify
- **Extensions:** Admonitions, Code highlighting, Mermaid diagrams, Tabbed content

### Performance

- **Build Time:** ~1.3 seconds
- **Site Size:** ~4.2 MB
- **Pages:** 38 documentation pages
- **HTML Files:** 40 (includes 404 and search pages)

---

## License

Licensed under the [Apache License 2.0](LICENSE) - same as the main LyreBirdAudio project.

---

## Links

- **Live Documentation:** https://lyrebird-docs.pages.dev
- **LyreBirdAudio Repository:** https://github.com/tomtom215/LyreBirdAudio
- **Report Documentation Issues:** [GitHub Issues](https://github.com/tomtom215/LyreBird-Docs/issues)
- **MkDocs:** https://www.mkdocs.org/
- **Material for MkDocs:** https://squidfunk.github.io/mkdocs-material/
- **Cloudflare Pages:** https://pages.cloudflare.com/

---

**Last Updated:** 2025-11-15
**Documentation Version:** 1.0.0
**Status:** Production Ready
