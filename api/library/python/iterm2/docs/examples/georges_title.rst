George's Title Algorithm
=========================

This code combines user-defined variables from Shell Integration with a custom session title function. The end result is a snazzy title that includes your git branch:

.. image:: georgesalgo.png
  :height: 32px
  :width: 375px

|
First, you need to install Shell Integration. Add this to your .bashrc:

.. code-block:: bash

    function iterm2_print_user_vars() {
      iterm2_set_user_var gitBranch $((git branch 2> /dev/null) | grep \* | cut -c3-)
      iterm2_set_user_var home $(echo -n "$HOME")
    }

Next, install this script in `~/Library/Application Support/iTerm2/Scripts/AutoLaunch`:

.. code-block:: python

    #!/usr/bin/env python3

    import asyncio
    import iterm2
    import os
    import subprocess
    import sys

    def hostname_dash_f():
	process = subprocess.Popen(["hostname", "-f"], stdout=subprocess.PIPE)
	(output, err) = process.communicate()
	exit_code = process.wait()
	return output.decode('utf-8').rstrip()

    def shortened_hostname(h):
	i = h.find(".")
	if i < 0:
	    return h
	else:
	    return h[:i]

    def make_title(auto_name, profile_name):
	if auto_name != profile_name:
	    return auto_name
	else:
	    return ""

    def make_hostname(hostname, localhost):
	if not hostname:
	    return ""
	short_hostname = shortened_hostname(hostname)
	if short_hostname and short_hostname != localhost:
	    return "➥ " + short_hostname
	return ""

    def make_pwd(user_home, localhome, pwd):
	if pwd:
	    if user_home:
		home = user_home
	    else:
		home = localhome
	    home_prefix = "📂 "
	    if pwd == home:
		home_prefix = ""
		pwd = "🏡"
	    elif pwd.startswith(home):
		pwd = "~" + pwd[len(home):]
	    if pwd:
		return home_prefix + pwd
	return ""

    def make_branch(branch):
	if branch:
	    return " ⎇ " + branch.rstrip()
	return ""

    async def main(connection, argv):
	app = await iterm2.async_get_app(connection)
	localhome = os.environ.get("HOME")
	localhost = hostname_dash_f()

	async def georges_title(pwd, hostname, branch, auto_name, profile_name, tmux_title, user_home):
	    if tmux_title:
		return tmux_title

	    parts = [make_title(auto_name, profile_name),
		     make_hostname(hostname, localhost),
		     make_pwd(user_home, localhome, pwd),
		     make_branch(branch)]
	    return " ".join(list(filter(lambda x: x, parts)))

	defaults = { "pwd":          "session.path?",
		     "hostname":     "session.hostname?",
		     "branch":       "user.gitBranch?",
		     "auto_name":    "session.autoName?",
		     "profile_name": "session.profileName?",
		     "tmux_title":   "session.tmuxWindowTitle?",
		     "user_home":    "user.home?" }
	await app.async_register_rpc_handler("georges_title",
					     georges_title,
					     role=iterm2.RPC_ROLE_SESSION_TITLE,
					     defaults=defaults,
					     display_name="George's Title Algorithm")
	await connection.async_dispatch_until_future(asyncio.Future())

    if __name__ == "__main__":
	iterm2.Connection().run(main, sys.argv)

Finally, select *George's Title Algorithm* in **Prefs > Profiles > General > Title**.

