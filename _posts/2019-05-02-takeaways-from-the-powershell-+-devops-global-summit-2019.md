---
title: 'Takeaways from the PowerShell + DevOps Global Summit 2019'
date: 2019-05-02
permalink: /2019/05/02/takeaways-from-the-powershell-+-devops-global-summit-2019/
categories:
  - PowerShell
  - PowerShell Summit 2019
tags:
  - powershell
  - onramp
  - pshsummit
---
This is my first blog post in a good while and I'm coming back to report on my experience over the past week at the PowerShell + DevOps Global Summit 2019! I first heard about the PowerShell Summit when I started learning PowerShell a couple of years ago, and this year was the first time I've attended. I attended as part of the OnRamp track, which is a mostly separate experience from the general conference.

## OnRamp Track and Scholarship Program

The [OnRamp Track](https://powershell.org/summit/summit-onramp/) is "a distinct ticket type that includes admission to a separate track of content designed for entry-level technology professionals" as described on PowerShell.org's website. Led by Don Jones, Jeff Hicks, and Jason Helmick, the program is geared towards kickstarting one's knowledge of PowerShell over the course of the conference in a small classroom environment. There were about 40 attendees in OnRamp which made for a very personal learning experience with the masters of PowerShell themselves. Two of which are the authors of [Learn Windows PowerShell in a Month of Lunches](https://www.amazon.com/Learn-Windows-PowerShell-Month-Lunches/dp/1617294160), I might add!

Additionally, PowerShell.org offered an application-based [scholarship program](https://powershell.org/summit/summit-onramp/onramp-scholarship/) that provided those awarded with free admission to the OnRamp track, five nights in a hotel, and round-trip airfare. I submitted an application and was honored to be awarded the scholarship.

We were also paired with volunteer mentors from the general conference as a part of the OnRamp "buddy" system. Mine were [Mathias Jessen](https://twitter.com/IISResetMe) and [Daniel Krebs](https://twitter.com/Dan1el42) who were both so friendly and willing to answer any questions that I had. I got a lot of direction about my career from speaking with them.

## Sunday Night Arrival and Monday

I flew in on Sunday night and arrived at the Seattle Marriott Bellevue around 9:30PM. I checked in to my room and noticed other conference attendees in the lobby sitting at the bar. One piece of advice that was consistently given to me in the weeks leading up to the convention was to approach anyone and everyone to talk about their experiences in the world of PowerShell and IT in general. I dropped my belongings in my room, went back to the lobby and immediately was welcomed into the crowd. A few others recognized me from Twitter, which is the first time that's ever happened to me.

After a long night of talking tech with those that I met, I arrived at the convention center the next morning for more over-breakfast discussions and a day of keynotes from notable speakers such as Don Jones and Jeffrey Snover. While there were plenty of PowerShell-focused speakings given (such as a plethora of lightning demos, and a PowerShell financial analysis lesson by [Lee Holmes](https://twitter.com/Lee_Holmes), author of the [Windows PowerShell Cookbook](https://www.amazon.com/Windows-PowerShell-Cookbook-Scripting-Microsofts/dp/1449320686)), I learned a great deal about what it really looks like to work on your career seriously in this industry. Technological agility is important, and the only way any of us will be able to consistently learn and keep up with the industry is to invest in our own education. Don Jones specifically shared great perspective on career development and reinforced my desire to begin investing in myself for the long term.

## Engage Full OnRamp Mode - Tuesday, Wednesday, Thursday

On Tuesday, the general conference began and the OnRamp attendees started their sessions with Don, Jeff, and Jason. The content of the learning track was geared towards following the structure of the first Month of Lunches book (linked above). Obviously the content of the entire book cannot be condensed into three days, so the most fundamental lessons were what we focused on. It was really a privilege to be able to ask the most widely known PowerShell instructors in the world my questions about design decisions in PowerShell. Notably I asked about the history of `$PSItem` versus `$_` and learned that `$PSItem` actually came second after enough people complained about the initially odd appearance of `$_` as a built-in variable.

We spent all three days of the general conference working through portions of the book with the three instructors and a few guests such as [Tim Warner](https://twitter.com/TechTrainerTim). Tim did an awesome presentation on PowerShell "gotchas" which can also be read about [here](https://devops-collective-inc.gitbook.io/the-big-book-of-powershell-gotchas/). It was an eye-opening review session for me with a some great new items learned along the way; I definitely understand PowerShell Core remoting at a more fundamental level and hope to begin experimenting with it on Linux with some Raspberry Pis that I have in my home lab.

On Thursday, the Summit was wrapped up by the [Iron Scripter](https://ironscripter.us/) challenge. I had decided to join the Battle Faction and had a great time working with them for the short duration of the challenge. I did contribute one line of code to fulfill one of the requirements:

```powershell
Limit-EventLog -LogName Application -MaximumSize 4194304 -OverflowAction DoNotOverwrite
```

Yes, it was a small contribution, but at least Jeffrey Snover himself opened up my file during judging...and proceeded to roast me in good spirit!

## Getting Involved in the Community is Important

One of the biggest takeaways from the summit is the fact that being a part of the community around PowerShell is priceless. The strength of the community is great, made all the more evident by the huge number of people in attendance this week and who were on the waitlist.

I am happy that I spent as much time as possible talking to as many different people as possible while I was at the summit. I learned as much during these conversations about PowerShell and technology in general as I have doing actual technical training. Imagine all of the most popular PowerShell educators being available in the same room; that's exactly what it was! For those that didn't attend, an easy way to start having these conversations is to join a local user group. Search around on [MeetUp](https://www.meetup.com/) to see if one exists, or join a virtual one if necessary.

Aside from face-to-face discussions, one question that was consistently asked around the convention was "are you on Twitter?" If you are not on Twitter, now is the time to jump on the wagon. It is an incredible resource for the tech community in general, not just PowerShell. A whole army of intelligent, kind, and helpful people are available around the globe to answer questions about anything. I witnessed many attendees joining Twitter while I was there after they realized how valuable it is.

## Onward and Upward

I'm looking forward to what I will accomplish in the time between this year's summit and the next. Regular blog posts are a good first step, which I hope to establish a rhythm for before long.