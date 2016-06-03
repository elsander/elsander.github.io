---
layout: post
title: Collaborating with your future self using Markdown documents
category: programming
tags: [Markdown, Academia, Data Science]

---

The pipeline for an analysis project can get complicated and
confusing, especially if you're simulating your own data. I often
create pipelines with several different scripts in different
languages, but it's easy to forget a step. But a couple of months ago,
I wrote myself a little Markdown file that looks something like this*:

    0. Format data and subsample to build training set
    **./RCode/BuildTrainingSet.R**
    (for dataset B, use **./RCode/BuildTrainingSetB.R**)

    1. Submit several runs of genetic algorithm to computing cluster
    **./PythonCode/LaunchGA.py**

    2. Check convergence
    **./PythonCode/CheckConvergence.py**
    (if unconverged, return to 1)

    3. Calculate mutual information for best solution
    **./RCode/MutualInformation.R**

    4. Visualize using alluvial diagrams
    **TBD**

This is simply a numbered list of what I need to do to run my entire
analysis from start to finish, complete with paths to the relevant
code, and small notes to myself. I used to trust myself to remember
this pipeline. I mean, I came up with the pipeline, and I commented
the code, and I gave my files informative names. But when I analyze
data, I end up writing lots of small scripts, to explore the structure
of the data and to test assumptions. By the time I need to rerun the
analysis, I've lost track of the files I need to run, and the result
involves opening and reading a bunch of files, mixed with some trial
and error running them.

But why rerun the analysis at all? If you've ever published an
scientific article, you probably know why. Reviewers always have changes
to make and model variants to test. This often involves changing a
few lines, then running the entire analysis over again. But even
outside of academia, if you work with data, this is a problem that
you'll run into. You'll learn something new about your dataset, or
want to try a different prior or a different algorithm. Writing things
down in a structured way has saved me hours during that process.

There's another good reason to write up your process like this. Writing
things down step by step forces you to think through the work that
you've done. Why *did* you write that script? Does this pipeline make
sense? Are there steps that you did by hand that you need to make
note of, or automate? Making the process clear in your head will make
it clearer when you present your work to someone else. It also
provides a convenient outline for the methods section if you're
writing a scientific article.

Why a Markdown file in particular? Well, you could use HTML, or a
simple text file, but I like Markdown for a couple of reasons. First,
it is very simple. If you don't know how to use it, you can
[read an overview](https://daringfireball.net/projects/markdown/syntax)
and learn in about five minutes. It also plays
well with code hosting sites like GitHub and Bitbucket. If you open
the file in github, it will render nicely. But it is also easy to read
in a text editor, because the markup is less intrusive than LaTeX or
HTML.

I only recently started doing these little Markdown write-ups, because
I somehow believed that I would always remember what my code
did. After all, I wrote it. But after a few months have passed, anyone
could have written that code. In essence, I'm collaborating with past
me, except that past me never responds to emails or answers my
perfectly reasonable questions.

Now, rather than dealing with the crappy collaborator that is past
me, I like to think about how future me will think of me as a
collaborator. Future me is also kind of a pain to collaborate with. She expects
me to know in advance what questions she has, and she forgets so much
of what I worked on. But it's not hard to collaborate well with your
future self. You already have a record of your analysis: the
code in your repository. The Markdown notes simply act as a table of
contents. Write one for each of your data pipelines, and your future
self will thank you.

#####\* This example is a simplified version of a project I'm working on right now. The details aren't important, but including this level of detail in your own write-ups will make it more useful to you.
