# garrettnay.com

This is the source code for my website, which is built with [Hugo](http://gohugo.io/). You probably won’t ever need to use this repo in its entirety, but if you want to see an how this particular hugo site is structured, here you go!

[![Build Status](https://travis-ci.org/garrettn/garrettnay.com.svg)](https://travis-ci.org/garrettn/garrettnay.com)
[![devDependencies Status](https://david-dm.org/garrettn/garrettnay.com/dev-status.svg)](https://david-dm.org/garrettn/garrettnay.com?type=dev)
[![forthebadge](http://forthebadge.com/images/badges/uses-html.svg)](http://forthebadge.com)
[![forthebadge](http://forthebadge.com/images/badges/uses-css.svg)](http://forthebadge.com)
[![forthebadge](http://forthebadge.com/images/badges/uses-badges.svg)](http://forthebadge.com)

## Development

### Disclaimer

You can download and use the Hugo binary directly, but in this project I’m using the [hugo-bin](https://github.com/fenneclab/hugo-bin) Node wrapper as a local dependency. Before you start thinking [this is a dumb idea](https://twitter.com/hipsterhacker/status/401817229672468481),
let me explain my reasons:

- I like to keep my dependencies local as much as possible so that it’s easier to get up and running.
- Since I’m using other Node modules to process assets like CSS and (eventually) JS, I'm using npm scripts to build anyway.
- It makes it easier to handle dependencies in a CI environment.

### Requirements

- [Node.js](https://nodejs.org) (I highly recommend a version manager like [n](https://github.com/tj/n) or [nvm](https://github.com/creationix/nvm))
- [Yarn](https://yarnpkg.com/) (recommended) or npm (comes with Node)

If you don’t use Yarn, `yarn` can be replaced with `npm` in the commands below.

### Getting Started

Clone this repository:

```
git clone https://github.com/garrettn/garrettnay.com.git
```

Go into the directory and install dependencies:

```
cd garrettnay.com
yarn install
```

### Running the Development Server

```
yarn start
```

This will kick off two parallel commands (using [npm-run-all](https://www.npmjs.com/package/npm-run-all)): the CSS processing in watch mode with [postcss](https://www.npmjs.com/package/postcss-cli) and the building and serving of the site itself with Hugo. Hugo is completely agnostic to how the static assets are processed, but it will rebuild and reload any time it detects changes in the `static` directory, which is where postcss puts it output files.

The site will now be served from http://localhost:1313.

### Building the Site

```
$ yarn test
```

That one command will take care of processing the CSS as well as building the site itself. I chose to use the `test` command because 1) it’s automatically run on Travis CI without any configuration, and 2) it makes sense as a test because I want to see if the site successfully builds.

The site will be built in the `public/` directory.

## Licenses

The source code of this website is licensed under the [MIT License](LICENSE.txt).

The content of this website is licensed under the [Creative Commons Attribution-ShareAlike 4.0 International License](LICENSE-content.txt).
