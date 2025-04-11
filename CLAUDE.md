# Jekyll GitHub Pages Site Guidelines

## Build Commands
- Local server: `bundle exec jekyll serve`
- Build site: `bundle exec jekyll build`
- Update dependencies: `bundle update`
- Install dependencies: `bundle install`

## Project Structure
- `_posts/`: Blog posts in Markdown format (`YYYY-MM-DD-title.md`)
- `_site/`: Generated site (don't edit directly)
- Content pages: `index.md`, `about.md`, etc.

## Content Guidelines
- Use Markdown for content
- Post frontmatter needs: `layout: post`, `title:`, `date:`, optional `categories:`
- Image path: `/assets/images/`
- Internal links: `[text]({% raw %}{% link pagename.md %}{% endraw %})`
- Code blocks: Use triple backticks with language specified

## Jekyll Configuration
- Theme: leap-day
- Markdown engine: kramdown
- Permalink structure: `/blog/:year/:month/:day/:title`