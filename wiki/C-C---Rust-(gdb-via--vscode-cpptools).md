# C/C++/Rust (gdb via  vscode-cpptools)

Configuration examples are in Lua. See `:help lua-commands` if your Neovim setup so far uses a `init.vim` file.

## Installation

Install [vscode-cpptools](https://github.com/Microsoft/vscode-cpptools):

- Download the [VS Code extension](https://github.com/microsoft/vscode-cpptools/releases).
- Unpack it. `.vsix` is a zip file and you can use `unzip` to extract the contents.
- Ensure `extension/debugAdapters/bin/OpenDebugAD7` is executable.



## Adapter definition

```lua
local dap = require('dap')
dap.adapters.cppdbg = {
  id = 'cppdbg',
  type = 'executable',
  command = '/absolute/path/to/cpptools/extension/debugAdapters/bin/OpenDebugAD7',
}
```


If you're on Windows, use this definition instead:

```lua
local dap = require('dap')
dap.adapters.cppdbg = {
  id = 'cppdbg',
  type = 'executable',
  command = 'C:\\absolute\\path\\to\\cpptools\\extension\\debugAdapters\\bin\\OpenDebugAD7.exe',
  options = {
    detached = false
  }
}
```


## Configuration

The [VSCode C/C++ documentation][vscode_docs] contains a full reference for all options supported by the debug adapter.


Common configuration examples:


```lua
local dap = require('dap')
dap.configurations.cpp = {
  {
    name = "Launch file",
    type = "cppdbg",
    request = "launch",
    program = function()
      return vim.fn.input('Path to executable: ', vim.fn.getcwd() .. '/', 'file')
    end,
    cwd = '${workspaceFolder}',
    stopAtEntry = true,
  },
  {
    name = 'Attach to gdbserver :1234',
    type = 'cppdbg',
    request = 'launch',
    MIMode = 'gdb',
    miDebuggerServerAddress = 'localhost:1234',
    miDebuggerPath = '/usr/bin/gdb',
    cwd = '${workspaceFolder}',
    program = function()
      return vim.fn.input('Path to executable: ', vim.fn.getcwd() .. '/', 'file')
    end,
  },
}
```

### Enable pretty-printing

Some versions of GDB ship with [pretty-printing](https://sourceware.org/gdb/current/onlinedocs/gdb/Pretty_002dPrinter-Introduction.html#Pretty_002dPrinter-Introduction) support, which eases the debugging of strings and STL containers. To enable such feature with nvim-dap, include the following setting in **each configuration**:

```
setupCommands = {  
  { 
     text = '-enable-pretty-printing',
     description =  'enable pretty printing',
     ignoreFailures = false 
  },
},

```

### gdb commands

You can execute gdb commands in the `dap-repl` if you prefix them with `-exec`.

Some examples:

```
dap> -exec info registers
rax            0x7                 7
rbx            0x0                 0
rcx            0x7ffff7e80257      140737352565335
rdx            0x1                 1
rsi            0x1                 1
rdi            0x7ffff7f7b570      140737353594224
rbp            0x7fffffffdb00      0x7fffffffdb00
rsp            0x7fffffffdaf0      0x7fffffffdaf0
```

```
dap> -exec print &x
$2 = (int *) 0x7fffffffdaf8
```


### Re-using configuration

If you want to use this debug adapter for other languages, you can re-use the configurations:


```lua
dap.configurations.c = dap.configurations.cpp
dap.configurations.rust = dap.configurations.cpp
```


The executables that you want to debug need to be compiled with debug symbols.

[vscode_docs]: https://code.visualstudio.com/docs/cpp/launch-json-reference
