## UI Extensions

- [nvim-dap-ui][7] - Experimental UI for nvim-dap
- [nvim-dap-virtual-text][1] - Inlines the values for variables as virtual text using treesitter.
- [telescope-dap.nvim][4] - Integration for nvim-dap with telescope.nvim
- [fzf-lua][fzf-lua] - UI integration for nvim-dap with `fzf`
- [cmp-dap][cmp-dap] - nvim-cmp source for using DAP completions inside the REPL.

## Language specific extensions

- [nvim-dap-python][2] - An extension for nvim-dap providing default configurations for python and methods to debug individual test methods or classes.
- [nvim-dap-go][8] - Allows debugging individual Go tests and provides default configurations for launching [delve][9] automatically.
- [nvim-jdtls][3] - Extensions for the built-in language server client in Neovim for eclipse.jdt.ls that also provides debugging support for Java via nvim-dap.
- [nvim-metals][5] - Scala/Metals plugin for the Neovim built-in LSP client. Integrates with nvim-dap to provide debug functionality.
- [jester][jester] - A Neovim plugin to easily run and debug Jest tests
- [rust-tools.nvim][rust-tools] - Tools for better development in rust using neovim
- [nvim-dap-ruby][dap-ruby] - An extension providing adapter definition and configurations for [debug.rb][debug.rb]
- [nvim-dap-vscode-js][dap-vscode-js] - An extension providing configurations for the [official vscode js debugger][vscode-js-debug], which runs javascript/typescript (both in-browser and node.js)

## Polyglot language extensions

- [vim-ulttest][6] - Test runner building upon vim-test with nvim-dap support.


[1]: https://github.com/theHamsta/nvim-dap-virtual-text
[2]: https://github.com/mfussenegger/nvim-dap-python
[3]: https://github.com/mfussenegger/nvim-jdtls
[4]: https://github.com/nvim-telescope/telescope-dap.nvim
[5]: https://github.com/scalameta/nvim-metals
[6]: https://github.com/rcarriga/vim-ultest
[7]: https://github.com/rcarriga/nvim-dap-ui
[8]: https://github.com/leoluz/nvim-dap-go
[9]: https://github.com/go-delve/delve
[jester]: https://github.com/David-Kunz/jester
[rust-tools]: https://github.com/simrat39/rust-tools.nvim
[dap-ruby]: https://github.com/suketa/nvim-dap-ruby
[debug.rb]: https://github.com/ruby/debug
[fzf-lua]: https://github.com/ibhagwan/fzf-lua
[cmp-dap]: https://github.com/rcarriga/cmp-dap
[dap-vscode-js]: https://github.com/mxsdev/nvim-dap-vscode-js
[vscode-js-debug]: https://github.com/microsoft/vscode-js-debug