---
layout: post
title: "A Problem Squared Episode 103 - Caesar Cypher Cycles"
date: 2025-02-18
---

# A Problem Squared Episode 103 - Caesar Cypher Cycles

[Context](https://podcasters.spotify.com/pod/show/a-problem-squared/episodes/103--Heaps-Splad-and-Pauls-Pad-e2uv2ag).

On the episode, a listener poses the problem:

> How many English words are also words after they've been put through a caesar cypher?
>
> Is there a pair of words which are caesar cyphers of each other, where there's a semantic link between the words? Or common theme to them, or some other interesting relationship?

I thought the episode was great (especially where there's terrible Python code, so relatable ❤️) - though to my ear, the bit about words cyphering into each other sounded more like finding cycles (so something where `Word A` -> `Word B` -> `Word C` -> `Word A`), so I had a go at answering that.

## Answers

As a caveat, the words list I'm using is whatever the default on Ubuntu is (`/usr/share/dict/words`), I'm not quite sure where its actually from originally - it seems pretty good, but your answers may vary, and I'm certainly missing some of the words in Matt's dictionary.

I've also excluded any words shorter than 3 letters, as they don't seem to be that interesting.

```bash
cat /usr/share/dict/words | python3 caesar.py
```

- _How many English words are also words after they've been put through a caesar cypher?_
  - 1410
- _Is there a pair of words which are caesar cyphers of each other_
  - I've found 56 pairs of words that cycle from one to the other, and back using `ROT13`
  - My favourites here are _green terra_, and Typhoid Mary's lesser known rival, _Ebola Robyn_
  - I don't think there are any cycles longer than 2 words, as the prime factors of 26 are 2, and 13 - I'm pretty sure this means the next shortest cycle will be 13 items long, and the largest actual group of words is 6 words (`aol,bpm,esp,max,rfc,the` being the closest to not gibberish here)

## Approach

While I was listening along, I was thinking about how I'd code it up, and came up with a way to calculate this fairly simply. I.e. words can be grouped together by how each letter of the word changes from the preceding one.

Its probably easier to give an example, from the episode [`ROT4(bec) = fig`](<https://gchq.github.io/CyberChef/#recipe=ROT13(true,true,false,4)&input=YmVj>).

- `b` to `e` is +3
- `e` to `c` is +24 (looping around `z` to `a`)

So the "signature" vector for `bec` is `[3, 24]`.

Similarly:

- `f` to `i` is 3
- `i` to `g` is 24

So `fig` is also `[3, 24]`.

However, there's no way to turn cypher `cat` into `bec`:

- `c` to `a` is 24
- `a` to `t` is 19

So `cat` is `[24, 19]`.

I haven't proved it rigorously, but it makes intuitive sense if you think about building the word letter by letter, the two inputs to making a word are the starting letter, and then how to move from letter to letter, until the end of the word.

This approach means that we can group all the words up into groups that can be cyphered into each other with a single pass through the dictionary, by working out this vector for every word.
