- [How to analyze bundle sizes without ejection](#how-to-analyze-bundle-sizes-without-ejection)
  - [1. **Using `source-map-explorer`**](#1-using-source-map-explorer)
- [How to analyze package sizes](#how-to-analyze-package-sizes)
  - [1. Use Webpack Bundle Analyzer](#1-use-webpack-bundle-analyzer)
  - [2. Source Map Explorer](#2-source-map-explorer)
  - [3. Lighthouse](#3-lighthouse)
  - [4. Custom Webpack Configuration](#4-custom-webpack-configuration)
  - [5. Analyze Dependencies](#5-analyze-dependencies)
  - [Best Practices](#best-practices)
- [Where is the create-react-app webpack-bundle-analyzer](#where-is-the-create-react-app-webpack-bundle-analyzer)
  - [1. Ejecting](#1-ejecting)
  - [2. Using `react-app-rewired`](#2-using-react-app-rewired)
  - [3. Using `craco` (Create React App Configuration Override)](#3-using-craco-create-react-app-configuration-override)
  - [Conclusion](#conclusion)

## How to analyze bundle sizes without ejection

Analyzing bundle sizes in a Create React App (CRA) project without ejecting can be accomplished using tools and techniques that don't require modifying the internal webpack configuration directly.

Here are several methods to do so:

### 1. **Using `source-map-explorer`**

`source-map-explorer` analyzes JavaScript bundles using the source maps. This allows you to see where code bloat is coming from.

- **Installation**: Install it by running:

  ```bash
  npm install --save-dev source-map-explorer
  ```

- **Usage**: After building your app with `npm run build`, you can analyze the bundle by running:

  ```bash
  npx source-map-explorer 'build/static/js/*.js'
  ```

- **Or in terminal**: List bundles with sizes in descending size order:

  ```bash
  npx source-map-explorer build/static/js/*.js --tsv | less
  ```

- **Even better in HTML**: List bundles with sizes in descending size order:
  ```bash
  npx source-map-explorer build/static/js/*.js --html > build/bundle.html
  open build/bundle.html
  ```

This method doesn't require any modifications to your CRA setup.

## How to analyze package sizes

Analyzing package sizes in a React application is crucial for optimizing its performance, particularly for user load times.

Here's how you can analyze and manage your package sizes:

### 1. Use Webpack Bundle Analyzer

**Note: this solution requires ejection.**

Webpack Bundle Analyzer is a powerful tool that provides a visual way of understanding the output of your webpack bundle. It shows which components make up the bundle and their sizes, helping identify which packages are bloating your application.

- **Installation**: Run `npm install --save-dev webpack-bundle-analyzer` in your project directory.
- **Usage**: After installing, modify your Webpack configuration to include the plugin. Here’s a basic setup:

```js
const BundleAnalyzerPlugin =
  require("webpack-bundle-analyzer").BundleAnalyzerPlugin;

module.exports = {
  plugins: [new BundleAnalyzerPlugin()],
};
```

When you run your build script, the plugin will open up a browser window showing a treemap visualization of your bundle.

### 2. Source Map Explorer

Source Map Explorer analyzes JavaScript bundles using the source maps. This makes it easy to see what code each bundle includes and understand where the code bloat is coming from.

- **Installation**: Run `npm install --save-dev source-map-explorer`.
- **Usage**: Ensure your build generates source maps. Then, analyze a specific bundle by running `source-map-explorer bundle.js`.

### 3. Lighthouse

Lighthouse, by Google, is an automated tool for improving the quality of web pages. It has audits for performance, accessibility, progressive web apps, and more. It can be run in Chrome DevTools, from the command line, or as a Node module. Lighthouse's performance audit includes a section on JavaScript and CSS size.

- **Usage**: In Chrome, go to the Lighthouse tab in DevTools, and run a performance audit on your site. Look for the "JavaScript and CSS Size" audits for insights.

### 4. Custom Webpack Configuration

By customizing your Webpack configuration, you can add optimizations like tree shaking, code splitting, and minification. These techniques help reduce bundle size by eliminating unused code, splitting your code into smaller chunks, and compressing your JavaScript files.

### 5. Analyze Dependencies

Before adding a new package to your project, evaluate its size and impact on your bundle. Tools like BundlePhobia (visit `https://bundlephobia.com/`) let you understand the cost of adding a npm package to your project.

### Best Practices

- **Regularly Audit**: Make bundle size analysis a regular part of your development workflow.
- **Lazy Load**: Use dynamic `import()` syntax for lazy loading components that aren't immediately necessary.
- **Prune Unused Dependencies**: Regularly review and remove unused dependencies from your project.

By incorporating these tools and practices into your development process, you can significantly improve your React application's load time and overall performance.

## Where is the create-react-app webpack-bundle-analyzer

`create-react-app` (CRA) abstracts away its Webpack config, making it initially inaccessible for direct modifications like adding plugins. To use `webpack-bundle-analyzer` or modify the Webpack config in a project bootstrapped with CRA, you have a few options:

### 1. Ejecting

Ejecting your CRA application will expose all the configuration files (including Webpack configs), allowing you to directly add and configure `webpack-bundle-analyzer`.

- **Ejecting**: Run `npm run eject`. Be cautious, as this action is irreversible. After ejecting, you can modify the Webpack config directly to include `webpack-bundle-analyzer`.

### 2. Using `react-app-rewired`

`react-app-rewired` is a popular alternative that allows you to override the CRA's Webpack config without ejecting.

- **Installation**: Install `react-app-rewired` and `webpack-bundle-analyzer`.

  ```
  npm install react-app-rewired webpack-bundle-analyzer --save-dev
  ```

- **Configuration**: Create a `config-overrides.js` file in your project root:

  ```js
  const { override, addWebpackPlugin } = require("react-app-rewired");
  const BundleAnalyzerPlugin =
    require("webpack-bundle-analyzer").BundleAnalyzerPlugin;

  module.exports = override(addWebpackPlugin(new BundleAnalyzerPlugin()));
  ```

- **Usage**: Replace `react-scripts` in your `package.json` scripts with `react-app-rewired`:

  ```json
  "scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build",
    "test": "react-app-rewired test",
    "eject": "react-scripts eject"
  }
  ```

Running `npm run build` now will include the bundle analysis.

### 3. Using `craco` (Create React App Configuration Override)

`craco` is another tool to customize CRA configurations without ejecting, similar to `react-app-rewired`.

- **Installation**: Install `@craco/craco` and `webpack-bundle-analyzer`.

  ```
  npm install @craco/craco webpack-bundle-analyzer --save-dev
  ```

- **Configuration**: Create a `craco.config.js` in your project root:

  ```js
  const { BundleAnalyzerPlugin } = require("webpack-bundle-analyzer");

  module.exports = {
    webpack: {
      plugins: {
        add: [new BundleAnalyzerPlugin()],
      },
    },
  };
  ```

- **Modify Package.json**: Update the scripts to use `craco` instead of `react-scripts`.

  ```json
  "scripts": {
    "start": "craco start",
    "build": "craco build",
    "test": "craco test"
  }
  ```

### Conclusion

While ejecting gives you full control over the configuration, it also means taking over the responsibility for maintaining this configuration. Tools like `react-app-rewired` and `craco` offer a more maintainable way to customize the Webpack configuration of your CRA project, including adding `webpack-bundle-analyzer` for bundle analysis.

https://chat.openai.com/share/73967d63-5e4a-4169-8d86-13d123a288ff
