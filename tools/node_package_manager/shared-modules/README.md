- [Summary](#summary)
- [Vercel build errors with shared modules (even HTTPS)](#vercel-build-errors-with-shared-modules-even-https)
  - [1. **Compile TypeScript Before Publishing**](#1-compile-typescript-before-publishing)
  - [2. **Publish Compiled Code and Type Declarations**](#2-publish-compiled-code-and-type-declarations)
  - [3. **Use the Package in Your Project**](#3-use-the-package-in-your-project)
  - [4. **Alternative: Custom Webpack Configuration (Not Recommended for This Use Case)**](#4-alternative-custom-webpack-configuration-not-recommended-for-this-use-case)
  - [Conclusion](#conclusion)

## Summary

Shared modules work locally nicely, but not when launching to Vercel, therefore a public NPM package is needed.

## Vercel build errors with shared modules (even HTTPS)

The error message you're encountering when deploying to Vercel indicates a problem with handling TypeScript files located in `node_modules`. Specifically, the issue is that Webpack (used by Next.js, which Vercel typically hosts) is not configured to transpile TypeScript files (`*.ts`) from `node_modules`. This can happen if you're importing TypeScript directly from a dependency, which is not standard practice for published npm packages. Typically, npm packages should be pre-compiled to JavaScript and include type declarations (`.d.ts` files) for TypeScript support, rather than containing raw TypeScript files.

Here are steps to resolve this issue and ensure compatibility with Vercel's build system:

### 1. **Compile TypeScript Before Publishing**

The most straightforward solution is to ensure that your package (`@masaok/shared`) is compiled to JavaScript before it's published to npm. This way, consumers of your package won't need to worry about compiling TypeScript from `node_modules`.

- Compile your TypeScript code to JavaScript using the TypeScript compiler (`tsc`).
- Include type declarations by setting `"declaration": true` in your `tsconfig.json`.
- Ensure your `package.json` points to the compiled JavaScript files as the main entry (`"main": "dist/index.js"`) and includes the type declarations (`"types": "dist/index.d.ts"`).

### 2. **Publish Compiled Code and Type Declarations**

When you publish your package, include both the compiled JavaScript code and the type declaration files, but exclude the original TypeScript source files. This is the standard practice for TypeScript libraries on npm.

### 3. **Use the Package in Your Project**

After updating and republishing your package, update the version used in your project. Importing functions from your package will now use the compiled JavaScript, with TypeScript types provided by the accompanying declaration files.

### 4. **Alternative: Custom Webpack Configuration (Not Recommended for This Use Case)**

Although you could theoretically customize your Webpack configuration in Next.js to include a loader for TypeScript files in `node_modules`, this approach is not recommended for solving this particular issue. It introduces unnecessary complexity and can lead to slower build times and other unforeseen issues.

### Conclusion

To ensure smooth deployment on Vercel and compatibility with other environments, it's important to publish your npm packages with JavaScript code and TypeScript declarations, rather than raw TypeScript files. This approach adheres to standard practices for npm packages and avoids the need for consumers of your package to adjust their build configurations to handle TypeScript compilation of `node_modules` dependencies.
