# Personal Website

This is my personal website built with [Hugo](https://gohugo.io/) - a fast and modern static site generator.

## Prerequisites

Before you begin, ensure you have the following installed:
- [Hugo Extended](https://gohugo.io/installation/) (v0.80.0 or newer recommended)
- [Git](https://git-scm.com/downloads)

## Getting Started

### Clone the Repository

```bash
git clone https://github.com/yourusername/your-repo-name.git
cd your-repo-name
```

### Local Development

To start the development server:

```bash
hugo server -D
```

This will start a local development server at http://localhost:1313/ with hot-reload enabled.

- The `-D` flag includes draft content
- Add `-F` flag to include future content
- Add `--navigateToChanged` to navigate to changed content file on live browser reload

### Building the Site

To build the site for production:

```bash
hugo
```

This will generate the static site in the `docs` directory.

## Folder Structure

```
.
├── archetypes/    # Content templates
├── assets/        # Files to be processed by Hugo Pipes
├── config.toml    # Site configuration
├── content/       # Site content
├── data/          # Configuration files
├── docs/          # Generated static site (for GitHub Pages)
├── layouts/       # Template files
├── static/        # Static files
└── themes/        # Site themes
```

## Customization

### Configuration

The main configuration file is `config.toml` (or `config.yaml`/`config.json`). Modify this file to change site title, theme, and other global settings.

### Content Management

Content is stored in the `content/` directory. To create new content:

```bash
# Create a new post
hugo new posts/my-new-post.md

# Create a new page
hugo new pages/about.md
```

## Deployment

### GitHub Pages

This site is configured to be published through GitHub Pages using the `docs` folder. After building your site:

1. Commit and push the changes to your repository
2. In your GitHub repository settings, go to "Pages"
3. Select "main branch" and "/docs" folder as the source
4. Click "Save"

## Troubleshooting

If you encounter any issues:

- Ensure Hugo is properly installed: `hugo version`
- Delete the `resources` directory and restart the server
- Check for errors in the terminal output

## Resources

- [Hugo Documentation](https://gohugo.io/documentation/)
- [Hugo Community Forum](https://discourse.gohugo.io/)
- [Hugo Themes](https://themes.gohugo.io/)