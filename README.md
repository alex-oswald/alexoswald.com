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
bundle exec jekyll build
```

## Development

In VSCode run the `Development (livereload/drafts)` task.

```
bundle exec jekyll serve
```

Open a browser and navigate to `localhost:4000`




<script src="https://giscus.app/client.js"
        data-repo="alex-oswald/blog"
        data-repo-id="R_kgDOGSe--w"
        data-category="Announcements"
        data-category-id="DIC_kwDOGSe--84CaHp6"
        data-mapping="pathname"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="preferred_color_scheme"
        data-lang="en"
        crossorigin="anonymous"
        async>
</script>