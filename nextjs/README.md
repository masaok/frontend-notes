- [Why NextJS uses SWC](#why-nextjs-uses-swc)
- [What is the equivalent of tsc for testing typescript validity?](#what-is-the-equivalent-of-tsc-for-testing-typescript-validity)

### Why NextJS uses SWC

Next.js uses SWC, a Rust-based platform for the Web, as a key part of its build toolchain for several compelling reasons:

1. **Performance**: SWC is designed to be extremely fast. It leverages the Rust programming language's performance advantages to compile JavaScript, TypeScript, and JSX code much faster than traditional JavaScript-based tools like Babel. This speed boost translates to quicker build and development times, significantly improving the developer experience, especially on large projects.

2. **TypeScript Support**: SWC includes built-in support for TypeScript, allowing Next.js projects to compile TypeScript without relying on additional tools like the TypeScript compiler (tsc). This integration further speeds up the development process by simplifying the toolchain and reducing compilation times.

3. **Customizability and Future-proofing**: SWC is highly customizable, supporting a wide range of JavaScript features and experimental proposals. This flexibility ensures that Next.js developers can use the latest JavaScript features without waiting for the tooling to catch up. Additionally, SWC's architecture is designed to be easily extendable, making it more adaptable to future web standards and technologies.

4. **Ecosystem Compatibility**: While SWC is a relatively new entrant in the JavaScript tooling landscape, it has been designed to be compatible with existing ecosystems and tools. This means that Next.js projects can easily integrate with the broader JavaScript ecosystem, leveraging a vast array of libraries and tools without significant compatibility issues.

5. **Reduced Configuration Overhead**: By using SWC, Next.js aims to reduce the complexity and overhead associated with setting up and maintaining the build toolchain. SWC provides sensible defaults and a simplified configuration process, making it easier for developers to get started and focus on building their applications rather than tinkering with build tools.

In summary, the adoption of SWC by Next.js is driven by the need for a fast, efficient, and flexible build process that enhances the development experience, supports modern web development practices, and ensures compatibility with the evolving JavaScript ecosystem.

### What is the equivalent of tsc for testing typescript validity?

When using Next.js with SWC, the equivalent functionality for testing TypeScript validity, akin to running `tsc` in projects that use the TypeScript compiler directly, is not provided directly by SWC. This is because SWC, while it does transpile TypeScript to JavaScript for performance gains, does not perform type checking. It focuses on compilation speed and leaves type checking to other tools.

For projects that need to enforce type safety and perform type checking, the recommended approach is to still use the TypeScript compiler (`tsc`) in parallel with your build process. Here’s how you can integrate type checking into your Next.js project using SWC for compilation:

1. **Install TypeScript**: Ensure TypeScript is installed in your project if it's not already. You can do this by running `npm install typescript --save-dev` or `yarn add typescript --dev`.

2. **TypeScript Configuration**: Ensure you have a `tsconfig.json` file in your project. This file configures how TypeScript performs type checking. Next.js will automatically generate a basic `tsconfig.json` for you if you start a project with TypeScript files but don't already have this file.

3. **Running Type Checks**: You can manually check types in your project by running `tsc --noEmit`. This command will perform type checking across your project without emitting compiled JavaScript files. You might want to add this as a script in your `package.json` for ease of use, like so:

   ```json
   "scripts": {
     "type-check": "tsc --noEmit"
   }
   ```

   Then, you can run `npm run type-check` or `yarn type-check` to perform type checks.

4. **Integration with Development Workflow**: For a more integrated experience, you might want to set up pre-commit hooks using tools like Husky to run type checks before allowing commits, or integrate type checking into your CI/CD pipeline to ensure type safety is enforced before deployment.

This approach allows you to leverage SWC for its fast compilation times while still maintaining the type safety guarantees provided by TypeScript. It's a bit of a workaround, as it involves running an additional step in your development process, but it effectively combines the strengths of both tools.
