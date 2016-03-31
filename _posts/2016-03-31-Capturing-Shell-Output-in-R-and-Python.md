---
layout: post
title: Capturing Shell Output in R and Python

---

Sometimes I spend significant time in R or Python trying to do
something which is trivial is bash. This is especially useful when I'm
working with very large files that will take a long time to read
in. Why read in an entire file to get the last line, when I could just
use `tail -n 1`? Or if I want the line count, why read it in when `wc
-l` will get the job done faster?

It turns out that it's not too complicated to capture shell output in
R or Python. Here's how I do it.

## Python

If you use Python 3, capturing shell output is pretty simple (if
you're still on Python 2, the tides are turning! It's time to make the
change!). You can use the `subprocess` module to get the output in
bytes, then decode and parse it.

    import subprocess

    ## Get the last line of the file 'fname'
    last_line = subprocess.check_output("tail -n 1 " + fname, shell = True)
    ## convert to string and parse
    ## 'UTF-8' is a common encoding, but you may need to use something else
    last_line = last_line.decode('UTF-8').strip()

## R

R makes this process easy too. You may have used `system()` before to
submit shell commands. It turns out that if you set the argument
`intern = TRUE`, you'll get the output as a character vector-- you
don't even have to deal with encoding! The output may take some
parsing, but the `stringr` package is good for that.

    require(stringr)
    ## Get the last line of the file 'fname'
    lastLine = system(stringr::str_c("tail -n 1 ", fname), intern = TRUE)
    ## strip leading/trailing whitespace
    lastLine = stringr::str_trim(lastLine)

This has saved me from reinventing the wheel many times since I
learned it. Hopefully it helps you too!
