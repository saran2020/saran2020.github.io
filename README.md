# saran2020.github.io

Personal website and technical blog built with [Jekyll](https://jekyllrb.com/) and the [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/) theme.

Visit the live site: [https://saran.sankaran.dev](https://saran.sankaran.dev)

---

## Local Development

### Prerequisites
- **Ruby**: Version `3.3.12` (managed via `mise`, `rbenv`, `asdf`, or system Ruby)
- **Bundler**: Version `2.5+`

### Setup and Running Locally

1. **Install dependencies**:
   ```bash
   bundle install
   ```

2. **Start the development server with live reload**:
   ```bash
   bundle exec jekyll serve --livereload
   ```

3. **Open in browser**:
   Navigate to [http://localhost:4000](http://localhost:4000).

---

## Deployment

The site automatically builds and deploys to **GitHub Pages** on every push to the `main` branch using GitHub Actions (`.github/workflows/deploy.yml`).

Pull requests are automatically validated using CI build verification (`.github/workflows/ci.yml`).

---

## License

Content and articles &copy; Saran Sankaran. Theme code licensed under MIT by Michael Rose.