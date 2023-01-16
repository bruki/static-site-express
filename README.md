# static-site-express

[![Netlify Status](https://api.netlify.com/api/v1/badges/bb6cf5c7-4ccc-4684-8a82-30e64ac00baa/deploy-status)](https://app.netlify.com/sites/static-site-express/deploys)

static-site-express is a simple Node.js based static-site generator that uses EJS and Markdown. Deploy your static site to Netlify or any platform to your liking. Suited for landing pages, portfolio, blogs, documentation, hobby projects.

## Getting started

### Install static-site-express

1. Click on "Use this template" button to get a exact copy of the repo / site builder that uses Flowbite, an open-sorce UI library of components created with TailwindCSS. Then use the `master` branch, which is the default. Or use the GitHub CLI:

```raw
gh repo create your-username/new-repo  -p SalsaBoy990/static-site-express
```

2. To have a basic e-commerce website Flowbite/TailWind starter incorporating the [Snipcart](https://snipcart.com/) ecommerce platform into static-site-express:

- Checkout branch `snipcart`
- Register at [Snipcart](https://snipcart.com/)
- Copy your Snipcart public test key at `src/layouts/partials/scripts.ejs` to the `publicApiKey` property value:

```html
<div id="snipcart" data-config-modal-style="side" data-api-key="YOUR_PUBLIC_TEST_API_KEY" hidden></div>
```

_Note:_ This api key is public, and can be submitted to version control. There is also a private key, but that should never be committed! You don't even need that for the project.

[Snipcart](https://snipcart.com/) is more than a simple cart: enjoy a full back-office management dashboard to track abandoned carts, sales, orders, customers and more.

- It supports card payments via PayPay, Stripe, and other payment gateways,
- It generates invoices and sends them to customers after purchase,
- etc.

_Note:_ Netlify will build your site from the default branch (usually the `master`) by default.
You can use a different branch other than the default one, but **in that case Netlify CMS will not work properly**. For example, the images uploaded through the CMS will be pushed into the default branch, not the other you set up in Netlify!)

_Test website:_ Use the 'Deploy to Netlify' button at the [project's website](https://static-site-express.netlify.com/) to have a test website.

### Build your site locally

First, create a `.env` file (see `.env.example`), and set the variables.

If you want to use Algolia Search, you need to [register](https://www.algolia.com/doc/guides/getting-started/what-is-algolia/) and generate your API credentials. If you don't want to use Algolia, set `enableSearch` to false in `config/site.config.js`.

Check out all the settings in the `site.config.js`. There are comments there with additional information.

Use npm scripts defined in package.json

Previously, I used [flow](https://flow.org/), a static type checker for JavaScript, to have type checking for the code, but I found it hardly useful.
It caught some wrong argument types, but that is all. Besides, a good IDE can catch those errors without tedious rebuilds all the time changes have been made.
So, building the app code into the `app` folder from `site-generator` folder, where the JS code with flow annotation used to be, is not necessary any more.
The `site-generator` folder was deleted.
Babel and flow packages were removed too.
This script was also deleted:

```raw
bin/generate
```

1. Build site from `./content` into the `./public` folder (in watch mode):

```raw
bin/watch
```

This bash script will call: `npm run watch-chokidar`.
Alternatively, you can also use `npm run watch-nodemon`.

Also call the `bin/css` and `bin/js` watcher scripts to make sure the bundles recreated after file changes.

_TODO:_ The build process is intentionally delayed with setTimeout, to have enough time the css to be compiled after changes (see watch-css script). So it is reacting slower to changes, and the css watch script is also a bit slow.

2. Serve website on `localhost:4000` (or the port you set in .env, default port is 4000):

```raw
bin/serve
```

_TODO:_ the Express dev server crashes rarely - not finding some file generated by the builder. The files and folders in the public folder are deleted and re-copied: for a brief moment it is possible not to have a .html file available to be served by the Express server. However, the site-builder generates everything in a few hundreds of milliseconds (generally less than 300 ms). So this error happens rarely.

Also call the `bin/css` and `bin/js` watcher scripts to make sure the bundles recreated after file changes.

If you don't see your changes:

- After the app code changes you have made, restart `bin/watch`
- Try to restart `bin/css`, and `bin/js`

3. Create the css bundle with PostCSS (in watch mode):

```raw
bin/css
```

4. Create the js bundle with Webpack (in watch mode):

```raw
bin/js
```

Make sure to build the final bundle in production mode for smaller size.

Check out the `bin` folder and the `package.json` file to know more about the available scripts.

### Modify the application code

The JavaScript source is in the `app/` folder. **Generally, you only need to modify the `core/generator.js` and the `core/methods.js` files.**

- `methods.js` contains most of the methods for the generator.
- In `generator.js`, you can modify the pages you want to generate in the switch statements starting from **line 280**. You also need to create a page (`.ejs`) in the `pages/` folder, and a template (in `layouts/`) to be used for that page (or use one of the pre-existing templates like `default.ejs`).
- Post properties can be extended **starting at line 142**, in the `templateConfig` object literal (`generator.js`)

After the changes, restart build/watch scripts. This process in suboptimal, but currently this is the workflow.

### Website content (in the `website/` folder)

- Post data comes from markdown files (in `posts/`) where the front matter block contains the post properties (you can change them, but do not forget to update the `templateConfig` object literal (`generator.js`) as well).
- Pages (in `pages/`) are using templates and partials defined in the `layouts/` folder.
- The `config/site.config.js` file contains some of the global site properties (like site title, author, description, social media links etc.) that are used in the EJS partials. Can also be extended it to your liking.

## Publish Website to Netlify

### Register at Netlify and publish your website

- Register on [Netlify](https://www.netlify.com/), and [see this tutorial video](https://www.netlify.com/docs/continuous-deployment/) if you are unfamiliar with the procedure.

- The `netlify.toml` configuration file contains important properties:

```raw
[build]
  base    = "/"
  publish = "public"
  command = "npm run build"
```

The base path, the build command, and the publish directory. You can keep those settings unchanged.

You can also define here some post-processing actions to be run in post-processing stages as part of Netlify's CI/CD pipeline.

In the optional `_headers` file you can specify the HTTP headers and set Content Security Policy (CSP) rules for the Netlify server. Currently, CSP rules are commented out. You can also specify these in `netlify.toml`.

The `_redirects` file is currently empty. When you have a custom domain, you can make a redirect from _.netlify.com_ to your custom domain.

`robots.txt` default settings:

```raw
# Disallow admin page
User-agent: *
Disallow: /admin/

# Disallow message-sent page
User-agent: *
Disallow: /message-sent/

# Rule 3
User-agent: *
Allow: /
```

For [Google Search Console](https://search.google.com/search-console/about) verification, you should have an HTML file from Google included in the root of your Netlify publish folder (in our case, `public`). The build script copies this file from `./content` to `./public`.

Add the name of the filename in the `filesToCopy` array at line 100 in `./app/core/generator.js` and restart watch script!

Netlify builds your website with its buildbot. It starts a Docker container running the [Netlify build image](https://hub.docker.com/r/netlify/build/#!)


#### For folks unfamiliar with Docker

TL;DR: Netlify install a lot of packages (copies files over) to be able to run your favorite tool to build your static website. And this is done in a Docker container. Read the overview section of Docker docs: https://docs.docker.com/get-started/

A Docker container is basically a writable OverlayFS (FS = filesystem) layer created on top of the numerous read-only OverlayFS layers of the Docker image (files copied on top of each other: each layer is represents a command in the Dockerfile). Which is destroyed after the build has been completed (the data can be made permanent using volumes which are kept).

The images are based on base images (the FROM statement at the first line of a Dockerfile) that are special distributions that "think they are operating systems", but are more lightweight that a complete OS.

[Alpine Linux](https://hub.docker.com/_/alpine/) is the most lightweight of them (around 5MB). Interesting to note, that [images can built from scratch as well](https://codeburst.io/docker-from-scratch-2a84552470c8) (scratch is a reserved image that is empty, and thus does nothing!). The base images are built this way ("FROM scratch").

Docker is using the kernel and obviously the resources of the host (which are shared), and are meant for process isolation only. Containers are more lightweight, don't have the overheads Virtual Machines do. [More about this topic](https://www.simplilearn.com/tutorials/docker-tutorial/docker-vs-virtual-machine).

VMs are used for full isolation including resources (for example, to subdivide the server resources for shared hosting: each hosting having a computing power of X CPUs of X type, have X GB of memory and X GB storage space), and have a separate (full) OS installed along with the host OS, so they do not share the kernel.

If you use Windows, you need to install Windows Subsystem for Windows 2 (WSL2) to have a (not fully featured) distro based on Linux kernel installed, that will run as a regular application. Although, there are container base images available for Windows as well. So, Docker can even use the Windows kernel now (for specific images).

Lots of images are pre-built for us (like the `netlify/build` image) and stored in the Docker (not DockerHub) registry (DockerHub is just a user interface). You don't need to build them from Dockerfile, you just download them from the registry.

If you know the Docker basics, you can understand some things about Netlify as well.
Check these shell scripts out:

When the Docker fires up, this script runs:
https://github.com/netlify/build-image/blob/focal/run-build.sh

This is the Dockerfile from which the Netlify image is built (currently based on `ubuntu:20.04`, older Ubuntu base images - like 16.04 - are deprecated now):
https://github.com/netlify/build-image/blob/focal/Dockerfile


### Netlify Forms

Netlify automatically discovers the contact form via custom netlify attributes added to the form. A bot field is present in the form to protect against spam bots. Netlify has first-class spam filter.

[Netlify Forms Docs](https://docs.netlify.com/forms/setup/)

### Netlify CMS

Optional: set `display_url` to your custom domain in `content/admin/config.yml`

[Netlify CMS Docs](https://github.com/netlify/netlify-cms)

### Algolia Search

These are the key parts in the code for Algolia:

```JavaScript
const algoliasearch = require("algoliasearch");
const client = algoliasearch(process.env.ALGOLIA_APP_ID, process.env.ALGOLIA_ADMIN_KEY);
const index = client.initIndex(process.env.ALGOLIA_INDEX);
```

Here, I use the AlgoliaSearch client library to send request to update and/or create records for the posts:

```JavaScript
index.partialUpdateObjects(searchIndexData, {
  createIfNotExists: true,
});
```

This is currently the structure of the search index (as a default example):

```JavaScript
searchIndexData.push({
  /**
   * The object's unique identifier
   */
  objectID: postData.attributes.date,

  /**
   * The URL where the Algolia Crawler found the record
   */
  url: canonicalUrl,

  /**
   * The lang of the page
   * - html[attr=lang]
   */
  lang: config.site.lang,

  /**
   * The title of the page
   * - og:title
   * - head > title
   */
  title: postData.attributes.title,

  /**
   * The description of the page
   * - meta[name=description]
   * - meta[property="og:description"]
   */
  description: postData.attributes.excerpt,

  /**
   * The image of the page
   * - meta[property="og:image"]
   */
  image: config.site.seoUrl + "/assets/images/uploads/" + postData.attributes.coverImage,

  /**
   * The authors of the page
   * - `author` field of JSON-LD Article object: https://schema.org/Article
   * - meta[property="article:author"]
   */
  authors: [config.site.author],

  /**
   * The publish date of the page
   * - `datePublished` field of JSON-LD Article object: https://schema.org/Article
   * - meta[property="article:published_time"]
   */
  datePublished: postData.attributes.date,

  /**
   * The category of the page
   * - meta[property="article:section"
   * - meta[property="product:category"]
   */
  category: postData.attributes.topic || "",

  /**
   * The content of your page
   */
  content: postContents,
});
```

_Note:_ Currently the `objectID` is the post publish date (like "2022-08-17"). Maybe better to change it to the whole slug to be completely unique. You can't have two posts at the same day now.

### Internationalisation (i18n)

Provided by **i18next** package. See in `assets/js/main.js`. Remove this feature if you don't need it. Some texts (like the ones coming from config) cannot be made translatable. Probably not a good solution. The code is left there mainly for reference.

The translations come from `content/lang/translations.json`.

[i18next Docs](https://www.i18next.com/)

### Open Hours library for displaying opening hours.

Credits: © Michael Lee.
[GitHub](https://github.com/michaellee/open-hours)
[His website/blog](https://michaelsoolee.com/)

When I started my journey as web developer, I started using [Jekyll](http://jekyllrb.com/) for my simple websites after reading some articles from Michael Lee about it. He has a great starter for Jekyll, the [Jekyll ⍺](https://github.com/michaellee/jekyll-alpha).

The data comes from `content/data/opening-hours.yml`. It can be edited from Netlify CMS as well.

## Changelog

### Release 2.1.1 (30 December 2022)

The project is now in mature state, there will be no more refactoring, only bugfixes and occasional improvements of features. No breaking changes.
I don't want to be part of the "rewrite culture". Also, not a fan of npm any more. I use as small amount of packages as possible.
Having 1000s of interdependent packages (with all these regular rewrites, and security issues as well) is a dependency hell.


**Delete:**

- Flow static type checker and Babel removed
- `site-generator` folder removed, `bin/generate`, and npm scripts related are deleted as well.

**Update:**

- The source code now is in the `app` folder that you should edit. No need to rebuild all the time with Babel.
- Update Readme


### Release 2.1.0 (16 August 2022)

**New feature:**

- Add Algolia search support for blogposts
- Add: Opening hours table (a small package)
- Add: i18n support (only an example)

**Fix:**

- CSP rules with whitelist for Algolia, Netlify CMS
- fix: Try to create a workaround for "Gotrue-js: failed getting jwt access token error
- seo fixes in head

**Security:**

- fix: Disallow using strings with DOM XSS injection sink functions (CSP)

**Update:**

- Post content

### Release 2.0.0 (14 August 2022)

This intended to be the last major version release.

**New:**

- Refactor folder structure **(breaking change)**.
- Asset folder refactoring **(breaking change)**.
- Add config files for Tailwind, PostCSS, and Webpack.
- Webpack will be used to bundle JS, and PostCSS to bundle CSS.
- Integrate TailwindCSS, use the Flowbite UI component library with flowbite/typography (CSS+JS).
- Add `.env.example`, remove `.env`, change .env vars.
- Add post-processing, http header settings to `netlify.toml` configuration.
- Add Content Security Policy in `netlify.toml` with whitelisting Netlify-specific domains.
- Add bash scripts with meaningful names to type less to call npm scripts.
- Add new favicon and app logos.

**Update:**

- Update package.json version number.
- Update existing, remove not needed, add new npm packages, supported Node version is: `v16.14.0`.
- Update paths in scripts **(breaking change)**.
- Update comments in the code.
- Set node version for Netlify to `v16.14.0`.
- Update layouts, pages and posts (to use TailWindCSS).
- Update Netlify CMS settings.
- Update Netlify CMS needs to have the website source files in `content/` folder.
- Update README.md.
- Update .gitignore.

**Delete:**

- Remove Docker configuration: using the site-builder in Docker would only add unnecessary complexity for zero gain.
- Remove GA scripts.
- remove Heroku Procfile.

**New/Update/Delete:**

- Define new, rename existing, remove not needed npm scripts.


### Release 1.0.2 (28 April 2021)

- Update README.md.
- Update package version number.
- Set node version for Netlify to avoid package compatibility errors.
- Update Netlify CMS settings, change media folder, input types.
- Netlify CMS needs to have the website source files in src/ folder.


### Release 1.0.1 (27 April 2021)

**Incorrect configuration in docker-compose.yml:**

- fix: "generator" and "devserver" services share volume data. "devserver" is dependent on the "generator" service.


### Release 1.0.0 (25 April 2021) ! breaking change from previous versions !

- version re-started with _1.0.0_ (from _4.1.0_, it was a complete mess before).
- Update npm dependencies to the newest versions.
- Build script partial code refactoring, code styling.

**Correct EJS syntax error after EJS version update:**

- fix: From now on, EJS include directives should be in this format:

`<%- include ('partial/element-name') %>`

This is a **breaking change**, you should update your `partials/templates`!

**Update build and watch scripts (using chokidar):**

- update: build script content moved into a module (`generator.js`) to be used in a build and the chokidar-based watch scripts.

In 2019, chokidar was not watching file changes properly, thus the npm script was named "watch-exp". The default watch script is using nodemon.

**Add flow types support and re-structure folders:**

- new: add Flow, a static type checker for JavaScript.
- update: site generator source moved to `src/`, Babel will transpile the source into the `lib/` folder where originally the source were.
- update: website source is moved to `website/` folder, necessary code changes are applied.

**Refactor site generator, code improvements, config changes:**

- update: `package.json` add dotenv package, update npm scripts.
- update/add: refactor static site generator scripts, changes in methods, add types to code with flow, update/add comments to every method.
- add: lang and month names options to site.config.js.

**Dockerize project:**

- new: read variables from `.env` file.
- new: add Dockerfile, docker-compose file, `.dockerignore`.

## Useful resources

- [Netlify Docs](https://docs.netlify.com/)
- [Netlify CMS docs](https://www.netlifycms.org/docs/intro/)
- [How the Netlify buildbot builds websites](https://www.netlify.com/blog/2016/10/18/how-our-build-bots-build-sites/)
- [Netlify Drop - formarly BitBalloon](https://www.netlify.com/blog/2018/08/14/announcing-netlify-drop-the-simplicity-of-bitballoon-with-the-added-power-of-netlify/)
- [Complete Intro to Netlify in 3.5 hours](https://www.netlify.com/blog/2019/10/07/complete-intro-to-netlify-in-3.5-hours/)
- [TailwindCSS Docs](https://tailwindcss.com/docs/installation)
- [TailwindCSS Setup](https://flaviocopes.com/tailwind-setup/)
- [Webpack tutorial](https://www.sitepoint.com/bundle-static-site-webpack/)
- [Terser Webpack plugin](https://webpack.js.org/plugins/terser-webpack-plugin/)
- [Flowbite UI library based on TailwindCSS](https://flowbite.com/docs/getting-started/introduction/)

## Known issues

### 1. Chokidar crashes on Ubuntu 20.04 LTS (12 August 2022)

Bug: Problem with an existing folder.

The build script should always delete the folders inside the public folder.
However, the assets folder is sometimes not deleted, so an exception occurs:
`[Error: EEXIST: file already exists, mkdir './public/assets']`


### 2. Nodemon was not working properly on Ubuntu (2019)

- `nodemon` not trigger re-build on Linux on file changes (this behavior was experienced on Ubuntu 18.04 LTS Bionic Beaver)
- On Ubuntu, you can run `npm run watch-exp` command which uses the [chokidar](https://github.com/paulmillr/chokidar) package.
- Update: Chokidar is working properly on Ubuntu 20.04 (28 April 2021)


If you have a problem or a question about static-site-express, [open an issue here](https://github.com/SalsaBoy990/static-site-express/issues).

## Credits

The idea of using a Node.js static site generator came from this good article by Douglas Matoso (not accessible any more): [Build a static site generator in 40 lines with Node.js](https://medium.com/douglas-matoso-english/build-static-site-generator-nodejs-8969ebe34b22).

This package uses some modified code parts from [**doug2k1/nanogen**](https://github.com/doug2k1/nanogen) (mainly from the `legacy` branch and some ideas from the `master` branch, MIT © Douglas Matoso 2018).

## Licence

MIT licence - Copyright © 2022 András Gulácsi.
