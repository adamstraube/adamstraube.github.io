---
layout: post
title: Hiding capture groups with grep
comments: true
tags: Bash Regex grep
date: 2019-10-22 18:30 +1100
---
Time for some regular expressions :)

Something I tried doing today with grep was searching and printing the text contained between 2 strings within a file. Trying my goto command grep I crafted a command like the following:

```shell
> grep -Po '(<\?=Html::esc\()(.*)(\)\?>)' ./file.php
```

This worked in that it found the text I was loooking for. But it also returned the 2 delimeter strings, I did not want these returned. After some research It appeared that grep does not support masking capture groups (the name given to search groups within regex) out of the box. Before resorting to using `sed` I found out `pcregrep` is able to show specified capture groups with mostly the same syntax as `grep`.

Working from the regex from the above command there are 3 capture groups:
1. `(<\?=Html::esc\()` - Which searches for the start string `(<\?=Html::esc(`
2. `(.*)` - Finds all characters between group 1 and 3
3. `(\)\?>)` - Finds the end string of `)?>`

With `pcregrep` you can select to return a single capture group by using the `-oX` parameter (with X being the number of group).
I was only after printing the second group only so used `-o2`:

```shell
> cat ./file.php | pcregrep -o2 '(<\?=Html::esc\()(.*)(\)\?>)'
```


