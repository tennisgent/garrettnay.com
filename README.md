# garrettnay.com

This is the source code for my website, which is built with [Hugo](http://gohugo.io/). You probably won’t ever need to use this repo in its entirety, but if you want to see an how this particular hugo site is structured, here you go!

[![Build Status](https://travis-ci.org/garrettn/garrettnay.com.svg)](https://travis-ci.org/garrettn/garrettnay.com)
[![devDependencies Status](https://david-dm.org/garrettn/garrettnay.com/dev-status.svg)](https://david-dm.org/garrettn/garrettnay.com?type=dev)
[![forthebadge](http://forthebadge.com/images/badges/uses-html.svg)](http://forthebadge.com)

## Development

### Disclaimer

You can download and use the Hugo binary directly, but in this project I’m using the [hugo-bin](https://github.com/fenneclab/hugo-bin) Node wrapper as a local dependency. Before you start thinking [this is a dumb idea](https://twitter.com/hipsterhacker/status/401817229672468481),
let me explain my reasons:

- I like to keep my dependencies local as much as possible so that it’s easier to get up and running.
- I’m eventually going to use other Node modules to process assets like JS and CSS, I will be using npm scripts to build anyway.
- It makes it easier to handle dependencies in a CI environment.

### Requirements

- [Node.js](https://nodejs.org) (I highly recommend a version manager like [n](https://github.com/tj/n) or [nvm](https://github.com/creationix/nvm))
- [Yarn](https://yarnpkg.com/) (recommended) or npm (comes with Node)
- (optional) [Hugo](http://gohugo.io/)

### Getting Started

Clone this repository:

```
$ git clone https://github.com/garrettn/garrettnay.com.git
```

Go into the directory and install dependencies:

```
$ cd garrettnay.com

$ yarn install

# OR
$ npm install
```

### Running the Development Server

```
$ yarn start

# OR
$ npm start

# OR, if you have Hugo in PATH
$ hugo serve
```

The site will now be served from http://localhost:1313.

### Building the Site

```
$ yarn test

# OR
$ npm test

# OR, if you have Hugo in PATH
$ hugo
```

The site will be built in the `public/` directory.

## Licenses

The source code of this website is licensed under the [MIT License](LICENSE.txt).

The content of this website is licensed under the [Creative Commons Attribution-ShareAlike 4.0 International License](LICENSE-content.txt).
