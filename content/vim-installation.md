+++
title = "Installing vim/neovim using Nixpkgs"
date = 2020-06-25
+++

This is a simplified nix expression

```nix
{ pkgs ? import <nixpkgs> {} }:

let config = {
        customRC = builtins.readFile ./nvimrc;
        packages.myVimPackage = with pkgs.vimPlugins; {
                start = [
                        vim-surround
                        haskell-vim
                        supertab
                        tabular
                        vim-gitgutter
                        syntastic
                        vim-airline
                        vim-airline-themes
                        ];
                };
        };
in
pkgs.neovim.override {
        vimAlias = true;
        configure = config;
}
```

This is similar to bjornfor and others.