# Configuration Cookbook

This page is for various configuration snippets that might be useful.


## Pick a process

Some debug adapters support attaching to a running process. If you want to have
a "pick pid" dialog, you can use the `pick_process` utils function in your
configuration. For example, in a configuration using the ``lldb-vscode`` debug
adapter, it can be used like this:


```lua
dap.configurations.cpp = {
    {
      -- If you get an "Operation not permitted" error using this, try disabling YAMA:
      --  echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope
      name = "Attach to process",
      type = 'cpp',  -- Adjust this to match your adapter name (`dap.adapters.<name>`)
      request = 'attach',
      pid = require('dap.utils').pick_process,
      args = {},
    },
}
```

See `:help dap-configuration` for more information about nvim-dap configuration.



## Map `K` to hover while session is active.


```lua
local dap = require('dap')
local api = vim.api
local keymap_restore = {}
dap.listeners.after['event_initialized']['me'] = function()
  for _, buf in pairs(api.nvim_list_bufs()) do
    local keymaps = api.nvim_buf_get_keymap(buf, 'n')
    for _, keymap in pairs(keymaps) do
      if keymap.lhs == "K" then
        table.insert(keymap_restore, keymap)
        api.nvim_buf_del_keymap(buf, 'n', 'K')
      end
    end
  end
  api.nvim_set_keymap(
    'n', 'K', '<Cmd>lua require("dap.ui.widgets").hover()<CR>', { silent = true })
end

dap.listeners.after['event_terminated']['me'] = function()
  for _, keymap in pairs(keymap_restore) do
    api.nvim_buf_set_keymap(
      keymap.buffer,
      keymap.mode,
      keymap.lhs,
      keymap.rhs,
      { silent = keymap.silent == 1 }
    )
  end
  keymap_restore = {}
end
```


## Reload `dap.configuration` before starting a debug session

If you configure the `dap.configurations` table in a Lua module and load that
module via a `require` call, the module gets cached. If you then modify the
module with the configurations the changes won't be picked up automatically.

There are two options to force load the changes:

### 1. Read the file via `:luafile`:

You could add something like `autocmd BufWritePost ~/your_dap_config_file :luafile %` in `init.vim`.


### 2. Clear the Lua package cache and reload the configuration module:

Assuming the following conditions:

- The code below is in the file `~/.config/nvim/lua/dap_util.lua`
- `dap.configurations` is set in `~/.config/nvim/lua/dap_config.lua`

```lua
local M = {}
local dap =  require'dap'

function M.reload_continue()
  package.loaded['dap_config'] = nil
  require('dap_config')
  dap.continue()
end

local opts = { noremap=false, silent=true }

-- <Leader>ec to continue
vim.api.nvim_buf_set_keymap( 0, 'n', '<Leader>ec',
       '<cmd>lua require"dap".continue()<CR>', opts)

-- <Leader>eC to reload and then continue
vim.api.nvim_buf_set_keymap(0, 'n', '<Leader>eC',
    '<cmd>lua require"dap_setup".reload_continue()<CR>', opts)

return M
```

## Making debugging .NET easier

When debugging .NET projects, you sometimes need to rebuild it. You also don't want to input the path to `dll` and/or the path to your `proj` file.

TL;DR - replace your config with the config below, it should be intuitive. Otherwise, read a [detailed description](#detailed-description).

```lua
vim.g.dotnet_build_project = function()
    local default_path = vim.fn.getcwd() .. '/'
    if vim.g['dotnet_last_proj_path'] ~= nil then
        default_path = vim.g['dotnet_last_proj_path']
    end
    local path = vim.fn.input('Path to your *proj file', default_path, 'file')
    vim.g['dotnet_last_proj_path'] = path
    local cmd = 'dotnet build -c Debug ' .. path .. ' > /dev/null'
    print('')
    print('Cmd to execute: ' .. cmd)
    local f = os.execute(cmd)
    if f == 0 then
        print('\nBuild: ✔️ ')
    else
        print('\nBuild: ❌ (code: ' .. f .. ')')
    end
end

vim.g.dotnet_get_dll_path = function()
    local request = function()
        return vim.fn.input('Path to dll', vim.fn.getcwd() .. '/bin/Debug/', 'file')
    end

    if vim.g['dotnet_last_dll_path'] == nil then
        vim.g['dotnet_last_dll_path'] = request()
    else
        if vim.fn.confirm('Do you want to change the path to dll?\n' .. vim.g['dotnet_last_dll_path'], '&yes\n&no', 2) == 1 then
            vim.g['dotnet_last_dll_path'] = request()
        end
    end

    return vim.g['dotnet_last_dll_path']
end

local config = {
  {
    type = "coreclr",
    name = "launch - netcoredbg",
    request = "launch",
    program = function()
        if vim.fn.confirm('Should I recompile first?', '&yes\n&no', 2) == 1 then
            vim.g.dotnet_build_project()
        end
        return vim.g.dotnet_get_dll_path()
    end,
  },
}

dap.configurations.cs = config
dap.configurations.fsharp = config
```

### Detailed description

So basically, when you start a debug session, the dialog will look like this:

#### Step 1.

`Should I recompile first? y/n`
- `y`
  - `Path to your *proj file` (if you already input once, it will substitute it ; but you still can change it)
- `n`

#### Step 2.

If you didn't specify the dll path yet:
- `Path to dll`

If you did:
- `Do you want to change the path to dll?` (it previews your previous dll path)
  - `y`
    - `Path to dll`
  - `n`

### Pro tip

You can bind building the project to some hotkey:
```lua
vim.api.nvim_set_keymap('n', '<C-b>', ':lua vim.g.dotnet_build_project()<CR>', { noremap = true, silent = true })
```