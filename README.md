# Blog

[![Deploy Hugo site to Pages](https://github.com/soupglasses/blog/actions/workflows/pages.yaml/badge.svg)](https://github.com/soupglasses/blog/actions/workflows/pages.yaml)

## Local Development

Pre-requisites: [Hugo](https://gohugo.io/getting-started/installing/), [Go](https://golang.org/doc/install) and [Git](https://git-scm.com)

```shell
# Clone the repo
git clone https://github.com/soupglasses/blog.git

# Change directory
cd blog

# Start the server
hugo mod tidy
hugo server --logLevel debug --disableFastRender -p 1313
```

### Update theme

Blog is built with the [Hextra](https://github.com/imfing/hextra) theme.

```shell
hugo mod get -u
hugo mod tidy
```

See [Update modules](https://gohugo.io/hugo-modules/use-modules/#update-modules) for more details.
