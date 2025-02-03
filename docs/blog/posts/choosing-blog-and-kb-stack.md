---
title: Choosing a stack for a dev blog
description: How I selected the stack for this dev block.
date:
  created: 2025-01-15
categories:
  - blog
draft: false
comments: false
---

# Choosing a stack for a dev blog nowadays

This year's resolution was to try to get more public - internet - exposure. The main reasons:

- Connect with people outside who share my interests.
- Consume less, create more: For years, I've been learning a lot from the internet. Now it's time to share.
- Trust your own “Curiosity Barometer.” If something sparks your interest in a unique way, it’s likely to resonate with others too. Write about it, share it, and see what unfolds.


Now, to align with the title: What should I use to publish it? When there are so many options available, I tend to simplify the decision to avoid getting trapped in **analysis paralysis**, so:

<!-- more -->

## Requirements

1. Being able to write in Markdown, as I'm comfortable with the syntax.
2. Should include blog-like content and flat pages.
3. Being able to create a kind of knowledge base or second brain.
4. Being able to customize it.
5. I should ship it in a morning or two.

### First Decision: Server-Side or Static Site?

This was easy with requirement #5. Although I'm a backend developer, I know if I go down that route, I won't ship it fast as I have so many ideas to build. So, **static** it is.

After searching and prompting for a while, three options seemed good:

#### [MkDocs (Material)](https://squidfunk.github.io/mkdocs-material/)

1. Pure Markdown, and I already built some project documentation with it.
2. Ok.
3. No specific way to create a KB.
4. I should be able to. Just Python and some JS.
5. Probably yes.

#### [Docusaurus](https://docusaurus.io/)

1. Markdown and some MDX (React-based).
2. Ok.
3. No specific way to create a KB.
4. I should be able to, but I prefer Python.
5. Likely.

#### [Quartz](https://quartz.jzhao.xyz/)

1. Markdown.
2. Ok, although it doesn't seem easy to customize the layout.
3. Very specific to KB—great!
4. JS, components…
5. Not likely.

### Decision

So, I went with MkDocs and the Material theme, prioritizing content over complexity. Let's ship!