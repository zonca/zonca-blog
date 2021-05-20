---
layout: post
title: Migrate from Google Docs to Overleaf and Github
categories: [latex,git]
---

Once a document on Google Docs becomes too big and complicated to make sure it is consistent, it is a good idea to migrate it to Latex.
In particular we can support both Google Docs style editing via Overleaf and Pull Request reviewing via Github.

## Convert Google Docs document to Latex

I was impressed about how well [docx2latex.com](https://docx2latex.com) worked,
it correctly imported:

* sectioning based on title styles in Google Docs
* all images
* tables!
* page header with image!

Just download the `docx` from Google Docs and upload to `docx2latex`,
if needs to be < 10 MB, so remove large images before exporting from Google Docs if it is bigger.

Then you will be able to download a zip with the Latex source and images.


## Import to Github

If you don't need Github, you can directly upload the zip archive to Overleaf and be done.

However, it is nice to have a backup on Github, and support users that prefer editing latex outside of their browser...

So create a repository, even private and upload the content of the archive after having removed the PDF artifacts.

## Integration with Overleaf

Main issue the Github integration with Overleaf is they require write access to all your repositories (!! crazy, I know).

So let's create a new Github account, I called mine `zoncaoverleafbot` and invite the new user to be a Collaborator to the repository.

Then login to Overleaf with your real account, click on "Import from Github", and when you are redirected to Github to link your account, link **the overleafbot** account instead, which only has access to the Latex repositories.

Now on Overleaf click on "New project" and "Import from Github".

If your repository doesn't show in the list, it is probably because it belongs to an organization, in that case [see the Github help to fix it](https://docs.github.com/en/github/setting-up-and-managing-your-github-user-account/managing-your-membership-in-organizations/requesting-organization-approval-for-oauth-apps).

## Synchronization

Synchronization of Overleaf and Github is always done on Overleaf.

Once in a while, you can manually synchronize from/to Github using the Github button.
If you share the project by sending someone the "Edit" link, they can also use the Github integration that the project owner has configured, so no need for them to link their Github account.
Best would be for each Overleaf user, each time they login, to pull changes from Github before starting to work and push their changes with a commit message when they are done.
