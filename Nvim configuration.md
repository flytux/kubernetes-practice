# Install and congifure Nvim and plugins

```bash
# Install nvim
curl -LO https://github.com/neovim/neovim/releases/latest/download/nvim.appimage
sudo chmod u+x nvim.appimage && sudo mv nvim.appimage /usr/local/bin

# Install nvim kickstarter
# Uncomment custom plugin config
# Install lua plugins

# filetree

# toggle terminal

vi ~/.config/nvim/lua/custom/plugins/toggleterm.lua

return {
  "akinsho/toggleterm.nvim",
  config = function()
  require("toggleterm").setup({
    open_mapping = [[<leader>t]],
    shade_terminals = false,
    shell = "zsh --login",
  })
  end,
  keys = {
    { "<leader>t", "<Cmd>ToggleTerm<Cr>", desc = "Terminal" },
  },
}

# comment

