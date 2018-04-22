+++
title = "Git diff Minified JS and CSS"
author = ["Jimmy.m.Gong"]
description = """
  Make the `git diff` output be more useful when diffing minified _.js_
  and _.css_ files.
  """
date = 2018-03-19T18:13:00+08:00
images = ["git-diff-minified-js.png"]
tags = ["minified", "javascript", "git", "diff", "css", "magit"]
categories = ["unix"]
draft = false
creator = "Emacs 25.3.1 (Org mode 9.1.9 + ox-hugo)"
[syndication]
  twitter = 975860854573936640
+++

While working on a [little PR for the Hugo doc site theme](https://github.com/gohugoio/gohugoioTheme/pull/84), I learned
that if I needed to make changes to JS/CSS, I had to commit my changes
in both unminified and minified versions.

I have a habit to always look at the diffs at the time of staging and
committing. So it felt very unnatural to commit a minified JS where
the diff would show just **one** changed line with thousands of
characters of minified+uglified JS.

So I started looking for solutions, and found [this post](https://cweiske.de/tagebuch/git-diff-minified-js.htm) by _Christian
Weiske_ where he suggests using [`js-beautify`](https://github.com/beautify-web/js-beautify) to _beautify_ minified
JS diffs.

&nbsp;&nbsp;&nbsp;&nbsp;_And that tool works beautifully!_<br />

-   I later found out that the same tool can also be used to _beautify_
    minified CSS.
-   .. and I installed that tool using `npm` as the `pip3` approach
    failed with /"Collecting js-beautify.. Could not find a version that
    satisfies the requirement js-beautify (from versions: ) No matching
    distribution found for js-beautify"/.

So here's how you can do useful `git diff` for minified JS and CSS.


## <span class="section-num">1</span> Install `js-beautify` using `npm` {#install-js-beautify-using-npm}

I see myself using `js-beautify` in many other projects too. So I
installed it globally.

```text
npm install --global js-beautify
```


## <span class="section-num">2</span> Configure `git` to use that tool {#configure-git-to-use-that-tool}

Add below to your `~/.gitconfig`:

1.  Use `js-beautify` to first beautify the minified JS for the `minjs`
    _diff configuration_, and then do a diff of those beautified files.
2.  Enable caching of those beautified files to speed up the diff, so
    that _re-beautification_ of unmodified minified files can be
    skipped.
3.  Similarly for minified CSS, use `js-beautify --css` to first
    beautify the minified CSS for the `mincss` _diff configuration_.

```ini
[diff "minjs"]
    textconv = js-beautify
    cachetextconv = true
[diff "mincss"]
    textconv = js-beautify --css
    cachetextconv = true
```


## <span class="section-num">3</span> Add/update `.gitattributes` file to the project repo {#add-update-dot-gitattributes-file-to-the-project-repo}

Now, in your project repo's `.gitattributes` file, you need to
associate files with the _diff configurations_ set above.

Below will use the `minjs` configuration for _\*.min.js_ and
_\*.bundle.js_ files, and `mincss` configuration for _\*.min.css_ and
_main.css_.

```ini
*.min.js diff=minjs
*.bundle.js diff=minjs
*.min.css diff=mincss
main.css diff=mincss
```


## Beautiful Result {#beautiful-result}

<a id="org36d0e14"></a>

{{< figure src="images/git-diff-minified-js-and-css/git-diff-minified-js.png" caption="Figure 1: `git diff` of minified JS as seen in _Magit_" >}}

Isn't that better than how GitHub shows the _exact same commit
diff_? :sunglasses:

<a id="orgb0aa136"></a>

{{< figure src="images/git-diff-minified-js-and-css/github-diff-minified-js.png" caption="Figure 2: Same commit `diff` shown on _GitHub_" >}}
