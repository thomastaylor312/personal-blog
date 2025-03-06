---
title: Poor Man's Secret Management in Nix
description: Doing secret management for Nix home-manager the hacky way
date: 2025-03-06 10:00:00-0700
draft: false
image: images/nix-snowflake-colours.svg
categories:
    - Tips and Tricks
tags:
    - nix
---

Ever since [I tried Nix](../a-review-of-nix/index.md), my favorite part of it has been Nix
[home-manager](https://github.com/nix-community/home-manager). As I expanded my usage of it, I
noticed that it was really annoying that I couldn't create many of my config files because they
often contained secrets. I knew tools like [sops](https://github.com/getsops/sops) exist, but
managing yet another key and then committing those files to a repo, even though encrypted, wasn't
really something I wanted to do. On top of that, I use 1Password and their very useful `op` CLI tool
to manage my secrets. There was a cool project called [opsops](https://github.com/vst/opsops) (props
on that name!), but that also just felt like a hassle for adding just a few secrets into places.

So, with that said, I figured I'd show my hacky, poor man's way to adding secrets. 

Basically, this leverages the `home.activation` section available as part of home-manager. This lets
you run scripts with side effects once everything is place. Here is a code snippet of an example
setting up my config file for [`aichat`](https://github.com/sigoden/aichat):

```nix
home.activation = {
    setupAichatConfig = lib.hm.dag.entryAfter [ "writeBoundary" ] ''
            run mkdir -p "$HOME/Library/Application Support/aichat"
            yaml_path="$HOME/Library/Application Support/aichat/config.yaml"
            run rm -f "$yaml_path"
            run cat > "$yaml_path" << 'EOF'
      model: openrouter:openai/gpt-4o
      function_calling: true
      clients:
      - type: openai-compatible
        name: openrouter
        api_base: https://openrouter.ai/api/v1
        api_key: <REPLACE ME>
      EOF
            # On macOS, sed -i requires an extension argument but we need to avoid escaping issues
            run ${pkgs._1password-cli}/bin/op read --account <ACCOUNT_ID_HERE> "op://Private/aichat-openrouter-token/credential" | run xargs -I{} sed -i"" 's/<REPLACE ME>/{}/g' "$yaml_path"
            run chmod 400 "$yaml_path"
    '';
}
```

There are a couple of notes here about how this works. First off, the `lib` here is referring to
`home-manager.lib` which can be passed into your `home.nix` file. The `run` command is a command
provided by Nix that ensures you don't run these commands when running in a dry run mode. This must
be run after the `writeBoundary` event, which is why that exists in the `entryAfter` line.
Otherwise, this is just pretty much normal bash. What we do here is pass in the `op` cli from the
nix package we are installing, explicitly pass an account ID (this matters when you're doing
something on a work machine where you might have multiple accounts), and then use sed to replace the
value in the file. Please note that you do need to have 1Password desktop app installed and the CLI
integration checkbox checked for this to work. For security purposes, and to imitate Nix, we then
set the file to readonly for the user.

And that is that! There are probably other, better ways to do this, so please reach out if you want
to correct me here. Otherwise, I hope this is a useful tip for people who might be doing something
similar with Nix
