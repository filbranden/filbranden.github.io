---
---

## Projects

These are projects I am an active maintainer for:

* [systemd](https://github.com/systemd/systemd/)
* [numactl](https://github.com/numactl/numactl/)

## Work in Progress

These are some of the recent ideas I have been working on:

### git-rpmbuild

Build an RPM package from a git working tree. I started exploring using
`rpmbuild --build-in-place` to build an RPM package from a developer tree, to
make my life easier when testing systemd changes on Fedora.

I liked this idea so much that at this point I'm looking into polishing my
current shell script into a full-featured tool that supports more projects.

Currently trying to make it work on my other active project with longest build
time: [selinux-policy](https://github.com/fedora-selinux/selinux-policy).
But that one is pretty tricky since the build happens in-tree and the same
tree is build three times for each separate policy. 😞

### Octocopycat

While planning to build this website, I wanted to make it look and feel quite a
bit like GitHub itself, since I like GitHub's interface so much and this
website is mostly about my work at GitHub.

After quite some struggle with Jekyll, I'm finally getting there... 🎉

So I thought I should get everything I learned so far and turn it into a Jekyll
theme, so that all the Jekyll/CSS/JavaScript magic could be maintained on its
own and also others who like this idea can reuse the template as well.

Right now I'm at the stage where I'm
[writing Ruby code](https://github.com/jekyll/github-metadata/pull/151),
but I'm hoping that part of it will be fixed soon...

The website is now finally getting together, so it's finally time for me to
start refactoring the code into a separate theme project.

I already have a name for it... Octocopycat! And I have a preview of it
[here](https://github.com/filbranden/jekyll-theme-octocopycat).
