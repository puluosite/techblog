## SSH setup
1. Install VSCode on your client machine
2. Install Remote Development extension pack from Microsoft (extension id: ms-vscode-remote.vscode-remote-extensionpack). You only really care about "Remote - SSH" extension.
3. Follow extension documentation to connect to the cloned source repository on your Linux desktop machine.[https://code.visualstudio.com/docs/remote/ssh]


## extension

1. Clangd extension (superior to Microsoft's C++ extension for most of the functionality – code completion, error highlighting, etc). 
    Extension id: llvm-vs-code-extensions.vscode-clangd
2. Official Bazel extension (BazelBuild.vscode-bazel), for syntax highlighting and formatting in BUILD files.
3. Alternative extension for Bazel (devondcarew.bazel-code), only needed if you didn't install the extension above.
4. Microsoft extension for C++ is still useful for the debugging support (id: ms-vscode.cpptools)
5. vscode-proto3 (zxh404.vscode-proto3), same for .proto files.
6. Rewrap (stkb.rewrap) to reflow comment blocks
7. Code Spell Checker (streetsidesoftware.code-spell-checker) if you hate typos in comments
8. Better-Solarized

## clangd
first need to compile the `compile_commands.json` for clangd
ctrl+shift+p: clangd

## workspace settings.json

```json
"settings": {
        "editor.formatOnSave": true,
        "C_Cpp.autocomplete": "Disabled",
        "C_Cpp.formatting": "Disabled",
        "C_Cpp.errorSquiggles": "Disabled",
        "C_Cpp.intelliSenseEngine": "Disabled",
        "C_Cpp.codeFolding": "Disabled",
        "clangd.arguments": [
            "--compile-commands-dir=.",
            "--background-index=false",
            "--enable-config",
            "--clang-tidy",
            "--function-arg-placeholders=false"
        ],
    }
```
