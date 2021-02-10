---
layout: post
title: "Building A Static Site Generator With Deno"
description: "Using TypeScript with Deno and Markdown to generate a static site."
date: 2021-02-10
tags: [deno,markdown,static-site-generator,typescript,javascript]
comments: true
share: true
---
![](https://res.cloudinary.com/dvivnklwq/image/upload/v1612798286/liqswmkgiqg77h7pdb6m.png)

Building static web content is an absolute necessity. If you've been active in web development any time in the last 10 years, you've more than likely come across static-site generators like [Jekyll](https://jekyllrb.com/) or [Gatsby](https://www.gatsbyjs.com/). Even CMS systems like [Wordpress](https://wordpress.com/) perform quite a bit of static content generation.

Thanks to modern JavaScript, you don't have to be a programming wizard to build fancy static-site generators. In fact, you can accomplish quite a lot with _very little code_. That's what we are going to do in this blog post, and we are going to use an amazing new piece of technology called **Deno**.

Also, we will be using TypeScript, **BUT WAIT!** no prior TypeScript skills are required for this tutorial. If you're comfortable with JavaScript you'll be absolutely fine. In fact, you can use this to dip your feet in some TypeScript if you haven't already.

## What is Deno?

I won't go into too much detail about **[Deno](https://deno.land/)** here, but you can think of it as the spiritual successor to `Node.js` that is also built by node's original creator.

It's a secure runtime for JavaScript with support for TypeScript, code formatting, testing, top-level async and many more features right out-of-the-box.

One huge difference between `deno` and `node` is that there is no package manager. Rather than specifying packages or third party dependencies in a separate file, deno packages are specified **in-code** with url-based import statements like so:

```javascript
import { parse } from 'https://deno.land/std@0.85.0/datetime/mod.ts'
```

When running your program, these dependencies are checked, installed and cached.

For security, you use flags to determine the level of file permissions and network access available to scripts when run with deno like so:

```bash
deno run --allow-read mod.ts
```

### Installation

_Note: You can skip this step if you already have deno installed._

To install deno for the first time, use any of the following options:

##### MacOS or Linux (Shell)

```bash
curl -fsSL https://deno.land/x/install/install.sh | sh
```

##### MacOS (brew)

```bash
brew install deno
```

##### Windows

```
iwr https://deno.land/x/install/install.ps1 -useb | iex
```

_Note: The remainder of the tutorial will use shell commands meant for MacOS/Linux, however all of the coding instructions will be the same for Windows._

## What We Will Be Generating

![](https://res.cloudinary.com/dvivnklwq/image/upload/v1612567343/ze3ayxhxu9bt5rd4h406.gif)

Our goal will be to write a deno script in TypeScript that will take a single `.md` file as input and generate the following:

- A website layout with navigation bar, content area and a footer
- Folders and `index.html` files that adhere to the website's structure (1 layer of depth)
- Valid `href` values in all navigation links
- A generated stylesheet that is linked in each page
- An SVG emoji favicon that is linked in each page

### Thinking About Websites Programmatically

If you are still working on your web fundamentals, a static-site generator is an amazing project to help strengthen those skills. Once you finish this, you'll see that the sky is the limit. Building your own functioning CMS, auto-generated documentation, or automating the maintenance of a marketing site won't be too far off.

## Let's Build Our Deno Markdown Site Generator

_Note: All source code for this blog post is available at [`github.com/nafeu/deno-md-site`](https://github.com/nafeu/deno-md-site)_

Start off by creating a new project folder and `main.ts` file:

```bash
mkdir deno-md-site
cd deno-md-site
touch main.ts
```

Open `main.ts` in your editor of choice and for now, add the following comments:

```typescript
/* Section: Dependencies */

/* Section: Constants */

/* Section: Interfaces and Globals */

/* Step 0: Grab CLI arguments */

/* Step 1: Parse metadata and components from markdown file */

/* Step 2: Construct page data from components */

/* Step 3: Generate templates for html content */

/* Step 4: Build pages into .html files with appropriate paths */

/* Step 5: Build additional asset files */
```


This is how the script will be structured so it is easy to follow along. We will jump between sections, update constants and add dependencies when they are needed.

### Step 0. Grab CLI arguments

When we run our script, we will provide the path to an `.md` file (filename) and a build path for where our website will be built, to access these command line arguments, we can use `Deno.args` like so:

```typescript
/* Step 0: Grab CLI arguments */
const filename = Deno.args[FIRST_ITEM_INDEX];
const buildPath = Deno.args[SECOND_ITEM_INDEX] || './build';
```

You'll notice we used two constants here, `FIRST_ITEM_INDEX` and `SECOND_ITEM_INDEX`, we haven't declared these yet so lets ago ahead and do that in our constants section like so:

```
/* Section: Constants */
const FIRST_ITEM_INDEX = 0;
const SECOND_ITEM_INDEX = 1;
```

If an argument isn't provided, we want to log a message informing the user and then exit the script, otherwise continue. We can do that by adding the following:

```typescript
/* Step 0: Grab CLI arguments */
const filename = Deno.args[FIRST_ITEM_INDEX];
const buildPath = Deno.args[SECOND_ITEM_INDEX] || './build';

if (!filename) {
  console.log('Please specify .md file');
  Deno.exit(1);
} else {
  console.log(`Building site with '${filename}' into '${buildPath}'`);
}
```


Now if we test our script we should see the following:

```text
$ deno run main.ts
Please specify .md file
```

And with two args we get:

```text
$ deno run main.ts testFile testDir
Building site with 'testFile' into 'testDir'
```

### Defining Our Solution

Before we continue, there are a couple of things we want to think through. What are some default values we should consider? Are we missing some dependencies? How can we use `types` to clearly define the problem we are going to solve?

Let's add the following into our **Interfaces and Globals** section:

```typescript
/* Section: Interfaces and Globals */
interface Page {
  path: string,
  name: string,
  html: string
}

interface Layout {
  [key: string]: any
}

let pages: Array<Page> = [];
let layout: Layout = {};
```


Here we are saying that we want our script to focus on an object type **`Page`** which will have:
- a string `path` to a page (think of a link to the page within the website, not a path in our filesystem)
- a string `name` which would be the title of our page
- a string `html` which will include all of the HTML for that page

We want to consider an additional object **`Layout`** which may store information on something like a reuseable footer. Lastly, we define an array `pages` which stores **`Page`** objects as well as instantiating our **Layout** object.

Now that we have these set up, we will import a couple of extra dependencies. We will import `Marked` from [https://deno.land/x/markdown](https://deno.land/x/markdown) and `ensureFileSync` from [https://deno.land/std](https://deno.land/std). These will help us parse markdown as well as create/persist file and directory paths respectively.

We can import them like so:

```typescript
/* Section: Dependencies */
import { Marked } from 'https://deno.land/x/markdown@v2.0.0/mod.ts';
import { ensureFileSync } from 'https://deno.land/std@0.84.0/fs/mod.ts';
```


### Step 1. Parse metadata and components from markdown file

Create a new `TextDecoder` with a `utf-8` encoding option and read the contents of the file we specified when running our script:

```typescript
/* Step 1: Parse metadata and components from markdown file */
const decoder = new TextDecoder("utf-8");
const fileContent = decoder.decode(Deno.readFileSync(filename));
```


Now we have access to our file content, but wait... what goes inside this file anyways? Let's take a step back and make some decisions on how we want to template using markdown.

### Deciding On Template Mechanics

Let's keep our rules simple:

- use [YAML Front Matter](https://jekyllrb.com/docs/front-matter/) to declare a website title, optional css styles and an optional emoji favicon
- use triple plus signs (`+++`) to separate pages and layout components
- use the format `/[PAGE_PATH]:[PAGE_TITLE]` underneath the triple plus signs to denote a page's path and title respectively
- use regular markdown for page content in the area below page path/title declarations
- a mandatory home page under the format `/home:Home`

We can create an example `.md` file with `touch example.md` and fill it with:

~~~text
---
title: Deno Markdown Site
styles: >
  body { color: #22a6b3; }
favicon: 🦕
---
/home:Home

# Home

Hello world!

+++
/about:About

# About

Built for learning.

+++
layout:footer

deno-md-site

~~~

Now in our `main.ts` file, let's split the file content on `+++`, create a constant called `COMPONENT_DELIMITER`:

```typescript
/* Section: Constants */
const FIRST_ITEM_INDEX = 0;
const SECOND_ITEM_INDEX = 1;
const COMPONENT_DELIMITER = '+++';
```

We will use this constant with `fileContent.split(...)` in `Step 1`:

```typescript
/* Step 1: Parse metadata and components from markdown file */
const decoder = new TextDecoder("utf-8");
const fileContent = decoder.decode(Deno.readFileSync(filename));

const components = fileContent.split(COMPONENT_DELIMITER);
```


If you temporarily add `console.log(components)` and run the script using `deno run --allow-read main.ts example.md` you should see the following:

```json
[
  "---\ntitle: Deno Markdown Site\nstyles: >\n  body { color: #22a6b3; }\nfavicon: 🦕\n---\n/home:Home\n\n# Hom...",
  "\n/about:About\n\n# About\n\nBuilt for learning.\n\n",
  "\nlayout:footer\n\ndeno-md-site\n"
]
```


We can see that there are `3` items in this array, the **home page** with front matter (title, styles and favicon), the **about page**, and the **footer** layout component. Each of these items are valid **markdown**, this is very important for us.

Let's extract the front matter from the first item using the `Marked` library:

```typescript
/* Step 1: Parse metadata and components from markdown file */
const decoder = new TextDecoder("utf-8");
const fileContent = decoder.decode(Deno.readFileSync(filename));

const components = fileContent.split(COMPONENT_DELIMITER);
const { meta: frontMatter } = Marked.parse(components[FIRST_ITEM_INDEX]);
const { title, styles, favicon } = frontMatter;
```


In this instance, `Marked.parse(...)` takes valid markdown, does some processing and then returns some values. From these values we choose to only deconstruct a field called `meta`, we then rename that field to `frontMatter` for better reading.

Let's add `console.log({ title, styles, favicon })` and then run our script using:

```bash
deno run --allow-read --unstable main.ts example.md
```

_Note: We are using the flags `--allow-read` to allow read operations in our file system and `--unstable` because some of Deno's standard library is not yet 100% stable._

We should see:

```javascript
{ title: "Deno Markdown Site", styles: "body { color: #22a6b3; }\n", favicon: "🦕" }
```


Great! Now we know how to split up our templated markdown file and extract useful information. We can move on to constructing the rest of the pages.

### Step 2. Construct Page Data From Components

Let's use `Marked` to generate HTML markup from our markdown, add the following into `Step 2`:

```typescript
/* Step 2: Construct page data from components */
for (const component of components) {
  const { content } = Marked.parse(component);

  console.log(content);
}
```


`Marked.parse(...)` returns a `content` field that contains HTML which we are destructuring here. We can use a temporary `console.log(...)` statement to see it's results. Run the script again and we should have:

```html
<p>/home:Home</p>
<h1 id="home">Home</h1>
<p>Hello world!</p>

<p>/about:About</p>
<h1 id="about">About</h1>
<p>Built for learning.</p>

<p>layout:footer</p>
<p>deno-md-site</p>
```


Awesome! We've finally got some generated markup to work with. For each component, you can see that the first tag is a paragraph tag with the path and title of the page. Let's use regular expressions to pull these out.

### Using Simple Regular Expressions To Extract Markup

_Note: I'm no RegExp expert and I usually use [`regex101.com`](https://regex101.com/) to help me construct patterns._

Given the sample text

```html
<p>/home:Home</p>
<h1 id="home">Home</h1>
<p>Hello world!</p>
```

We want to pull out `/home` and `Home` from the first tag. Similarly, we want to pull out `layout` and `footer` from

```html
<p>layout:footer</p>
<p>deno-md-site</p>
```

We can think of these values as our **Component Type Values**, and to extract them we can use the following pattern:

```typescript
const COMPONENT_TYPE_PATTERN = /<\S>(.*?)\:(.*?)<\/\S>/g;
```


This can be broken down as follows:
- match the first instance of `<tag>[GROUP_1]\:[GROUP_2]</tag>`
- `<\S>` and `<\/\S>` will match the opening and enclosing tag
- `(.*?)\:(.*?)` will match ANYTHING between `<tag>` to `:` and then `:` to `</tag>` respectively into _capturing groups_
- the first `(.*?)` capturing group gives a **component type** which can be
  - a **page path** starting with a `/`
  - the word **layout** which means this is a layout component
- the second `(.*?)` capturing group gives a **component value** which can be
  - the **page title** for a **page path**
  - a specific layout component (ie. **footer**)
- `\S` matches any non-whitespace character
- using the `g` global pattern flag means that we will match the entire body of text given

Let's add this pattern to our constants

```typescript
/* Section: Constants */
const FIRST_ITEM_INDEX = 0;
const SECOND_ITEM_INDEX = 1;
const COMPONENT_DELIMITER = '+++';
const COMPONENT_TYPE_PATTERN = /<\S>(.*?)\:(.*?)<\/\S>/g;
```

And then let's update `Step 2` as follows:

```typescript
/* Step 2: Construct page data from components */
for (const component of components) {
  const { content } = Marked.parse(component);

  const [matchedComponentType] = content.matchAll(COMPONENT_TYPE_PATTERN);
  const [, path, name] = matchedComponentType;

  console.log({ path, name });
}
```


Here we are taking that html `content` and using `content.matchAll(COMPONENT_TYPE_PATTERN)` to get all of our `matchedComponentType` information. We are then using array destructuring with `const [, path, name]` to ignore the first item (which would be our full match) and then pulling `path` and `name` out of the matched groups.

Since we are logging the `path` and `name`, running the script again should give us:

```javascript
{ path: "/home", name: "Home" }
{ path: "/about", name: "About" }
{ path: "layout", name: "footer" }
```


Now we also want the actual HTML content for each of these components, to start, let's declare another pattern like so:

```typescript
const HTML_CONTENT_PATTERN = /\n(.*)/gs;
```

This is a much simpler pattern and can be broken down as follows:
- `\n(.*)` matches everything that comes after the first newline (`\n`) character
- `(...)` creates a capturing group
- `.*` means it will match ANY sequence of characters
- using the `s` global pattern flag means that a newline character is also matched within the scope of `.`
- using the `g` global pattern flag means that we will match the entire body of text given

So given the text

```html
<p>/home:Home</p>
<h1 id="home">Home</h1>
<p>Hello world!</p>
```

We want to extract our HTML page content as

```html
<h1 id="home">Home</h1>
<p>Hello world!</p>
```

Let's update our constants with our new pattern

```typescript
/* Section: Constants */
const FIRST_ITEM_INDEX = 0;
const SECOND_ITEM_INDEX = 1;
const COMPONENT_DELIMITER = '+++';
const COMPONENT_TYPE_PATTERN = /<\S>(.*?)\:(.*?)<\/\S>/g;
const HTML_CONTENT_PATTERN = /\n(.*)/gs;
```

And update `Step 2` to also use the new pattern to extract HTML

```typescript
/* Step 2: Construct page data from components */
for (const component of components) {
  const { content } = Marked.parse(component);

  const [matchedComponentType] = content.matchAll(COMPONENT_TYPE_PATTERN);
  const [, path, name] = matchedComponentType;

  const [matchedHtml] = content.matchAll(HTML_CONTENT_PATTERN);
  const [, html] = matchedHtml;

  console.log({ path, name, html });
}
```


We've also added `html` into the log statement, running the script should give us:

```javascript
{
  path: "/home",
  name: "Home",
  html: '<h1 id="home">Home</h1>\n<p>Hello world!</p>\n'
}
{
  path: "/about",
  name: "About",
  html: '<h1 id="about">About</h1>\n<p>Built for learning.</p>\n'
}
{
  path: "layout",
  name: "footer",
  html: "<p>deno-md-site</p>\n"
}
```


Wicked! Now that we have our components, we can store them into the appropriate variables `layout` and `pages` we declared earlier in our `Interfaces and Globals` section. Add our final update to `Step 2`:

Declare another constant `LAYOUT_PREFIX` and add it like so:

```typescript
/* Section: Constants */
const FIRST_ITEM_INDEX = 0;
const SECOND_ITEM_INDEX = 1;
const COMPONENT_DELIMITER = '+++';
const COMPONENT_TYPE_PATTERN = /<\S>(.*?)\:(.*?)<\/\S>/g;
const HTML_CONTENT_PATTERN = /\n(.*)/gs;
const LAYOUT_PREFIX = 'layout';
```

We will use `LAYOUT_PREFIX` to help us decide whether the `path` that was returned is a layout component or not, then we will either map the component values into the `layout` object or push the entire component into the `pages` array.

```typescript
/* Step 2: Construct page data from components */
for (const component of components) {
  const { content } = Marked.parse(component);

  const [matchedComponentType] = content.matchAll(COMPONENT_TYPE_PATTERN);
  const [, path, name] = matchedComponentType;

  const [matchedHtml] = content.matchAll(HTML_CONTENT_PATTERN);
  const [, html] = matchedHtml;

  const isLayoutComponent = path === LAYOUT_PREFIX;

  if (isLayoutComponent) {
    layout[name] = html;
  } else {
    pages.push({ path, name, html });
  }
}
```


If we log `layout` and `pages` at this point and run the script we should see:

```javascript
{
  layout: { footer: "<p>deno-md-site</p>\n" },
  pages: [
    {
      path: "/home",
      name: "Home",
      html: '<h1 id="home">Home</h1>\n<p>Hello world!</p>\n'
    },
    {
      path: "/about",
      name: "About",
      html: '<h1 id="about">About</h1>\n<p>Built for learning.</p>\n'
    }
  ]
}
```


Pretty sweet huh?

### 3. Generate Templates For HTML Content

Now that we have all of our content, we need to actually build all the other HTML necessary for our site. We will create a bunch of template helper functions:

Before we start, let's add two more constants `HOME_PATH` and `STYLESHEET_PATH`:

```typescript
/* Section: Constants */
const FIRST_ITEM_INDEX = 0;
const SECOND_ITEM_INDEX = 1;
const COMPONENT_DELIMITER = '+++';
const COMPONENT_TYPE_PATTERN = /<\S>(.*?)\:(.*?)<\/\S>/g;
const HTML_CONTENT_PATTERN = /\n(.*)/gs;
const LAYOUT_PREFIX = 'layout';
const HOME_PATH = '/home';
const STYLESHEET_PATH = 'styles.css';
```


These will help us going forward, let's add a helper method `isHomePath` to help us decide if the component we are processing is the home page:

```typescript
/* Step 3: Generate templates for html content */
const isHomePath = (path: string) => path === HOME_PATH;
```


Then let's add a helper that takes a string `path`, decides if we are on the home page and then returns the final path to a stylesheet (we will generate soon):

```typescript
/* Step 3: Generate templates for html content */
const isHomePath = (path: string) => path === HOME_PATH;

const getStylesheetHref = (path: string) => {
  return isHomePath(path) ? STYLESHEET_PATH : `../${STYLESHEET_PATH}`;
}
```


Now for some templating, let's add a template helper to build a favicon as an svg:

```typescript
/* Step 3: Generate templates for html content */
const isHomePath = (path: string) => path === HOME_PATH;

const getStylesheetHref = (path: string) => {
  return isHomePath(path) ? STYLESHEET_PATH : `../${STYLESHEET_PATH}`;
}

const getFaviconSvg = (favicon: string) => `
  <svg xmlns="http://www.w3.org/2000/svg">
    <text y="32" font-size="32">${favicon ? favicon : '🦕'}</text>
  </svg>
`;
```


Since the emoji favicon is optional, if we get one, we render it, if not, we default to the 🦕. Now let's create a `<div>` for navigation as follows:

```typescript
const getNavigation = (currentPath: string) => `
  <div id="nav">
    ${pages.map(({ path, name }) => {
      const href = path === HOME_PATH ? '/' : path;
      const isSelectedPage = path === currentPath;
      const classes = `nav-item ${isSelectedPage ? 'selected': ''}`;
      return `<a class="${classes}" href=${href}>${name}</a>`;
    }).join('\n')}
  </div>
`;
```


This takes a string `currentPath`, loops through each `page` we have in our `pages` variable and maps a set of `<a>` tags where ONE will have an additional class `selected` if the `currentPath` and page `path` match. The `href` is also determined by whether or not the page is the home page. This method constructs something like:

```html
<div id="nav">
  <a class="nav-item selected" href="/">Home</a>
  <a class="nav-item" href="/about">About</a>
</div>
```


We can also quickly declare a footer template like so:

```
const footer = layout.footer ? `<div id="footer">${layout.footer}</div>` : '';
```


This just checks our previously declared `layout` mapping to see if a footer exists, and if it does, pulls it's HTML into a div. In our scenario it would give us:

```
<div id="footer">
  <p>deno-md-site</p>
</div>
```

Let's add both of these helpers to `Step 3` so we have:

```typescript
/* Step 3: Generate templates for html content */
const isHomePath = (path: string) => path === HOME_PATH;

const getStylesheetHref = (path: string) => {
  return isHomePath(path) ? STYLESHEET_PATH : `../${STYLESHEET_PATH}`;
}

const getFaviconSvg = (favicon: string) => `
  <svg xmlns="http://www.w3.org/2000/svg">
    <text y="32" font-size="32">${favicon ? favicon : '🦕'}</text>
  </svg>
`

const getNavigation = (currentPath: string) => `
  <div id="nav">
    ${pages.map(({ path, name }) => {
      const href = path === HOME_PATH ? '/' : path;
      const isSelectedPage = path === currentPath;
      const classes = `nav-item ${isSelectedPage ? 'selected': ''}`;
      return `<a class="${classes}" href=${href}>${name}</a>`;
    }).join('\n')}
  </div>
`;

const footer = layout.footer ? `<div id="footer">${layout.footer}</div>` : '';
```


Now for our MAIN html content that is the essential structure of the `index.html` files themselves. Let's create another template helper called `getHtmlByPage` which takes a type `Page` as input:

```typescript
const getHtmlByPage = ({ path, name, html }: Page) => `
  <!DOCTYPE html>
  <html>
  <head>
    <title>${name} | ${title}</title>
    <link rel="stylesheet" href="${getStylesheetHref(path)}">
    <link rel="icon" href="/favicon.svg">
  </head>
    <body>
      <div id="title">
        ${title}
      </div>
      ${getNavigation(path)}
      <div id="main">
        ${html}
      </div>
      ${footer}
    </body>
  </html>
`;
```


Here you can see we are using all of the previously declared helpers and variables that are relevant to our cause.
- `${name}` is the name of the current **Page**
- `${title}` is the title of our whole website (ie. `Deno Markdown Site`)
- `${getStylesheetHref(path)}` links us to a stylesheet (which again, we will construct later on)
- `${getNavigation(path)}` generates the navigation div
- `${html}` is the HTML of the current page

Our final `Step 3` section should look like

```typescript
/* Step 3: Generate templates for html content */
const isHomePath = (path: string) => path === HOME_PATH;

const getStylesheetHref = (path: string) => {
  return isHomePath(path) ? STYLESHEET_PATH : `../${STYLESHEET_PATH}`;
}

const getFaviconSvg = (favicon: string) => `
  <svg xmlns="http://www.w3.org/2000/svg">
    <text y="32" font-size="32">${favicon ? favicon : '🦕'}</text>
  </svg>
`;

const getNavigation = (currentPath: string) => `
  <div id="nav">
    ${pages.map(({ path, name }) => {
      const href = path === '/home' ? '/' : path;
      const isSelectedPage = path === currentPath;
      const classes = `nav-item ${isSelectedPage ? 'selected': ''}`;
      return `<a class="${classes}" href=${href}>${name}</a>`;
    }).join('\n')}
  </div>
`;

const footer = layout.footer ? `<div id="footer">${layout.footer}</div>` : '';

const getHtmlByPage = ({ path, name, html }: Page) => `
  <!DOCTYPE html>
  <html>
  <head>
    <title>${name} | ${title}</title>
    <link rel="stylesheet" href="${getStylesheetHref(path)}">
    <link rel="icon" href="/favicon.svg">
  </head>
    <body>
      <div id="title">
        ${title}
      </div>
      ${getNavigation(path)}
      <div id="main">
        ${html}
      </div>
      ${footer}
    </body>
  </html>
`;
```


Everything is slowly coming together. The last thing we have to do is to generate our `index.html` files and their associated folders.

### Step 4. Build Pages Into .html Files With Appropriate Paths

This step is relatively simple, we know that each `page` in our `pages` variable has all of our HTML content as well as a `path`. For example, the path can look like `/` or `/about`, etc. Based on these values, let's define **output paths** which will be the actual file system path for where our index.html files get written:

```typescript
/* Step 4: Build pages into .html files with appropriate paths */
for (const page of pages) {
  const { path } = page;

  let outputPath: string;

  if (path === HOME_PATH) {
    outputPath = `${buildPath}/index.html`;
  } else {
    outputPath = `${buildPath}${path}/index.html`;
  }

  console.log({ outputPath });
}
```


Remember when we grabbed our `buildPath` from the CLI arguments earlier in the script? Here we can stitch it together with the page path to give us our **output path**. Using the temporary `console.log(...)`, we can run the script and we get:

```javascript
{ outputPath: "./build/index.html" }
{ outputPath: "./build/about/index.html" }
```


Let's update `Step 4` to now persist these files into our file system, to make sure we include the correct content inside the files, we use our `getHtmlByPage` template helper:

```typescript
/* Step 4: Build pages into .html files with appropriate paths */
for (const page of pages) {
  const { path } = page;

  let outputPath: string;

  if (path === HOME_PATH) {
    outputPath = `${buildPath}/index.html`;
  } else {
    outputPath = `${buildPath}${path}/index.html`;
  }

  ensureFileSync(outputPath);
  Deno.writeTextFileSync(outputPath, getHtmlByPage(page));
}
```


Here, [`ensureFileSync`](https://deno.land/std@0.74.0/fs#ensurefile) ensures that the file exists. If the specified path contains directories that do not exists, these directories are created. After we guarauntee that file path exists, `Deno.writeFileSync` uses `getHtmlByPage` with our selected **Page** object and generates all the necessary HTML content for it (as per our templates).

Also, since we are **writing** to our file system now, we have to use the `--allow-write` flag, test the script out like so:

```
deno run --allow-read --allow-write --unstable main.ts example.md
```


This should generate a `build` directory with the following structure:

```text
build
├── about
│   └── index.html
└── index.html
```

### Step 5. Build Additional Asset Files

Our last step is to generate additional asset files and make sure they are also persisted in the appropriate place for our build. We only have two additional assets in this example, a stylesheet (`styles.css`) and a favicon (`favicon.svg`). We can create them like so:

```typescript
/* Step 5: Build additional asset files */
Deno.writeTextFileSync(`${buildPath}/styles.css`, styles ? styles : '');
Deno.writeTextFileSync(`${buildPath}/favicon.svg`, getFaviconSvg(favicon));
```


Now when we run our script our directory structure is

```text
build
├── about
│   └── index.html
├── favicon.svg
├── index.html
└── styles.css
```

### Previewing Our Site

First let's `cd build` to go inside our build folder. There are many options for us to run a local web server, pick whichever of the following is easiest for you:

- **Python 2**: `python -m SimpleHTTPServer 8001`
- **Python 3**: `python3 -m http.server 8001`
- **PHP**: `php -S localhost:8001`
- **Browsersync** (node package): `npm install -g browser-sync; browser-sync --port 8001`

Then if we open `localhost:8001` in our local browser we should have:

![](https://res.cloudinary.com/dvivnklwq/image/upload/v1612797137/vh5hbxez28gttxpu526s.gif)

That's it, if you want to try a more advanced example with styles, check out this [`example-site.md`](https://raw.githubusercontent.com/nafeu/deno-md-site/main/example-site.md) available at [`github.com/nafeu/deno-md-site`](https://github.com/nafeu/deno-md-site).

Thanks for getting to the end of this tutorial, I hope it was helpful and you have opened your imagination to more clever static site generation. Happy coding!