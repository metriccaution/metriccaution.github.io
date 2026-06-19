---
layout: post
title: "Rules Cheat-Sheets"
date: 2026-06-17
---

# Rules Cheat-Sheets

A casual inspection of my Github will make it look like I do little else except play D&D, while I do play most weeks, it's over-represented in my coding because it's a domain I understand fairly well, and one with a lot of space for useful helpers - today's blog is about an RPG tool I've built recently.

I'm a long way from the first to write a rules cheat sheet for games I play, but I do think that [my version is a useful spin on the idea](https://metriccaution.github.io/rules-cheatsheets/) - I've written up a bunch of snippets that either cover a tricky mechanic that I know will be a problem, or answer a specific question during play, each of which is tied back to where to look up the rule itself in the original source.

This started as a React SPA in my [mono-repo of mini-projects,](https://metriccaution.github.io/web-snippets/) but I decided to take the time to break it out into its own thing when I wanted to add a couple of other game systems to it, and I've used it as an excuse to get a bit more comfortable with Github Actions.

## Design

The main design principle here is that I want writing rules snippets to be super simple, and I don't want to have to think about the framework itself again, unless I need to crack it open to make a site-wide change. The content itself is a few directories of markdown files, with a frontmatter schema, which then get rolled into a basic Metalsmith site generator, by getting converted into structured data, and then fed into the templates.

At that point, writing up a [Github action](https://github.com/metriccaution/rules-cheatsheets/blob/main/.github/workflows/deploy.yml) to deploy any changes to `main` was pretty simple, so now pretty much any remaining effort with the tool should simply be content authoring.

## AI Development

This was one of the first projects I used [Claude Code](https://code.claude.com/docs/en/overview) on, and it was both interesting, and a decent lesson in some things it's very good at, and some things it's bad at.

### Problems

To get the negative side out of the way first:

- I had written up a little framework to get Claude code to validate the info with, if I gave it access to rulebook PDFs, this was pretty hit or miss, honestly, it did catch a couple of issues with games I'm not so familiar with, _but_ it also got quite a lot wrong, persistently so too.
- Getting it to fact-check the ~80 rule hints that I'd already written up ended up chewing through my token limit for the day, so it was pretty expensive honestly!

### Highlights

On the plus side, it genuinely did take care of some things I didn't really want to spend my spare time doing:

- While there's not much code present to create the site, I had it mostly written before I started using Claude Code, and it understood the structure and flow of data without any prompting.
- Claude's UI design skills are much better than mine, and while the resulting site is hardly a work of art, I'd been dreading turning my unstyled code into something I could stand to look at, and it just took care of that.
- While it wasn't great at doing the extra fact checking, it _was_ pretty handy at adding nice markdown styling to my existing rules backlog, and editing for consistency of tone, even if I did spend quite a while fighting it on this too, when it broke the actual rules clarification more than once!

## Looking Forwards

As far as I'm concerned, the code here is pretty much done, but I do intend on keeping on churning out new rules hints for the foreseeable future. As an exercise in Github Actions, this ended up being pretty insubstantial, so I'll keep an eye out for more things to do there. Finally, I am, unsurprisingly, going to continue to experimenting with Claude.
