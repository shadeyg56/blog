+++
title = 'How the xz backdoor highlights a major flaw in Nix'
date = 2024-04-02T21:09:17-05:00
draft = false
+++

## Background

On Friday, March 29th, 2024, a historical and sophisticated security vulnerability [(CVE-2024-3094)](https://nvd.nist.gov/vuln/detail/CVE-2024-3094) was discovered in the XZ Utils package and liblzma api in version 5.6.0 and 5.6.1. While this vulerability mostly affects Debian and RedHat distributions, there was some interesting discussion regarding xz and Nix.

## How did this affect Nix and NixOS?

The truth is not a whole lot in reality. I saw conflicting reports, but supposedly, the tarballs of xz that Nix downloads were not infected. Even if Nix did install the infected tarballs, since Nix is not FHS compliant, this likely would not be an issue to most Nix installations. Either way, its not a great idea at all to have a package installed with a major CVE. 

The issue is how security updates are handled. Nixpkgs uses Nix's CI system known as [Hydra](https://github.com/NixOS/hydra). For each branch, a number of tests are ran to ensure everything builds correctly. This is a very lengthy process due to the number of packages available in the Nixpkgs repo. If Hydra takes a while to get a succesful run, nixos-unstable will start to lag behind. At the time of writing this, it has been nearly 5 days and the reversion to xz 5.4.6 has still **NOT** made its way into the unstable branch. This means users such as myself who use the unstable branch for all of their package will still be pulling the (potentially) infected xz tarballs onto their machines!

This is a pretty big deal and because of the way Nixpkgs is set up, it is not uncommon for security updates to get pushed back because of CI failures. While Nix was seemingly unaffected by this attack, what if in the future we're not so lucky, and an attack occurs that targets Nix and NixOS specifically? Certainly nobody would want this package remaining in nixpkgs for multiple days. While one could argue that this is simply a potential consequence of using the unstable branch, nixos-unstable has a history of actually being quite stable due to the CI and most people I've seen using Nix tend to run the unstable branch. There needs to be some sort of priority in the nixpkgs system for these major security updates

## To unstable or not to unstable?
This incident has really made me wonder if running the unstable branch is a great idea or not. Being on the bleeding edge is important for many people after all. Earlier today, I switched over to `nixos-unstable-small`-a branch that is subject to less Hydra tests-as the xz reversion has already made its way to this branch. The reason people don't use this branch as often is because there are less binaries available in the Nix cache so more building must be done when updating and it's more prone to build failures. 

To combat the build times, I use a [Github action](https://github.com/shadeyg56/nixos-config/blob/master/.github/workflows/update-flake.yml) on my repo that updates my `flake.lock` file every 24 hours and another [action](https://github.com/shadeyg56/nixos-config/blob/master/.github/workflows/build-nix.yml) that builds the config on every push and uploads binaries to my own [Cachix](https://www.cachix.org/) cache. Then when I go to update my system, I can pull from my own binary cache in case packages aren't available in the official Nix cache. 
In terms of the build failures, while it is true that the unstable-small branch is prone to more, it also updates much faster. When a build failure reaches the regular unstable, it often takes much longer for a patch to make its way there. I think I'm going to try this setup for a while and see how it goes.

It also seems reasonable to pull most packages from the stable branch and have a seperate input for the unstable branch where certain packages that need to be on a more recent version are pulled, but I have not tried this for myself.

## Conclusion
While most likely nothing will change, I think the nixpkgs maintainers need to consider for the future how security updates are handled and that there is a lack of priority in getting these updates onto large channels. Most open-source contributers are trustworthy, but there are always bad actors and we never know when a serious exploit could be targeted at Nix users
