## Overview

The 2016-07-14 release of Visual Studio Code Insiders edition changed the behavior of their _Open New Command Prompt_ command (`ctrl+shift+C`) on Windows. Now, it lower-cases the drive letter in the new console's prompt, and that seems to affect the ability of at least `ts-node` (and possibly other tools) to build typescript code when the `forceConsistentCasingInFileNames` compiler option is set in _tsconfig.json_.

## Step 1 - Open the command prompts for comparison

- Using vscode insiders build from 2016-07-14, invoke the `Open New Command Prompt` command via `ctrl+shift+C` (or find it using _View | Command Palette..._ in the menu). Notice that the drive letter in the prompt is lower case now!

- Open a standard Windows Command Prompt outside of vscode in the normal way, and navigate to this project's root folder. Notice that the standard console has a captial drive letter.
3

## Step 2 - Build successfully using typescript directly

Run the build script from _package.json_:
```
> npm run build
```
This builds successfully in both command prompt windows.

## Step 3 - Break stuff!

Run the break-stuff script from _package.json_:
```
> npm run break-stuff
```
This builds and runs successfully in the standard Windows command prompt that has an upper-case drive letter, but it breaks in the console window that vscode opens!
