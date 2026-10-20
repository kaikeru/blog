# Blog
The source files used to create the blog @ kaikeru.com.

## Purpose
The blog uses [Hugo](https://gohugo.io) as its base and serves as the the personal website for Kyle Wagner. This is also an exercise in how to use the free tools available to publish a website with minimal or no overhead cost other than time.

## Build Pipeline
Pushes to `master` run `.github/workflows/deploy.yml`, which builds the site with Hugo and publishes it to GitHub Pages. Only `master` deploys; runs from other branches just verify the build. The custom domain is set in the repo's Settings > Pages. To preview locally, install Hugo and run `hugo server -D`.

## GPLv3 License
The source code is licensed under the [GPLv3 license](https://www.gnu.org/licenses/gpl-3.0.en.html).