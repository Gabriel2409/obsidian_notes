#neovim

https://neovim.io/doc/user/lsp.html#lsp

## Install
### No plugins install

First we need to install our language server, for ex `luals` so that the command `lua-language-server` is available to neovim. 

Then in `init.lua`:

```lua
-- full config of the luals language server
vim.config['luals'] = {
  -- Command and arguments to start the server.
  cmd = { 'lua-language-server' },
  -- Filetypes to automatically attach to.
  filetypes = { 'lua' },
  -- Sets the "root directory" to the parent directory of the file in the
  -- current buffer that contains either a ".luarc.json" or a
  -- ".luarc.jsonc" file. Files that share a root directory will reuse
  -- the connection to the same LSP server.
  root_markers = { '.luarc.json', '.luarc.jsonc' },
  -- Specific settings to send to the server. The schema for this is
  -- defined by the server. For example the schema for lua-language-server
  -- can be found here https://raw.githubusercontent.com/LuaLS/vscode-lua/master/setting/schema.json
  settings = {
    Lua = {
      runtime = {
        version = 'LuaJIT',
      },
    },
  },
}

-- Attach the LSP for lua files automatically
vim.api.nvim_create_autocmd('Filetype', {
  pattern = 'lua',
  callback = function()
	-- here we enable the configuration but it is not actually needed 
    vim.lsp.enable 'luals'
    -- start the lsp server with the correct config
    vim.lsp.start(vim.lsp.config['luals'])
  end,
})
```

When opening a lua file, run `:checkhealth vim.lsp`, you should see the active client and the enabled config.

Commands such as `gd` work. The reason it does not appear when doing `:map` is that it is a default command (do `:help gd` to see it)


Alternatively, instead of putting this in `init.lua`, 
we can create folder in runtime_path (simple is basically at the root of config), in 
`lsp/luals.lua` and put
```lua
-- here we don't need to specify the luals key, neovim infers it from the filename
return {
  cmd = { 'lua-language-server' },
  ...
}
```

Similary to attach, instead of an autocommand, in `ftplugin/lua.lua`:

```lua
-- from the name of the file lua.lua, neovim runs these commands when opening a .lua file
vim.lsp.enable 'luals'
vim.lsp.start(vim.lsp.config['luals'])
```
### With nvim-lspconfig
You still need to install lua-language-server manually, but you can then just put this in your plugin folder

```lua
return {
  'neovim/nvim-lspconfig',
  config = function()
    vim.lsp.config('lua_ls',{...})
  end,
}
```
By default, it comes with the correct filetypes so no need to create the `lsp/lua.lua` anymore

More info on all available servers: `:help lspconfig-all`
More info on setup: `:help lspconfig-setup`: basically `setup` can use all the keys from `vim.lsp.start` but it also accept extra keys. usually we overwrite the settings key: 
```lua
-- example
  vim.lsp.config('pyright',{
    settings = {
      pyright = {
          disableLanguageServices = true,
          disableOrganizeImports  = true,
        }
    }
  })
```

### With mason
Now you don't even need to install anything manually
Mason install to `standard_path/mason/bin` so that all dependencies are available at the same place.
It can be used with `mason-lspconfig.nvim` which serves as a bridge between lsp-config and mason
```lua
return {
  {
    'neovim/nvim-lspconfig',
    dependencies = {
      -- Automatically install LSPs and related tools to stdpath for neovim
      'mason-org,
      'mason-org',
    },
    config = function()
      -- make sure to call in this order
      require('mason').setup()
      require('mason-lspconfig').setup {
        -- force install lua_ls
        ensure_installed = { 'lua_ls' },
      }
      -- can put all the lspconfig here
      vim.lsp.config('lua_ls', {})
    end,
  },
}
```
### With LazyVim
LazyVim does some merging behind the scenes but I have the impression it also changes the way to install them. 
After digging in https://github.com/LazyVim/LazyVim/blob/main/lua/lazyvim/plugins/lsp/init.lua, i came to the following conclusions: 
```lua
-- in a plugin, just add the servers you are interested in 
return {
  'neovim/nvim-lspconfig',
  opts = {
    servers = {
      lua_ls = {
        -- all of the keys in setup mentioned above such as settings
      },
    },
    setup = {
      lua_ls = function()
        -- Additional server setup here. 
      end,
    },
  },
}
```

## How it works
Neovim communicates with the server using Json rpc. 
Example:
- set `:lua vim.lsp.set_log_level("DEBUG")`
- Hover by running `:lua vim.lsp.buf.hover()`
- Go to the log path, get it with `:lua vim.print(vim.lsp.get_log_path())`

You should see something like
```bash
# Step 1: Client Sends the Hover Request
[DEBUG][2025-02-14 17:39:08] ...m/lsp/client.lua:677 "LSP[lua_ls]" "client.request" 1 "textDocument/hover" { position = { character = 37, line = 16 }, textDocument = { uri = "file:///home/gabriel/.config/nvim/nvim-raw/lua/plugins/lsp_config.lua" } } <function 1> 225 

# Step 2: RPC Layer Sends the Request to the LSP Server
[DEBUG][2025-02-14 17:39:08] .../vim/lsp/rpc.lua:277 "rpc.send" { id = 1529, jsonrpc = "2.0", method = "textDocument/hover", params = { position = { character = 37, line = 16 }, textDocument = { uri = "file:///home/gabriel/.config/nvim/nvim-raw/lua/plugins/lsp_config.lua" } } } 

# Step 3: LSP Server Responds to the Hover Request
[DEBUG][2025-02-14 17:39:08] .../vim/lsp/rpc.lua:393 "rpc.receive" { id = 1529, jsonrpc = "2.0", result = { contents = { kind = "markdown", value = "
lua\n(field) ?.lua_ls.setup: unknown\n" }, maxLevel = 0, range = { ["end"] = { character = 39, line = 16 }, start = { character = 34, line = 16 } } } }

```

When you first open a doc, client sends a `textDocument/didOpen` containing the full text of the file to the server. However, when file is updated, it only sends updtes with `textDocument/didChange`

An LSP server must track state per client because each client has its own. Note that a buffer can have multiple clients attached (each neovim instance is a client)