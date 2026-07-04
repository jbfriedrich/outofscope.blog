---
title: Freshly brewed
date: 2024-03-22T20:54:55
tags: [automation, mac, software]
languages: [en]
---

I do not reinstall my systems that often, so I always forget how to properly [dump to and restore from my Brewfile](https://github.com/Homebrew/homebrew-bundle). Maybe this post will help me to not forget (which I will), or at least help me jog my memory in case it does slip my mind again.

We would want to dump everything from the [Mac App Store](https://www.apple.com/app-store/), as well as all [Virtual Studio Code extensions](https://code.visualstudio.com/docs/editor/extension-marketplace), all our [Casks](https://formulae.brew.sh/cask/) and "[Taps](https://docs.brew.sh/Taps). We save it all to our iCloud Drive – not because we like iCloud that much, but because this is immediately available after the Mac is installed and the base configuration is completed:

```shell
brew bundle dump --mas --vscode --describe --taps --casks --brews --file ~/Library/Mobile\ Documents/com~apple~CloudDocs/Brewfile
```

On the new Mac, we just need to install brew and install everything from the Brewfile:

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
[...]
brew bundle install --file ~/Library/Mobile\ Documents/com~apple~CloudDocs/Brewfile
```

Brew will now install everything from the Brewfile. If you include apps from the Mac App Store, like I do, make sure that you are signed-in to the Mac App Store before installing everything via "brew bundle".

That's it, done!
