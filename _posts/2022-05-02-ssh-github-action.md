---
layout: post
title: Access running Github action with SSH
categories: [github]
---

Sometimes Github actions are failing and it is difficult to reproduce the error locally,
in particular if you have a different OS.

Fortunately we can use the [Debugging with SSH Github action](https://github.com/marketplace/actions/debugging-with-ssh),
make a temporary branch, and add this step before the step that produces an error:

      - name: Setup upterm session
        uses: lhotari/action-upterm@v1
        with:
          ## limits ssh access and adds the ssh public key for the user which triggered the workflow
          limit-access-to-actor: false

**NOTE:** Anybody can connect to this session, so make sure you don't have sensitive data

Create a pull request to trigger execution of the Github action workflow.

Then check the logs of your Linux and Mac OS builds, you should find a connection string of the form:

    ssh <somestring>:<somestring>=@uptermd.upterm.dev

Type it in a terminal to connect to the virtual machine running on Github and debug the issue interactively.

If you get a "Permission denied (public key)" on Linux, see [this workaround](https://github.com/lhotari/action-upterm/issues/9#issuecomment-1060684368), pasted here for convenience (you don't need to add this key to your github account):

```
ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/id_ed25519 -C "yourusername@company"
ssh -i ~/.ssh/id_ed25519 <somestring>:<somestring>=@uptermd.upterm.dev
```

See [the pull request I used for testing](https://github.com/zonca/healpy/pull/4)

Once done, take a look at the current running Actions and cancel any leftover runs.
