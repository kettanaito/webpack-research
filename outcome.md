# Outcome
* Setup
  * [Repository integration](#repository-integration)
  * [Configuration composition](#configuration-composition)
  * [Vendor bundle](#vendor-bundle)
* Development
  * Hot Module Replacement
* Production
  * Minification and tree shaking
  
## Foreword
For the sake of keeping this document clean, chosen solutions will be marked with :white_check_mark: icon.

## Results
This is the collection of personal findnigs which were acquired during the research:

---

### Repository integration
There are multiple ways to integrate webpack into your project's repository. Let's go through each of them, compare their benefits and drawbacks, and see which one is the most suitable for our projects.

#### A. Using webpack via CLI
**Example** (package.json):
```
{
 "scripts": {
  "init": "NODE_ENV=development webpack --watch"
  "build": "NODE_ENV=production webpack"
 }
}
```
> **Note:** This is an example code. It is recommended to use [webpack-dev-server](https://github.com/webpack/webpack-dev-server) for a real-world usage. This will allow hot module replacement and much more.
* **Pros:**
  * Easy to setup, minimum pipeline changes
  * Ideal for small projects (i.e. libraries, plugins)
* **Cons:**
  * Setup is not extendable
  * Not suitable for large-scale projects
  
#### B. Using webpack via NodeJS API :white_check_mark:
By that, it means using `webpack(webpackConfig)` as a function. Read more about this in [webpack NodeJS API](https://webpack.github.io/docs/node.js-api.html).
* **Pros:**
  * Easy to adapt to any build pipeline
  * Extendable, since used directly in the code (i.e. may be configured depending on the environment)
  * Direct access to the generated `stats.json`, since it is availabile as one of the callback arguments
* **Cons:**
  * Requires to setup stdout manually (see the repository for an example), otherwise webpack output will not be seen in the terminal

> **Note:** You **should not** run `webpack()` in a `watch` mode. Use [webpack-dev-server](https://github.com/webpack/webpack-dev-server) or [webpack-dev-middleware](https://github.com/webpack/webpack-dev-middleware) instead.

---

### Configuration composition
One of the first aspects to consider was a proper approach toward configuration composition. Webpack configurations may easilty get quite big, making them harder to maintain and debug.
Therefore, the following approaches were tested:

#### A. Keeping isolated configurations for different bundles (client, vendor, etc.)
* **Pros:**
  * Less boilerplate, everything needed for the configuration is inside the configuration
  * Easier to debug
* **Cons:**
  * Large configuration files
  * Harder to read
  * Repetitive code
  * Impossible to achieve good maintainability for same loaders
 
#### B. One default configuration (`default.babel.js`) extended by each target configuration (`client.babel.js`)
* **Pros:**
  * Commonly used parts (loaders, plugins, etc.) are at one place, making them easier to maintain
  * Great for ESLint's `import-webpack-resolver`, where you can reference `default.babel.js` with the included resolvers
* **Cons:**
  * Becomes harder to maintain once introducing webpack configurations for different matters (client, server, api).
  Configurations may share certain parts between each other, while having completely unique parts. In some cases, inheriting certain loaders/plugins by extension
  leads to unnecessary processing, making the builds slower and bundles larger
  
#### C. Reusable *parts*, configurations composed from the parts (like LEGO) :white_check_mark:
* **Pros:**
  * Ability to have independant, reusable and maintainble parts
  * Each part may have relatively complex internal logic without making the whole setup more complex
  * Each part may have its own API, preventing from the duplication of code
  * Parts are interchangable, can be easilty replaced or omitted (better debugging)
  * Parts serve as a single source of truth (better maintainence and debugging)
* **Cons:**
  * More boilerplate
  * Parts should have reasonable internal logic, otherwise they would grow to complex, decreasing their readability

### Vendor bundle
Vendor bundle is a dedicated JS file containing all used dependencies of another bundle. In our case, `vendor.js` contains the modules (third-party libraries) required to run `client.js` (application bundle). There are a few ways to approach vendor modules management.

#### A. Specify vendor modules manually
```js
// webpack/presets/vendor.js
{
 entry: ['react', 'react-dom', 'react-redux', ...]
}
```
* **Pros:**
  * More granular control over what gets included into the vendor bundle
* **Cons:**
  * Forces to modify module's array each time a dependency is installed or removed (bad maintainance)

#### B. Use `package.dependencies` dynamically :white_check_mark:
Since our projects are not libraries, and are not meant to be installed differently rather than `npm install`, the separation into `devDependencies` and `dependencies` is quite relative. This allows us to control what goes into the vendor bundle by simply installing the module using a `-S` flag (which says that the module is a save dependency for the project).
* **Pros:**
  * Dynamic control over what gets included into the vendor bundle
  * No need to maintain vendor configuration at all; maintainance is done by installing/removing the modules using a proper flag
  * Feels intuitive, since vital (save) dependencies *should* be present in order for the app to run
* **Cons:**
  * Requires developers' acknowledgement, so to prevent unnecessary things being bundled into vendor
