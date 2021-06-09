---
title: "⚡️ Making Tajweed Bot"
date: 2018-05-15T15:33:42+02:00
tags: ["Bots", "Twitter", "Design", "Writing"]
summary: "Recently I created a Twitter bot which tweets tajweed rules, in the form of visual cards, every eight hours. Just like haiku, I had to work within lots of constraints – space, resources and time. But just like haiku, these constraints helped me to focus and find the essence. This project not only pushed me out of my comfort zone as a programmer, it also challenged me as a writer and a designer."
cover:
  image: "/img/tajweed-bot/tajweedbot-2.png"
---

<br>
I admire haiku poetry. I like how the poets convey such powerful messages within the confines of three lines and 17 syllables.

<br>

> The poet is forced to choose, forced to simplify, forced to find the essence of the message. The constraints are actually a very powerful thing, because constraints force you to be disciplined, to understand that because you have limits, every element in the container must be important, and you can’t just waste words. ~ Zen Habits

<br>

Recently I created a [Twitter bot](https://twitter.com/tajweedbot) which tweets tajweed rules, in the form of visual cards, every eight hours from Sheikha Kareema Czerepinski’s book Ahkam Tajweed alQuran. It was like writing a haiku. I had to work within lots of constraints – space, resources and time. But just like haiku, these constraints helped me to focus and find the essence.

This project not only pushed me out of my comfort zone as a programmer, it also challenged me as a writer and a designer.

## Challenges as a writer

I am using the content from Ahkam Tajwed alQuran (with the express permission of the authoress, of course) which is written in three volumes. But writing for social media is a completely different genre than writing for a book so I have to rephrase most of the content.

My first challenge was space, physical space. Since most of the people would view the cards on a small cell phone screen, I had to make sure that the text was visible at a small size and that meant I could not put too much text on a single card. Sometimes I have to write and rewrite a rule many times before it fits on the card while staying exactly the same in meaning.

I also had to take into account that since the twitter bot will tweet these cards randomly, a single card should give all the information about the categories and sub category of the tajweed science it belongs to. And every single card should tell the reader exactly if there are other rules in the same category and sub category.

So even before I started, I had to plan the layout very carefully.

![tajweedbot-1](/img/tajweed-bot/tajweedbot-1.png#center)

## Challenges as a designer

Since every card contained quite a bit of text along with headings and sub headings, I made sure it was easy on the eyes. After discarding lots of font combinations, I settled for two fonts from Adobe Typekit.

![tajweedbot-1](/img/tajweed-bot/tajweedbot-2.png#center)

I also had to rely heavily on the colors to make my text heavy cards visually appealing and eye catching. I also had to use colors to make some text more subtle and other pop out.

My limitation was that I am not very good with colors. So rather than making sub standard color pallets myself, I took help from my favourite color pallets site, Design Seeds. I found several good color combinations from there and I am using those for the cards.

![tajweedbot-1](/img/tajweed-bot/tajweedbot-3.png#center)

Twitter’s default image compression also became a problem, since it would compress my high-quality image and the resulting image on twitter had blurry text. Sheikh Google came to my help as always and after spending a long time in trial and error I finally found a size and resolution which could uphold the twitter brutality and still give crisp text on image.

![tajweedbot-1](/img/tajweed-bot/tajweedbot-4.png#center)

The biggest set of challenges I faced, however, was as a developer. But that is a [story for some other time](https://blog.naveeraashraf.com/posts/building-twitter-bot/).

My bot is running on Twitter Alhamdulillah, and tweets tajweed rules every eight hours. Apart from the initial development of the bot and the new tajweed cards I am making daily, it is running mostly without any effort on my part. May Allah make it a heavy addition to my weight scale on the day of Qiyamah.

You can follow it at [https://twitter.com/tajweedbot](https://twitter.com/tajweedbot).

The next phase for my tajweed bot is a personal tajweed companion bot, where a user can query for a topic and the bot will send him the resources for that topic. May Allah give me the time and ability to make this resource. Ameen.

Update 2021: I have created a Messenger bot where you can take micro courses to learn the theory of Tajweed. The learning is delivered through the Messeneger bot and contain multi-media resources, including text, images, and videos. You can [see the sample lesson here](https://m.me/101288928290163?ref=w14648943).
