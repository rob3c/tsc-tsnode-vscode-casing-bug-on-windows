BUG REPRO HERE: https://github.com/rob3c/tsc-tsnode-vscode-casing-bug-on-windows

## Overview

Starting with the 2016-07-14 release of Visual Studio Code Insiders edition (and now released in regular VS Code 1.3.1), the behavior of the _Open New Command Prompt_ command (`ctrl+shift+C`) changed on Windows. Now, it lower-cases the drive letter in the new console's prompt, and that seems to affect the ability of at least `ts-node` (and possibly other tools) to build typescript code when the `forceConsistentCasingInFileNames` compiler option is set in _tsconfig.json_.

## Step 1 - Open two command prompts for comparison

- Within VS Code, invoke the `Open New Command Prompt` command via `ctrl+shift+C` (or find it using _View | Command Palette..._ in the menu). Notice that the drive letter in the prompt is lower case now!

- Open a standard Windows Command Prompt outside of VS Code in the normal way, and navigate to this project's root folder. Notice that the standard console has a captial drive letter.

## Step 2 - create some source files

```
// tsconfig.json
{
    "compilerOptions": {
        "forceConsistentCasingInFileNames": true,
        "module": "commonjs",
        "moduleResolution": "node",
        "outDir": "bin",
        "target": "es5"
    },
    "exclude": [
        "bin",
        "node_modules"
    ]
}
```
```
// foo.ts
export class Foo {
    constructor(public bar: string) {}
}
```
```
// index.ts
import { Foo } from './foo';
let foo = new Foo('bar');
console.log(JSON.stringify(foo));
```

## Step 3 - Build successfully using typescript directly

```
> tsc
```
Or, run the `build` script in _package.json_ to use the `tsc` in node_modules without requiring a global install):
```
> npm run build
```
This builds successfully in both command prompt windows.

## Step 4 - Break stuff!

```
> ts-node index.ts
```
Or, run the `break-stuff` script in _package.json_ to use the local `ts-node` without requiring a global install):
```
> npm run break-stuff
```
This builds and runs successfully in the standard Windows command prompt that has an upper-case drive letter, but it breaks in the console window that VS Code opens with this error:
```
> ts-node index.ts


C:\dev\bug-repros\tsc-tsnode-vscode-casing-bug-on-windows\node_modules\ts-node\src\index.ts:280
        throw new TSError(diagnosticList)
              ^
TSError: ? Unable to compile TypeScript
index.ts (1,21): File name 'C:/dev/bug-repros/tsc-tsnode-vscode-casing-bug-on-windows/foo.ts' differs from already included file name 'c:/dev/bug-repros/tsc-tsnode-vscode-casing-bug-on-windows/foo.ts' only in casing (1149)
    at getOutput (C:\dev\bug-repros\tsc-tsnode-vscode-casing-bug-on-windows\node_modules\ts-node\src\index.ts:280:15)
    at compile (C:\dev\bug-repros\tsc-tsnode-vscode-casing-bug-on-windows\node_modules\ts-node\src\index.ts:289:14)
    at loader (C:\dev\bug-repros\tsc-tsnode-vscode-casing-bug-on-windows\node_modules\ts-node\src\index.ts:304:23)
    at Object.require.extensions.(anonymous function) [as .ts] (C:\dev\bug-repros\tsc-tsnode-vscode-casing-bug-on-windows\node_modules\ts-node\src\index.ts:321:14)
    at Module.load (module.js:458:32)
    at tryModuleLoad (module.js:417:12)
    at Function.Module._load (module.js:409:3)
    at Function.Module.runMain (module.js:575:10)
    at Object.<anonymous> (C:\dev\bug-repros\tsc-tsnode-vscode-casing-bug-on-windows\node_modules\ts-node\src\_bin.ts:172:12)
    at Module._compile (module.js:541:32)
```

