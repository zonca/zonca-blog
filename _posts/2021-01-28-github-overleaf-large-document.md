---
layout: post
title: Coordinate a large Latex document with multiple Overleaf projects and Github
categories: [latex,git]
---

In case we need to build a large document (~hundreds of pages) and we have a large number of collaborators (more than 50), it is convenient to have each section of the document be a
different Overleaf project. Also, we don't want to rely on Overleaf only, we prefer backup of everything to Github.

Then have a Github repository that includes all of those as `git submodules` and can build the whole document together automatically using Github Actions and provides the PDF. We cannot use Overleaf here because it doesn't support `submodules`.

The setup is based on the [Overleaf tutorial on multi-page Latex](https://www.overleaf.com/learn/latex/Multi-file_LaTeX_projects#The_standalone_package) using the "Standalone package" setup, and then splitting it into separate repositories.

See [this Overleaf project](https://www.overleaf.com/read/xfcwyzncxhjh) for the project configuration before being split into separate projects.

## Configuration of the Github repository

Here you can use your official Github account.

Create a repository on Github for the main document and for each of the subsections

See for example:

* <https://github.com/zonca/overleaf_largedoc_main>
* <https://github.com/zonca/overleaf_largedoc_intro>
* <https://github.com/zonca/overleaf_largedoc_section_1>

Each repository has a `main.tex` file in the root folder which contains the
content of that section.

In the main repository, we configure `git submodules` for each section, e.g.:

    git submodule add git@github.com:zonca/overleaf_largedoc_intro.git 0_introduction

Now you should double-check you can compile locally the document with `pdflatex`.

## Automation with Github Actions

I have configured Github Actions to run at every commit just in the main repository, see [the details](https://github.com/zonca/overleaf_largedoc_main/blob/master/.github/workflows/build_latex.yml):

* At each commit, Github runs `latexmk` to build the full PDF, then attaches it to that run, they can be downloaded from the Github Actions tab, see [the artifacts tab at the end of this run](https://github.com/zonca/overleaf_largedoc_main/actions/runs/519568514).
* Whenever you create a `tag` in the repository, for example `1.0` or `2021.01.28`, Github creates a release with the PDF attached (named after the release), see [an example](https://github.com/zonca/overleaf_largedoc_main/releases/tag/2021.01.28)
* In the main repository, I prepared a script to update all the sections to their latest commit, see [`update_sections.sh`](https://github.com/zonca/overleaf_largedoc_main/blob/master/update_sections.sh). This can be also triggered via web by manually running the ["Update all sections" workflow](https://github.com/zonca/overleaf_largedoc_main/actions?query=workflow%3A%22Update+all+sections%22) (click on the "Run workflow" button and choose the `master` branch). This creates a new commit therefore triggers creation of the PDF. It is also configured to run every morning at 7am PT.

This should work also for private repositories, both Free and Team (academic) organization on Github have 2000+ Github action minutes a month. You will need to switch plan if you have a Legacy account which doesn't offer Github Actions on private repositories.

## Integration with Overleaf

Main issue the Github integration with Overleaf is they require write access to all your repositories (!! crazy, I know).

So let's create a new Github account, I called mine `zoncaoverleafbot` and invite the new user to be a Collaborator to each of the sections repositories.

Then login to Overleaf with your real account, click on "Import from Github", and when you are redirected to Github to link your account, link **the overleafbot** account instead, which only has access to the sections repositories.

Now create a Overleaf project for each document section, Overleaf can build independent PDF for each of the subsection.

Once in a while, you can manually synchronize from/to Github using the Github button.
If you share the project by sending someone the "Edit" link, they can also use the Github integration that the project owner has configured, so no need for them to link their Github account.
Best would be for each Overleaf user, each time they login, to pull changes from Github before starting to work and push their changes with a commit message when they are done.


Moreover, Overleaf also has the standard Git interface, so if there is any complex merging issue, an expert user can rely on that to resolve and then switch back to the automatic interface.

## Conclusion

This is just a prototype implementation, I'd be interested in ideas for improvements, please [open an issue on Github](https://github.com/zonca/overleaf_largedoc_main/issues)
