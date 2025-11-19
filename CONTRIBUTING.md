# Contributing to LyreBirdAudio Documentation

Thank you for your interest in contributing to the LyreBirdAudio documentation! This guide will help you get started.

## Code of Conduct

This project adheres to a Code of Conduct. By participating, you are expected to uphold this code. Please report unacceptable behavior to the project maintainers.

## How Can I Contribute?

### Reporting Documentation Issues

If you find errors, unclear explanations, or missing information:

1. Check if the issue already exists in [GitHub Issues](https://github.com/tomtom215/LyreBird-Docs/issues)
2. If not, create a new issue with:
   - **Title**: Clear, descriptive summary
   - **Description**: What's wrong and where you found it
   - **Expected**: What the documentation should say
   - **Actual**: What it currently says
   - **Page**: Link or file path to the affected page

### Suggesting Enhancements

Have ideas for improving the documentation?

1. Open an issue with the "enhancement" label
2. Describe the improvement and why it would be valuable
3. Include examples or mockups if applicable

### Submitting Documentation Changes

#### Prerequisites

- Python 3.8 or higher (3.11 recommended)
- git
- A GitHub account

#### Development Workflow

1. **Fork the Repository**
   ```bash
   # Click "Fork" on GitHub, then clone your fork
   git clone https://github.com/YOUR-USERNAME/LyreBird-Docs.git
   cd LyreBird-Docs
   ```

2. **Set Up Development Environment**
   ```bash
   # Create virtual environment
   python3 -m venv venv
   source venv/bin/activate  # On Linux/macOS
   # venv\Scripts\activate   # On Windows

   # Install dependencies
   pip install -r requirements.txt
   ```

3. **Create a Feature Branch**
   ```bash
   git checkout -b feature/improve-installation-docs
   ```

4. **Make Your Changes**
   - Edit markdown files in the `docs/` directory
   - Follow the [Documentation Guidelines](#documentation-guidelines) below
   - Test locally with live reload:
     ```bash
     mkdocs serve
     ```
   - View at http://127.0.0.1:8000

5. **Test Your Changes**
   ```bash
   # Build with strict mode (catches broken links, missing files)
   mkdocs build --strict

   # If build succeeds with no warnings, you're good to go
   ```

6. **Commit Your Changes**
   ```bash
   git add .
   git commit -m "Improve installation documentation clarity"
   ```

   **Commit Message Guidelines:**
   - Use present tense ("Add feature" not "Added feature")
   - Use imperative mood ("Fix typo" not "Fixes typo")
   - Start with a verb (Add, Fix, Update, Remove, Improve)
   - Keep first line under 72 characters
   - Add detailed description if needed

7. **Push to Your Fork**
   ```bash
   git push origin feature/improve-installation-docs
   ```

8. **Open a Pull Request**
   - Go to the original repository on GitHub
   - Click "New Pull Request"
   - Select your fork and branch
   - Fill in the PR template with:
     - **What**: Description of changes
     - **Why**: Reason for the changes
     - **Testing**: How you verified the changes
   - Request review from maintainers

## Documentation Guidelines

### Writing Style

- **Clarity**: Use simple, clear language
- **Conciseness**: Be brief but complete
- **Consistency**: Match existing tone and style
- **Accuracy**: Verify all commands and examples work
- **User-Focused**: Write from the user's perspective

### Formatting Standards

#### Headings

Use semantic heading hierarchy:
```markdown
# Page Title (H1) - Only one per page
## Main Section (H2)
### Subsection (H3)
#### Detail (H4)
```

#### Code Blocks

Always specify the language:
```markdown
```bash
sudo systemctl start lyrebird
```
```

Supported languages: `bash`, `python`, `yaml`, `json`, `text`, `diff`

#### Admonitions

Use Material for MkDocs admonitions for important information:

```markdown
!!! note
    Additional helpful information

!!! warning
    Important warnings users should heed

!!! danger
    Critical warnings about destructive operations

!!! tip
    Helpful tips and best practices

!!! example
    Example usage or implementation
```

#### Links

- **Internal Links**: Use relative paths
  ```markdown
  [Installation Guide](../getting-started/installation.md)
  ```

- **External Links**: Use absolute URLs
  ```markdown
  [MediaMTX](https://github.com/bluenviron/mediamtx)
  ```

- **Anchor Links**: Reference specific sections
  ```markdown
  [Prerequisites](#prerequisites)
  ```

#### File Paths and Commands

- File paths: Use inline code with full path
  ```markdown
  Edit `/etc/lyrebird/config.env`
  ```

- Commands: Show full command with prompt
  ```markdown
  ```bash
  sudo systemctl status lyrebird
  ```
  ```

- Environment variables: Use uppercase in code
  ```markdown
  Set `DEVICE_MAIN_SAMPLE_RATE=48000`
  ```

### Content Organization

Each documentation page should follow this structure:

1. **Introduction**: Brief overview of what this page covers
2. **Prerequisites**: What the user needs before proceeding
3. **Main Content**: Step-by-step instructions or reference material
4. **Troubleshooting**: Common issues (if applicable)
5. **Next Steps**: Links to related documentation

### Testing Your Documentation

Before submitting, verify:

- [ ] All commands execute without errors
- [ ] Code examples are syntactically correct
- [ ] File paths reference actual files in LyreBirdAudio
- [ ] Internal links work (mkdocs build --strict passes)
- [ ] External links are not broken (404)
- [ ] Spelling and grammar are correct
- [ ] Page renders correctly in browser

## Project Structure

```
LyreBird-Docs/
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
├── CONTRIBUTING.md           # This file
├── LICENSE                   # Apache 2.0 License
└── README.md                 # Repository README
```

## Documentation Types

### Getting Started

Guides for first-time users:
- Quick Start
- System Requirements
- Installation
- Basic Usage

### User Guide

Operational documentation:
- Configuration
- USB Device Management
- Stream Management
- MediaMTX Integration

### Components

Reference for each LyreBirdAudio component:
- Orchestrator
- Stream Manager
- USB Audio Mapper
- Capability Checker
- Diagnostics Tool
- Version Manager
- MediaMTX Installer

### Advanced

In-depth technical documentation:
- Architecture
- Performance Tuning
- Troubleshooting
- Diagnostics & Monitoring
- Custom Integration

### Reference

Quick reference material:
- Configuration Files
- Environment Variables
- Command Reference
- Exit Codes
- Log Files

### Maintenance

Operational maintenance:
- Version Management
- Backup & Restore
- Uninstallation

## Review Process

### What Reviewers Look For

1. **Accuracy**: Do commands and examples work?
2. **Clarity**: Is the explanation easy to understand?
3. **Completeness**: Is any critical information missing?
4. **Consistency**: Does it match existing documentation style?
5. **Navigation**: Are cross-references appropriate?

### Review Timeline

- **Small changes** (typos, clarifications): 1-2 days
- **Medium changes** (new sections, rewrites): 3-5 days
- **Large changes** (new pages, restructuring): 1-2 weeks

### Handling Feedback

- Reviewers may request changes - this is normal and helpful
- Address feedback by updating your branch and pushing
- Engage in discussion if you disagree or need clarification
- Once approved, maintainers will merge your PR

## Getting Help

### Documentation Questions

- **General Questions**: [GitHub Discussions](https://github.com/tomtom215/LyreBirdAudio/discussions)
- **Bug Reports**: [GitHub Issues](https://github.com/tomtom215/LyreBird-Docs/issues)
- **LyreBirdAudio Project**: [Main Repository](https://github.com/tomtom215/LyreBirdAudio)

### MkDocs and Material Theme

- [MkDocs Documentation](https://www.mkdocs.org/)
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
- [Python-Markdown Extensions](https://python-markdown.github.io/extensions/)

## License

By contributing to LyreBirdAudio documentation, you agree that your contributions will be licensed under the Apache License 2.0.

---

**Thank you for contributing to LyreBirdAudio documentation!**

Your efforts help make this project more accessible and useful for everyone.
