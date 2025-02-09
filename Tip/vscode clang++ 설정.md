
```json
{
    "workbench.colorTheme": "Default Dark Modern",
    "cmake.configureOnOpen": true,
    "terminal.external.osxExec": "iTerm.app",
    "editor.fontSize": 15,
    "workbench.iconTheme": "material-icon-theme",
    "cmake.showOptionsMovedNotification": false,
    "git.openRepositoryInParentFolders": "always",
    "workbench.startupEditor": "none",
    "windsurf.autocompleteSpeed": "default",
    "editor.defaultFormatter": "llvm-vs-code-extensions.vscode-clangd",
    "editor.formatOnSave": true,
    "C_Cpp.clang_format_fallbackStyle": "{BasedOnStyle: Google, IndentWidth: 2, ColumnLimit: 0, UseTab: Never}",
    "editor.tabSize": 2,
    "editor.insertSpaces": true,
    "editor.detectIndentation": false,
    "windsurf.autoExecutionPolicy": "off",
    "windsurf.chatFontSize": "default",
    "clangd.fallbackFlags": [
        "-std=c++17",
        "-I/Users/psyche95/.asdf/installs/java/zulu-17.56.15/zulu-17.jdk/Contents/Home/include",
        "-I/Users/psyche95/.asdf/installs/java/zulu-17.56.15/zulu-17.jdk/Contents/Home/include/darwin"
    ],
    "C_Cpp.intelliSenseEngine": "disabled"
}
```


```json

{
  "workbench.colorTheme": "Default Dark Modern",
  "git.openRepositoryInParentFolders": "always",
  "cmake.configureOnOpen": false,
  "cmake.pinnedCommands": [
    "workbench.action.tasks.configureTaskRunner",
    "workbench.action.tasks.runTask"
  ],
  "editor.defaultFormatter": "llvm-vs-code-extensions.vscode-clangd",
  "editor.formatOnSave": true,
  "C_Cpp.intelliSenseEngine": "disabled",
  "clangd.fallbackFlags": [
    "--std=c++17",
    "-I//Users/psychehose/.asdf/installs/java/zulu-17.42.19/zulu-17.jdk/Contents/Home/include",
    "-I//Users/psychehose/.asdf/installs/java/zulu-17.42.19/zulu-17.jdk/Contents/Home/include/darwin"
  ],
  "windsurf.autocompleteSpeed": "default",
  "windsurf.enableAutocomplete": true,
  "windsurf.enableSupercomplete": true,
  "windsurf.enableTabToJump": true,
}

```

프로젝트 루트에 .clang-format
```
BasedOnStyle: Google
IndentWidth: 2
ColumnLimit: 0
UseTab: Never
AlwaysBreakTemplateDeclarations: Yes
```