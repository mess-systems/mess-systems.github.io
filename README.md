# mess.systems website

Source of [mess.systems](https://mess.systems/): an open organization used as a worked example of how work, companies, people and systems change in the move from the pre-AI era to the AI era. The name comes from Russell Ackoff's idea of a *mess*, a system of interdependent, changing problems.

The site is built with [Hugo](https://gohugo.io/) and a small custom layout. It loads no external fonts, scripts, trackers or analytics.

## Run locally

Install [Hugo extended](https://gohugo.io/installation/) 0.167.0 or later, then:

```sh
hugo server            # http://localhost:1313, published content only
hugo server -D         # include drafts
hugo --minify          # production build into public/
```

## How content is organized

| Path | What it holds |
|---|---|
| `content/en/` | English, the canonical text, served at `/` |
| `content/ru/` | Russian translation, served at `/ru/` |
| `content/zh/` | Simplified Chinese translation, served at `/zh/` |
| `content/<lang>/blog/` | Blog posts |
| `data/projects.toml` | Project list with names and descriptions in all three languages |
| `i18n/` | Interface strings (navigation, footer) per language |
| `layouts/` | Templates: `baseof.html`, `home.html`, `page.html`, `section.html`, partials and shortcodes |
| `assets/css/main.css` | The only stylesheet; light and dark themes follow `prefers-color-scheme` |
| `static/CNAME` | Custom domain for GitHub Pages |

## Translations

English is canonical. If a translation and the English text disagree, the English text applies.

A translation is a file with the same path in another language folder, for example `content/en/about.md` and `content/ru/about.md`. Hugo links them automatically, and the language switcher in the header goes to the same page in the other language.

When you change an English page, update the Russian and Chinese pages in the same pull request or open an issue saying they are behind. Names, identifiers and code are never translated, only the explanations around them.

## Drafts and publishing

New posts start with `draft: true` in the front matter. Drafts are not built in production. Remove the flag once the post is approved.

Every push to `main` builds the site and deploys it to GitHub Pages through `.github/workflows/pages.yml`. The Hugo version is pinned there in `HUGO_VERSION`.

## License

- Content (everything under `content/` and `data/`) is licensed under [CC BY 4.0](LICENSE).
- Code, layouts and styles are licensed under the [MIT License](LICENSE-CODE).
