# My Engineering Blog

## Stack

| Layer | Tech |
|-|-
| Static site generator | [Hugo](https://gohugo.io/) |
| Theme | [Hextra](https://github.com/imfing/hextra) |
| Comments | [Giscus](https://giscus.app/) |
| Hosting provider | [Azure Static Web Apps](https://learn.microsoft.com/en-us/azure/static-web-apps/) |

## Building

> Requires **Hugo Extended** (the Hextra theme uses SCSS) and **Go** (the theme is
> pulled in as a Hugo Module). The pinned versions live in `.devcontainer` and the
> GitHub Actions workflows. Opening the repo in the dev container gives you both.

### Production

In VSCode run the `Production Build` task.

```
hugo
```

## Development

In VSCode run the `Development (livereload/drafts)` task.

```
hugo server -D
```
