Born out of frustration with the current Ruby deployment practices.

Capistrano is a workhorse, but if you combine it with rvm and bundler, you're
in for a treat.

git-deploy works pretty well, but it's still not seamless and you end up having
to work around it too often for my taste.

heroku solves the deployment process bang on. This command-line utility brings
the same heroku-style deploys convenience to a regular VPS, {cloud-provider}
instance or even bare metal (if you're that hard core).

At [GoSquared](http://www.gosquared.com/), we use this utility to deploy Ruby, PHP
and node.js applications. Here's a Ruby app being deployed:

<img src="http://c2990942.r42.cf0.rackcdn.com/deliver.png" />



## 1 ASSUMPTIONS

These are all good conventions which have been bread over the years from
administrating many different infrastructure setups. You can disregard
everything here and go back to your existing deployment process. Or, you can
fork, add your improvements and contribute towards a modern and efficient
deployment tool that just works.

### 1.1 Ubuntu

Your server is running Ubuntu, preferably 10.04 LTS. Ubuntu + upstart are by no
means the holy grail, but they work very well. Feel free to share your love for
other distros & service managers via pull requests.

### 1.2 Remote logins and privileges

Your local username can gain sudo privileges on the server without being
prompted for a password. Don't login with root. Don't use password logins.

If you're using chef to manage your servers
[sudo-cookbook](https://github.com/opscode/cookbooks/tree/master/sudo) &
[ssh-cookbook](https://github.com/gchef/ssh-cookbook) will work right out of
the box.

### 1.3 Every app gets its own system user

A system user has been created for the app that you'll be delivering. I
personally prefer the same name as the app itself. You should be able to log in
as this user, without any password.

If you're already using chef,
[bootstrap-cookbook](https://github.com/gchef/bootstrap-cookbook) with the
`ruby_apps` recipe will set everything up for you. It handles the entire rvm
integration, even down to configuring the user's bash environment.

### 1.4 [RVM](http://beginrescueend.com/)

You have rvm installed on the server that you'll be delivering your code to. I
prefer system-wide setups in production. Yes, you've guessed it, use chef's
[rvm-cookbook](https://github.com/gchef/rvm-cookbook) for the best experience.

ps: rbenv support is on the roadmap. If you want it right now, fork away.

### 1.5 [Foreman](https://github.com/ddollar/foreman)

Every Ruby app should have this. It allows you to painlessly scale your app
components, [just as if you were running on
Heroku](http://devcenter.heroku.com/articles/procfile). A server-side foreman
would be even nicer, something that will take care of node.js, or any other
type of app. Already on the roadmap.



## 2 INSTALLATION

### 2.1 Check out deliver into `~/.deliver`.

    $ git clone git://github.com/gerhard/deliver.git ~/.deliver

### 2.2 Add `~/.deliver/bin` to your `$PATH` for access to the `deliver` command-line utility

    $ echo 'export PATH="$HOME/.deliver/bin:$PATH"' >> ~/.bash_profile
    # if using zsh
    $ echo 'export PATH="$HOME/.deliver/bin:$PATH"' >> ~/.zshrc 

### 2.3 Source your shell profile

    $ . ~/.bash_profile
    # if using zsh
    $ . ~/.zshrc 

### 2.4 Personalize

There are no fancy generators, you will need to create a `.deliver` file
manually in the root folder of the app that you want delivering. Yes, you got
that right, there's no [`Loudfile` *faux pas*](http://blog.hasmanythrough.com/2011/12/1/i-heard-you-liked-files).

Here's a `.deliver` example:

    APP="squirrel"          # You know, the shipit one

    USER="$APP"             # Can be anything really, but it would be nice if you stuck to this convention

    PORT=7000               # The TCP port on which your app will be listening on (think reverse-proxies)
                            # Not at all scalable, will be revised not before long

    DEPLOY_TO="~$USER/app"  # The location where the app will be 'git pushed' to

    SERVER="shipit"         # The hostname or IP where the app will be delivered

    REMOTE="$USER@$SERVER"  # You will be performing most remote tasks as this user,
                            # e.g. git pushing, bundling etc.
                            # Tasks requiring sudo privileges will be performed as
                            # your local user. See 1.2 from ASSUMPTIONS.



## 3 USAGE

From the root of your project which has been configured via the `.deliver`
file, run:

    $ deliver

### 3.1 Options

<table>
  <tr>
    <td>-v, --version</th>
    <td>current version</td>
  </tr>
  <tr>
    <th>-V, --verbose</th>
    <td>doesn't suppress the output of commands</td>
  </tr>
  <tr>
    <th>-D, --debug</th>
    <td>shows the deliver script as it is being run</td>
  </tr>
</table>



## 4 ROADMAP

The utility has just enough to solve our deployment woes. It's still missing a
few important features which will be added as the need arises. In no particular
order:

* multiple apps in a single repository
* revise the `PORT` option with something more scalable
* multi-server deploys
* don't run the full deploy if nothing has changed
* Post deploy hooks:
  * Campfire
  * Graphite
* rbenv integration (not a priority for me personally)
* better error handling (particularly when remote tasks fail)
* system-wide foreman supporting both node.js & Ruby apps (anything else, fork away)



## 5 LICENSE

(The MIT license)

Copyright (c) 2012 Gerhard Lazu

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies
of the Software, and to permit persons to whom the Software is furnished to do
so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
