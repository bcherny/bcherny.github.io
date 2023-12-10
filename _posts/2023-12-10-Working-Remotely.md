---
layout: post
title: 'Learning to work (very) remotely'
date: 2023-12-10
categories: Tech
---

For the last year and a half, I have been working at Instagram in Japan. It's a bit of an unusual setup: I live and work remotely from Nara, Japan, in a timezone that few other Meta engineers work in (most of Instagram and Meta is spread across the US and Europe).

I am fortunate to have the opportunity to have a setup like this. How I ended up here is a long story, which I will save for another post. For this post, I'd like to talk about what I have learned about effectively contributing to an engineering organization when you are isolated from your coworkers physically, temporally, and organizationally.

## Learning to work again

Before moving to Japan, I served as the tech lead for Facebook Groups. That meant organizing meetings, building slide decks and spreadsheets, writing scoping documents, respinding quickly to chat messages and emails to keep teams unblocked... You know what I wasn't doing a lot of? Coding.

After moving to Japan, with so little timezone overlap (9am my time is 4pm in San Francisco, 7pm in New York) I suddenly found myself with nearly the whole day free. My coworkers went offline a couple hours after my day started, until London woke up for the last hour of my working day. That meant nearly zero meetings and few synchronous chats.

Due to the incompatible timezones, I started attending group meetings less and less. At the same time, my coworkers naturally took me out of the critical paths of important projects. If anything was time-sensitive, I was no longer a good person to take care of it. I wouldn't be included in chats and email threads anymore. My directors wouldn't ask me to put together quick planning docs after exec reviews, nor would I be included in those reviews in the first place. This stung, at first -- I was used to being included in meetings, and to learn about important information before others did. These are the things that made me feel important! I didn't want to give them up. But I had no choice.

At first I struggled to figure out how to spend my new-found free time, and how to feel good about my contributions to Instagram. I experimented with a few ideas: hacking on new product ideas with the other engineers in Japan, mentoring engineers in our Singapore office (larger than the one in Tokyo), fixing tech debt that others left behind.

It turned out that most of this work wasn't easy to do. It was hard to effectively work on product while disconnected from the bulk of our US-based product organization; and it was hard to be a good mentor while having little context on what was going on in the Singapore office.

However, I did find that writing code again was rewarding. In my first two weeks in Japan, I landed more code than I had in the previous year in Menlo Park. I'm an engineer, and this made me happy -- I missed coding! While hacking on a few product ideas ([Instagram Maps](https://techcrunch.com/2022/07/19/instagram-new-searchable-map-experience/) and [QR Codes](https://mashable.com/article/qr-code-instagram) and fixing tech debt around our Python codebase, I kept a running list of opportunities I found along the way: abstractions that were repeatedly slowing down engineers and causing reliability issues, small improvements we could make to the type safety of our libraries, new infrastructure that we could build and ways we could make existing infrastructure better.

## Leaning in

I started to shift the kinds of things I worked on a bit, to focus more on simplifying our codebase, improving our infrastructure, and making our engineers more productive. I had done infra and DevX work in the past (some of it [open source](https://github.com/bcherny), but it had always been limited to 20% time projects. It was never my main gig. I considered myself a product engineer, but found myself sliding deeper into the world of infrastructure and DevX.

This shift was not something I did intentionally. Because my time no longer got filled up by meetings and obligations that other people put on my plate, coding and fixing the issues I found while coding gradually took up more and more of my time, until six months into being in Japan I realized that it had become my main work. I felt happy about the work I was doing, and satisfied writing code again. I started to gain a reputation as an engineer that knows the codebase and has good technical opinions, despite still having been at Instagram a relatively short time, due to how much code I was churning out and the pain points I was fixing for other engineers along the way. (By this point, I was in the top 1% of engineers at Instagram by code output.)

Reflecting back on this a year later, I realize I was doing the work that other engineers should have been doing, but often couldn't due to the realities of being in an organization that demands more of your time and attention as you grow more senior. Famously, at Meta even senior engineers are expected to code (we don't have many architect types), but their level of code output is often not the same as more junior engineers. I had essentially turned myself into an intern, coding 80% of the day. This was a powerful change, which let me identify and execute on opportunities that others simply couldn't.

## Shifting how I operate

As I did this, I had to get comfortable leading from behind. I spun out a number of projects, recruiting engineers in the US and EMEA to lead and take over as soon as an effort developed sigificant communciation or coordination components. This meant delegating, and letting these emerging leaders be the public faces of projects that I might have previously spent longer incubating myself. I felt anxious about getting credit for this work, but quickly found that my worries were misplaced, and that my reputation continued to grow, and engineers and managers around the organization were not only aware of my work, but were appreciative of the opportunities I created for others and the impact this was having on Instagram. By becoming a more effective delegator, I was able to grow more engineers more quickly than I had in the past. It was gratifying, and it let me deliver results without having to participate in most meetings and synchronous communications myself.

Around this time, I shifted my communication style from trying to make my previous approach work in a new environment, to using my odd timezone to my advantage:

1. Instead of rapid back-and-forth over chat, I caught up on accumulated messages in the mornings and spent the first hour of every day responding in long form. I had to anticipate what the person at the other end of the chat was really asking, how they would respond, and what else they would need to know. By taking time to word a better response, I avoided time-consuming back-and-forth for both of us.
2. Instead of meetings, I used Workplace posts (Meta's internal version of Facebook Group posts) to share thoughts, start discussions, and make announcements. This had the benefit of serving as a system of record and a form of documentation, while at the same time being more inclusive of people in different timezones and those who would not have been invited to or would not have felt comfortable speeking up at a meeting.
3. I focused my 1-2 hours of available meeting time each day on the conversations that absolutely had to be synchronous. This mean spacing weekly 1-1s out to biweekly or monthly, and turning down meetings more often.
4. I began flying to the US every quarter or so for in-person time. No matter how good I felt about my newfound ability to work asynchronously, there is simply no alternative to seeing your coworkers in person, catching up about their families and lives without the ticking clock of a virtual meeting, and grabbing beer after-hours to kvetch and ideate and get to know each other as friends.

I also had to adjust my schedule to better manage my own health. When I worked in California, I would drive from San Francisco to get to Menlo Park around 10am, then head home around 6 or 7pm. Now, since my time overlap with the US was in the morning, I had to shift my meeting times earlier, often to 8 or 9am. At the same time, it is important for my mental health to have a morning routine: I like to run, make coffee, and study in the morning; to avoid compromising on this, I shifted my schedule up, waking up at 6:30am to have time for these must-haves.

## Closing

Overall, the move turned out well for me: I have found how to work effectively from a very different timezone, and am happy with the results. I don't think this setup would work well for everyone, and I feel lucky that I was able to iterate to a working style that works well for me. I am similarly grateful that my organization appreciates the work I do, and is accommodating of the way I do it. I hope my experience resonates with others, and am eager to learn about others' experiences in similar setups.
