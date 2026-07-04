# My Engineering Blog

## Stack

| Layer | Tech |
|-|-
| Static site generator | [Hugo](https://jekyllrb.com/) |
| Theme | [Hextra](https://github.com/imfing/hextra) |
| Comments | [Giscus](https://giscus.app/) |
| Hosting provider | [Azure Static Web Apps](https://learn.microsoft.com/en-us/azure/static-web-apps/) |

## Building

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

- **Open or update a pull request against `main`** → deploys to an automatic preview ("dev") environment with its own URL, posted as a comment on the PR.
- **Merge to `main`** → deploys to the production site.
- **Close the pull request** → the preview environment is torn down.
