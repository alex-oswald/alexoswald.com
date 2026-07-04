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

## Deployments

The site is hosted on a single [Azure Static Web App](https://learn.microsoft.com/en-us/azure/static-web-apps/) (`salmon-ground`) and deployed by [`.github/workflows/azure-static-web-apps-salmon-ground-05ba16610.yml`](.github/workflows/azure-static-web-apps-salmon-ground-05ba16610.yml):

- **Open or update a pull request against `main`** → deploys to a single, shared preview environment named `dev` (stable URL), posted as a comment on the PR. Every PR reuses this one environment so we stay within the Free plan's staging-environment limit.
- **Merge to `main`** → deploys to the production site.
- **Close the pull request** → the close job runs, but the shared `dev` preview is intentionally persistent and reused by the next PR.
