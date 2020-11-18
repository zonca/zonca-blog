---
layout: post
title: Paper review workflow with Overleaf, git and Google Docs
categories: [latex,git]
---

In this blog post I propose a workflow to use the collaborative features of Google Docs
to review a paper being written in Overleaf (unless you have Overleaf Pro which has
change tracking embedded).
The key point is that we paste the Latex source to a Google Document (see an [example here](https://docs.google.com/document/d/1rOksxkZUOdNW1alA6lqXg1xre4Q7bV6oyCVoIGGh0Z8/edit?usp=sharing) so we can
suggest changes and then push it back to Overleaf programmatically using `wget` and `git`.

## Workflow

Create first the paper in Overleaf

Next we can configure `git` access, see more details [on Overleaf](https://www.overleaf.com/learn/how-to/Using_Git_and_GitHub):

Linking Github requires too much permissions, so better just activate plain Git access,
this will provide a link to clone it:

    git clone https://git.overleaf.com/xxxxxxxxxxxxxxxxxxxxxxxx overleaf-paper

Here we can make local changes and push them back to Overleaf (you can configure the git credential
helper to store the username and password).

Now the main authors can develop the paper using a mix of local editing and git push or online editing in Overleaf,
optionally they could also push to Github.

## Review round on Google Docs

When the paper is ready for a round of review with co-authors or other reviewers, the main authors can
circulate the PDF and copy-paste the Latex source into a Google Doc. It can also be useful to paste
the images, but not strictly necessary, the reviewers can look into the PDF.

Now the reviewers can read the PDF source and make suggested edits and comments on Google Doc,
"Review changes" is also available on Overleaf, but just for paid accounts.

### Synchronization from Google Docs to Overleaf

As the review progresses, some changes are merged using the Google Docs review functionality,
they can be programmatically be merged into Overleaf going through `git`.

Create a `download_google_doc.sh` script (Linux, but could work on Mac as well):

    KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    wget -O - https://docs.google.com/document/d/$KEY/export?format=txt | dos2unix | cat -s > main.tex

it downloads a text version of the Google Doc document, fixes line endings and spurious repeated blank lines
and overwrites `main.tex`.

This is executed with `bash download_google_doc.sh` and then reviewed and merged with the standard `git` workflow.
Finally pushed back to Overleaf with:

    git push

This is useful for example while the review progresses, we can create updated PDF versions.

### Synchronization from Overleaf/git to Google Docs

This is not doable, because writing into Google Docs erases the comments.
Therefore it would be better to organize review into rounds, 1 Google doc per round.
So we never need to update Google Doc with changes made on Overleaf.

However, while the review is progressing in Google Doc, the main authors could keep working in Overleaf/git
and then merge the changes from Google Doc as they come in relying on `git`.
The issue is that it is not easy to propagate their changes to the current round of review on Google Doc.
