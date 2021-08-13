# Intro
Hi! I'm ~~Troy Mclure~~ pcnorden, a bored person that don't like settings up git several times since I always forget something and mess up the whole process.

Due to me needing to set up my git SSH keys several times over right now, I've decided to just write a quick documentation on how to set SSH keys and git up quickly.

I'm using github, so using normal username and password ain't something that I'm gonna do with git, I'm gonna use SSH keys since they are more secure.

## Steps to do

* On a fresh install of ubuntu or alike, installing `build-essentials xclip git` is a really good start.

* Check if you already have ed25519 generated keys with a command like `ls -al ~/.ssh` [More information here](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh/checking-for-existing-ssh-keys)

* If you don't have any, time to generate some!

* In command line, run `ssh-keygen -t ed25519 -C "github@email.here"` where you subsitute the github@email.here with your actual email that you use for github. Save to default location (just press enter), type a passphrase you'll remember or have in a password database and wait for the key generation to be completed.

* Make sure that ssh-agent is running, you can check this with a simple `eval "$(ssh-agent -s)"` which should return a pid to you. If not, [More information is available here](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent).

* Add the SSH key to the ssh-agent with a command something like this: `ssh-add ~/.ssh/id_ed25519`. If you used a custom name, make sure that you change the filename to the custom one instead.

* Copy the public key to your clipboard. In my case I'll just use xclip since it's faster than opening the key file in a text editor. `xclip -selection clipboard < ~/.ssh/id_ed25519.pub` (MAKE SURE IT'S THE `.pub` FILE!)

* Head on over to [this](https://github.com/settings/keys) link while you are signed into your github account in a web browser.

* Press "New SSH key" on the page, add a name that describes what the key is connected to and then paste your public key into the text field called "Key" and finish it all off by pressing the big green "Add SSH key" beneath the key field.

* This is where we deviate from the normal route on the github guide on setting up SSH keys. There are some protections in github that makes sure that you don't expose your email address unless you explicity allow it in settings, but I like that setting so here is what you do now.

* Head on over to [this](https://github.com/settings/emails) link and take note of the primary email address, cause the one that you should take note of is the email that ends with `@users.noreply.github.com`. If you use your normal email address, github will likely complain that you can't `git push` or alike because that would expose your normal email address, so take note of the noreply email address instead.

* Back on the command line, enter `git config --global user.email "numbers+username@users.noreply.github.com` to configure what email to use when pushing or something, and after that enter `git config --global user.name "github username goes here"`

* Now, test your SSH connection to github by issuing `ssh -T git@github.com`. You only want the SSH session to tell you that you've successfully connected to github. They don't actually give a full SSH terminal there, it's just so it's letting you know that your SSH keys are set up correctly.

* The most important part is now, you need to `git clone` the repository you're interested in downloading, but you need to make sure that you clone it via SSH and not the normal .git method. For example, here is me cloning my ReSteel repository: `git clone git@github.com:pcnorden/ReSteel` which when I enter the passphrase for the SSH key we set up a while ago, will download the entire repo to my machine.

If I want to `git push` the repo, I just enter the root directory of the repository folder and just `git status`, fix problems, `git commit -m "commit message here"` and then finally `git push` the repo without needing to do anything really special.
