# AGENTS.md

Neovim configuration built on the **LazyVim** starter template. All config is Lua, distributed via `lazy.nvim`. This is a config repo, not a code library: the "source code" is the editor setup itself.

## What this is

- A personal Neovim config that layers custom plugins/keymaps/options on top of the LazyVim distribution.
- Most defaults come from LazyVim upstream, not from files here. Before assuming an editor behavior is configured in this repo, check whether it's a LazyVim default.
- The git history is trivial (`init` only) and the config is mostly the untouched starter template plus edits in `lua/plugins/plugins.lua`.

## Directory layout

- `init.lua` — entrypoint; only bootstraps `require("config.lazy")`.
- `lua/config/lazy.lua` — bootstraps/installs `lazy.nvim` and runs the plugin `spec = { ... { import = "plugins" } }`.
- `lua/config/options.lua`, `keymaps.lua`, `autocmds.lua` — load on the LazyVim `VeryLazy` event; currently just doc comments (additive customization points). Filenames under this dir are fixed conventions LazyVim expects.
- `lua/plugins/` — every `*.lua` file here is automatically treated as a lazy.nvim spec and merged. This is where virtually all real customization lives.
  - `example.lua` — LazyVim sample spec, **dead on arrival**: first line `if true then return {} end` makes it load nothing. Left as documentation/reference, not active.
  - `plugins.lua` — the active, hand-tuned spec. Holds all real plugin config.
- `lazyvim.json` — LazyVim meta (extras, install version). **JSON has no comments.** To add extras you must edit this and then run `:LazyExtras`.
- `lazy-lock.json` — auto-generated plugin version lockfile; managed by lazy.nvim, don't hand-edit.
- `.neoconf.json` — enables neodev/neoconf library resolution for `lua_ls`.
- `stylua.toml` — LSP style: 2-space indent, 120 col width. Applies to all Lua here.

## Commands

Each file here is a spec; there is no build/test step. Validation is done inside Neovim:

- `nvim` — start; everything loads from here.
- `:Lazy` — lazy.nvim UI. `:Lazy update` syncs `lazy-lock.json` (this is why it shows as modified).
- `:LazyExtras` — toggle upstream extras (from a `lazyvim.json` entry), live.
- `:LazyHome` / `:LazyInstall dev` — for hacking on a plugin locally.
- `:LazyCheck` — dependency health check.
- `:Lazy load <plugin>` — force-load a lazy plugin.
- `:Mason` — manage installed LSP servers/format tools (`:MasonInstallAll` installs the `ensure_installed` list).
- `:CheckHealth` — diagnostics for plugins/LSPs.
- `stylua` — format Lua (2 spaces, 120 cols; already configured).
- No CI, no Makefile, no lint/test/typecheck scripts exist.

## Plugin spec patterns (LazyVim conventions)

Specs use LazyVim's lazy-loading fields; know the difference between load triggers and *merging*:

- **Plain table = merge with defaults** by `vim.tbl_deep_extend` (merges tables, **not lists**). `ensure_installed` in treesitter/mason are lists, so a second spec for the same plugin **overwrites** the list rather than extending it. To extend, use the function form below.
- **`opts = function(_, opts)`** — receives the merged opts so far; mutate in place (e.g. `vim.list_extend(opts.ensure_installed, {...})`). This is the correct way to extend list-valued opts.
- **Returning a new opts table** from `opts = function()` overrides all defaults for that plugin.
- Duplicate spec tables for the same plugin are allowed and merged in order — you'll see several for `nvim-lspconfig`, `nvim-treesitter`, `LazyVim/LazyVim` itself.
- **`config = function()`** fully bypasses LazyVim's config for that plugin (see `nvim-tree` in `plugins.lua`), while `opts` merges into LazyVim's default config.

## Conventions in `plugins.lua`

- Colorscheme set via `LazyVim/LazyVim` `opts.colorscheme` (`"sonokai"` currently); install the theme plugin (e.g. `sainnhe/sonokai`) in the same spec list or the scheme won't exist.
- LSP servers (pyright, gopls) are configured by adding empty `{}` under `neovim/nvim-lspconfig` `opts.servers`; matching binaries auto-install via Mason. Tool binaries (stylua, shellcheck, shfmt, flake8, gopls, goimports, gofumpt, delve) go in `mason-org/mason.nvim` `opts.ensure_installed`.
- Vendor prefix change: this uses `mason-org/mason.nvim` (new org) whereas the stock LazyVim template currently references `williamboman/mason.nvim`. Don't "fix" it.
- git_ignored/dotfiles filters enabled for `nvim-tree`: it shows dotfiles and git-ignored files. Deliberate.
- `kitty-scrollback.nvim` is pinned to `^6.0.0` (major-version pin) — intentional, updates won't auto-bump majors.

## Gotchas

- `example.lua` intentionally no-ops — don't try to make it load.
- Lazyloading: many specs use `event = "VeryLazy"`, `cmd`, `keys` triggers. A plugin configured lazily won't be active until triggered; run its `cmd` or `:Lazy load <name>` to test config.
- LazyVim provides a large keymap/autocmd set by default; adding to `lua/config/keymaps.lua` is additive, but to override/remove a default you must delete it by group (`lazyvim_` prefixed) or shadow the leader key.
- `lazy-lock.json` shows as modified whenever plugins update; committing it is normal and expected.
