# Steven Lin's Blog

Source for [stevenlin.cc/blog](https://stevenlin.cc/blog/), built with Hugo and the [Stack theme](https://github.com/CaiJimmy/hugo-theme-stack).

The site supports English (`en`, default) and Traditional Chinese (`zh-tw`).

## Requirements

- Hugo Extended 0.164.0
- Go 1.24 or newer

## Local preview

```bash
hugo server -D
```

Open <http://localhost:1313/blog/>. Hugo watches the source files and refreshes the site after changes.

## Writing

Create an English article:

```bash
hugo new content post/my-article/index.md
```

For a translated page bundle, keep the resources together and add the language suffix to the content file:

```text
content/post/my-article/
├── index.md          # English (default)
├── index.zh-tw.md    # Traditional Chinese
└── cover.jpg      # Shared resource
```

See [WRITING.md](WRITING.md) for the front matter and publishing conventions.

## Production build

```bash
hugo --minify --gc
```

Pull requests run the same build check with Hugo Extended 0.164.0. Pushing to `master` deploys the generated site to the `gh-pages` branch through GitHub Actions.
