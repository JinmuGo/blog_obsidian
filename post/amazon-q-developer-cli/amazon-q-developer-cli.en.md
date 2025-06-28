---
title: Amazon Q Developer CLI Quick Start Guide
date: 2025-06-04T13:25:20.437Z
draft: false
layout: PostBannerX
summary:
tags:
  - AmazonQCLI
  - AWS
lastmod: 2025-06-05T13:34:57+09:00
category: aws
---

AWSKRUG alerted me to an event on creating a simple game with Amazon Q Developer, so I thought I'd join in.
![[image-67.webp]]

It's easy to participate.

You create a game with the Amazon Q CLI, write a blog or make a video about how you did it, and submit the link to the form.

> [!info] How to participate
>
> **STEP 1** : Sign up to an AWS Builder ID and claim your unique community.aws username from [this link.](https://community.aws/builderid?trk=b085178b-f0cb-447b-b32d-bd0641720467&sc_channel=el) If you need help or network with other fellow builders, join the discord [server from this link](https://discord.gg/KNC9JQAfKT).
>
> **STEP 2** : [Install Amazon Q CLI](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-installing.html) in your machine. Here are two guides from Ricardo Sueiras on how you can install in [Linux](https://community.aws/content/2ulGwNwLFj5grS8hXJBMCN78Qwl/the-essential-guide-to-installing-amazon-q-developer-cli-on-linux?trk=6f6cb092-f1ba-456b-8644-73ed7ccbd567&sc_channel=el_) and in [Windows](https://community.aws/content/2v5PptEEYT2y0lRmZbFQtECA66M/the-essential-guide-to-installing-amazon-q-developer-cli-on-windows?trk=e07eca93-fa2f-4351-b567-f293b83eb635&sc_channel=el_). Install [PyGame](https://www.pygame.org/wiki/GettingStarted) or some other gaming library on your laptop.
>
> **STEP 3** :Start a chat session with Amazon Q CLI and build a game with just prompts in the chat. Try to be innovative as possible with your game so you explore the possibilities of Amazon Q CLI.
>
> **STEP 4** : Write a blog about what you've built or create a video about it. Make a public post in a social media of your liking with the hashtag #AmazonQCLI. It can be in your local language, which we encourage. Optionally you can also host your code in a [github](https://github.com/) repository.
>
> **STEP 5** : [Fill this T shirt redemption form](https://pulse.aws/survey/ZO9G4AEL).

Since we need to create a game, we first need to install the Amazon Q Develop CLI.
It looks like it can only be installed on macOS and Linux-based OSes by default. Windows needs to use WSL. You can find detailed installation instructions at the link below.

```embed
title: "Installing Amazon Q for command line - Amazon Q Developer"
image: "https://docs.aws.amazon.com/assets/r/images/aws_logo_light.svg"
description: "Learn how to install Amazon Q Developer for command line on different operating systems including macOS, Linux, and Windows Subsystem for Linux (WSL)."
url: "https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-installing.html"
```

After installing and typing `q` in the terminal, you should see something like this!
![[image-69.webp]]

## Time To Create

After completing the installation, we need to create the game, and since I don't use Python very often and don't know anything about Pygame, I really let `q` do all the coding logic and image generation.

I had a planning document from a previous school class where I had designed a game, and I gave it to GPT, who recreated it in English markdown and gave it to AWS Q.

It was a game plan called Elysian-Arcana, and I was wondering how to give it to amazon q, so I looked at the `/help` function of amazon q, and there was a command called `context`, so I was able to provide context.

![[image-70.webp]]

![[image-68.webp]]

So I put my game plan document in the context of Q and asked him to create a mini-game based on this markdown document.

He created a roguelike, turn-based card game.

![[image-71.webp]]

They did a good job, but it's still really cool to see an AI make a turn-based game. It still lacks a lot of details and gameplay, but it's still cool to see something that was made with just vibe coding.

![[image-72.webp]]
To make it a bit more gameable, it lets you pick a card after you clear a stage.

After five stages, my characters have leveled up, their mana barrels have increased, their health has increased, and they now show the intent of their opponent's attacks. You can also see that I've taken care of the game's convenience.

![[image-73.webp]]

It's exciting to be able to make a game like this, but it's also a little disappointing.
I asked it to keep creating and linking pictures of the opponent's monsters about 3 times, but it didn't keep doing it because I didn't write the prompt well enough, and the images of all the cards were the same.

But it's amazing how far we've come. I was really surprised that they got it to this point, because I kept asking for things like feature and screen definitions without knowing anything about the implementation details.

It's free to use, so if you're interested, you might want to give it a try. I've attached the source code below.

```embed
title: "GitHub - JinmuGo/elysian_arcana_aws_q: build game with aws q"
image: "https://opengraph.githubassets.com/d0cf93f2488e20f0c8ccd3df91300d029b5f4bfc5f2e02ab63a5acd186bd0c03/JinmuGo/elysian_arcana_aws_q"
description: "build game with aws q. Contribute to JinmuGo/elysian_arcana_aws_q development by creating an account on GitHub."
url: "https://github.com/JinmuGo/elysian_arcana_aws_q"
```
