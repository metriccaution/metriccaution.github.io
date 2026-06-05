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

## Approach

While I was listening along, I was thinking about how I'd code it up, and came up with a way to calculate this fairly simply. I.e. words can be grouped together by how each letter of the word changes from the preceding one.

It's probably easier to give an example, from the episode [`ROT4(bec) = fig`](<https://gchq.github.io/CyberChef/#recipe=ROT13(true,true,false,4)&input=YmVj>).

> The code below handles this a little differently, but explaining it this way round is a little simpler.

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

Then, once we've grouped up, we can examine the contents of each group for cycles.

## Answers

As a caveat, the words list I'm using is whatever the default on Ubuntu is (`/usr/share/dict/words`), I'm not quite sure where it's actually from originally - it seems pretty good, but your answers may vary, and I'm certainly missing some of the words in Matt's dictionary.

I've also excluded any words shorter than 3 letters, as they don't seem to be that interesting.

```bash
cat /usr/share/dict/words | python3 caesar.py
```

- _How many English words are also words after they've been put through a caesar cypher?_
  - 1410 words
- _Is there a pair of words which are caesar cyphers of each other_
  - I've found 56 pairs of words that cycle from one to the other, and back using `ROT13`
    - `ROT13` is significant here as doing it twice takes you back to the original input
  - My favourites here are _green terra_, and Typhoid Mary's lesser known rival, _Ebola Robyn_
  - I don't think there are any cycles longer than 2 words, as the prime factors of the number of letters in the alphabet (`26`) are `2`, and `13`
    - The only cycle lengths possible are divisors of `26`, if you want to get back to where you started (so `1`, `2`, `13`, `26`).
    - This means the next shortest cycle will be 13 items long, and the largest actual group of words with the same signature is 6 words (`aol,bpm,esp,max,rfc,the` being the closest to not gibberish here)

## The Code

```python
import csv
from pathlib import Path
import sys
from typing import Generator, List

# Vector generation


def distance(c1: chr, c2: chr) -> int:
    "Rotation distance from here to there."

    return (ord(c1) - ord(c2)) % 26


def diffs(word: str) -> str:
    "Work out a key to group words on for cypher-equality."

    char_diffs = [distance(c, word[i + 1]) for i, c in enumerate(word[:-1].lower())]
    return ";".join(str(c) for c in char_diffs)


# Text processing


def clean_word(word: str) -> str:
    "Tidy up a word so we don't get messed up on capital letters etc."
    return word.strip().lower()


def words_from_stdin(min_word_length=3) -> Generator[str, None, None]:
    "Read data piped in, normalise it, and filter down to actual words."
    for word in map(clean_word, sys.stdin):
        if not word.isalpha():
            continue

        if len(word) < min_word_length:
            continue

        yield word


# Answering problems


def find_cycles(word_list: List[str]) -> Generator[List[str], None, None]:
    "Spit out any cycles in a word list."

    # While this isn't generally the case, realistically, we're only looking for ROT-13.
    # Otherwise, the next shortest cycle will be 13 items long, and it gets worse from there.
    # Once we ignore words 2 characters or less, the biggest group is 6 items.
    for i, word in enumerate(word_list):
        for other_word in word_list[i + 1 :]:
            if word == other_word:
                continue

            if distance(word[0], other_word[0]) == 13:
                yield [word, other_word]


def answer_question(word_list: List[str], report_file: Path):
    "Actually answer the question."

    unique_words = set()
    for words in word_list:
        for word in words:
            unique_words.add(word)

    with open(report_file, mode="w") as f:
        w = csv.writer(f)
        w.writerow(["How many words?"])
        w.writerow([len(unique_words)])


def output_cycles(word_list: List[str], report_file: Path):
    "Write out lists of words where cyphering loops."

    # Sort by word length
    grouped = sorted(word_list, key=lambda x: -1 * len(x[0]))

    with open(report_file, mode="w") as f:
        w = csv.writer(f)
        for g in grouped:
            for w1, w2 in find_cycles(g):
                w.writerow([w1, w2])


def write_all(word_list: List[str], report_file: Path):
    "Write out all of the groups of cypherable words"

    # Sort by group length
    grouped = sorted(word_list, key=lambda x: -1000 * len(x) + -1 * len(x[0]))

    with open(report_file, mode="w") as f:
        w = csv.writer(f)
        for g in grouped:
            w.writerow(g)


if __name__ == "__main__":
    print("Starting")

    grouped = {}
    for i, word in enumerate(words_from_stdin(min_word_length=3)):
        cypher_id = diffs(word)

        if cypher_id not in grouped:
            grouped[cypher_id] = set()

        grouped[cypher_id].add(word)

    grouped = [sorted(v) for v in grouped.values() if len(v) > 1]
    print(f"Checked {i} words, and found {len(grouped)} clusters of words")

    report_dir = Path()

    print(f"Writing results out to {report_dir.absolute()}")

    answer_question(grouped, Path(report_dir, "answers.csv"))
    write_all(grouped, Path(report_dir, "cypher-groups.csv"))
    output_cycles(grouped, Path(report_dir, "cycles.csv"))

    print("Done")
```
