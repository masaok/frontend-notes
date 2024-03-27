- [Cannot use import statement outside a module](#cannot-use-import-statement-outside-a-module)
  - [Solution](#solution)

### Cannot use import statement outside a module

```bash
    /path/to/jest.setup.js:1
    ({"Object.<anonymous>":function(module,exports,require,__dirname,__filename,jest){import '@testing-library/jest-dom'
                                                                                      ^^^^^^

    SyntaxError: Cannot use import statement outside a module
```

#### Solution

```
bun add -d @swc/jest
```

`jest.config.js`:

```
  // Must use @swc/jest to transform TypeScript files in NextJS, which uses SWC to compile TypeScript
  // https://github.com/swc-project/jest/issues/102#issue-1253068032
  transform: {
    '.*\\.(j|t)sx?$': ['@swc/jest'],
  },
```
