---
layout: post
title:  "GPG signature of commits in GitHub"
date:   2021-09-23 23:59:00 +0100
categories: github git gpg bash
tags:   GitHub git gpg
---
GitHub is a web-based version-control and collaboration platform for software developers. I use it all the time to version the soruce code of my PhD project. The GRIAC and RBMB bioinformatical groups both have a github page setup as well, even though we don't use it very much. But if we were to work together even more, we should be able to trust the commits we make to our collaborative repositories. For that reason, GitHub had instroduced signed commits. Below is a script to make a GPG signature and to add this to your local Git configuration to automatically sign your commits. 
<!--more-->

## Mac / Linux

```BASH
EMAIL="9002122+vanNijnatten@users.noreply.github.com" # See https://github.com/settings/emails
SECRET_KEYID="2F01902897288144" # See output of "gpg --list-secret-keys --keyid-format LONG"


brew install gnupg
brew install pinentry-mac


echo "pinentry-program /usr/local/bin/pinentry-mac" >> ~/.gnupg/gpg-agent.conf
killall gpg-agent


gpg --full-generate-key
gpg --list-secret-keys --keyid-format LONG "$EMAIL"
gpg --armor --output gpg_github_pub.asc --export "$EMAIL"
cat gpg_github_pub.asc


# If you do something wrong while creating the GPG key, delete it:
gpg --list-secret-keys --keyid-format LONG "$EMAIL"
# sec   rsa4096/$SECRET_KEYID 2021-09-24 [SC] [expires: 2031-09-22]
#       RANDOMNUMBERSANDLETTERSRANDOMNUMBERSANDL
# uid                 [ultimate] Jos van Nijnatten (This is for GitHub only.) <$EMAIL>
# ssb   rsa4096/SOMEOTHERIDENTIF 2021-09-24 [E] [expires: 2031-09-22]
gpg --delete-secret-key "$SECRET_KEYID"


git config --list
git config --global user.email "$EMAIL"
git config --global user.signingKey "$SECRET_KEYID"
git config --global commit.gpgsign true
git config --global gpg.program gpg


vim "README.MD"
git commit -m "R: added commented line for auto stacktrace"
git push
git show head --show-signature
git log --show-signature -1
```

## Windows
[Tutorial](https://neurotechnics.com/blog/configure-gpg-to-sign-git-commits-in-windows/)
Remember to do the GPG related commands in Powershell and the Git related commands in Bash.

