---
title: "A Beginner’s Guide to System Design"
source: "https://medium.com/@sentalkssane/a-beginners-guide-to-system-design-76d64689788b"
author:
  - "[[Aritra Sen]]"
published: 2024-09-18
created: 2026-04-20
description: "A Must-Read Guide: Insights and Tips from my experiences to ace the System Design Interview (as a Mid-level Software Engineer)"
tags:
  - "clippings"
---
## Must-Read Guide with insights and tips from my experiences to ace the System Design Interview in FAANG and other big tech companies

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*W7FPF0ywvPDMSEXr)

Photo by Nick Morrison on Unsplash

“ *It’s time to switch* ” — It was December of last year when the thought first occurred to me. I had been working at Google for nearly 3.5 years and it was high time for me to try out new opportunities and learn from them. **However, a big hurdle stood in my way.**

## I HAD NO EXPERIENCE WITH SYSTEM DESIGN INTERVIEWS!!!

At the time, I was attempting a lateral career switch. I had been looking into mid-level software engineering roles (*“Google L4 equivalent”*) and a minimum of one mandatory System Design interview was a step every company had in their interview process. There was no way around it and I had to prepare. I dove head-first into it only to realize, I was doomed.

> System design seemed like a never-ending rabbit hole of design and architectural choices and there was no right answer — *“or so they say”*.

The problem wasn’t that resources weren’t available — there was just NO STRUCTURE. Information seemed to be scattered everywhere and time seemed to be limited. Additionally, every YouTube video of “ *mock system design interviews* ” I encountered, sounded more like a well-researched script on the subject, rather than a demonstration of approaching new requirements in a 45–60 minute interview span setting an unrealistic expectation. The start had been completely demotivating and anxiety-inducing. However, I persevered, researched, and prepared more, learning from Reddit, Blind, a few GitHub repositories, other medium articles, and YouTube. The struggle eventually ended with me managing to ace my interviews at *DoorDash* and *Meta*, choosing to join Reality Labs @ Meta, and deciding to write this article.

> **✨** I hope this article helps candidates dreading their first System Design interviews find a better starting point than what I did. For others, I hope you find the “missing piece” to your own preparation strategy. While my process wasn’t perfect, it worked for me (call it beginners luck) and I hope it adds some value for you too. I have tried to recommend only **free resources** like [System Design Handbook](https://www.systemdesignhandbook.com/guides/system-design-interview-questions/) so that everyone can benefit from this. ✨

That being said, here is how I recommend preparing for your next system design interview.

## Step 1: Back to the Fundamentals

The sad truth is that professional work experience as a software engineer does very little when it comes to system design interviews. ==**A typical system design interview revolves around the idea of building a scalable distributed backend solution to address a given problem statement.**== The problem statement may cover a vast number of domains, something that our on-the-job experience most often doesn’t cover. For example, my work experience in the past includes developing web applications and automation solutions for internal systems and working on the Android OS (neither of which screams *highly scalable distributed systems*).

At times like these, the student mindset of understanding the fundamentals and building from there has helped me the most. Here’s what I would cover.

- **Understand common principles.** My absolute favorite starting point would be [**this Harvard lecture**](https://www.youtube.com/watch?v=-W9F__D3oY4). It gives a good layman's explanation of many scaling concepts you would eventually deep-dive into later. Following it up with this series of [**posts**](https://web.archive.org/web/20221030091841/http://www.lecloud.net/tagged/scalability/chrono) should give you a clear picture of what scaling and building distributed systems involve. Gaurav Sen’s [**system design playlist**](https://www.youtube.com/playlist?list=PLMCXHnjXnTnvo6alSjVkgxV-VH6EPyvoX) is another great place to understand fundamental concepts. **A non-exhaustive list of topics to cover would include —** *\* Load Balancing  
	\* Consistent hashing  
	\* Scaling  
	\* CAP theorem  
	\* CDN’s  
	\* Caching  
	\* Failovers  
	\* Replication  
	\* APIs (REST/ gRPC )  
	\* Polling, Long Polling, and Websockets*
- **Understand your trade-offs.** Reading through the [**system design primer**](https://github.com/donnemartin/system-design-primer?tab=readme-ov-file#system-design-topics-start-here) would cover a lot of trade-off choices you would be looking into during your system design interviews, esp. *the CAP theorem*. The reading material is fairly comprehensive, with linked sources to dive further into the topic. Designing a system architecture is a balancing act of sorts, and having a clear understanding of your trade-offs upfront would help you make the right design choices for your *requirements* during your interview.
- **Understand databases.** Most of the choices you make during your interview would revolve around data. The 3 big questions are — how do you **represent, transport, and store** your data? Understanding databases helps you answer at least 2 of these questions. Different databases have different perks and compromises that are important for you to realize. Also understanding your data replication/backup strategy, read v/s write performances (if your system is read-heavy or write-heavy), do you need to *shard* your data, indexing, etc would be necessary parameters you would need to consider and know to make reasonable database design choices.

> 📝 **Note:** The expectation isn’t to pick a database offering (e.g. you don’t need to choose between MySQL or PostgreSQL). Rather, you must have a clear idea and make choices that consider the gains you would appreciate from your DB and compromises with it the design architecture is okay with.

- **Queues are OP**. Scaling your architecture is hard. One of the bigger challenges you face is handling the frequency of requests at a time. Integrating queues allows you to handle requests asynchronously. They have many other uses as well. Understanding the WHY and HOW you should use a queue is **equally important**. Queues come in various forms as well — **Message Queue and Publish-Subscribe Models** being the more common ones**.** Using the right queue in your design (*with the right justifications*) can be a big strength in your interviews.

The above list is not exhaustive by any means but should be a good enough starting point. The next steps would cover how to approach your interviews, but without having sound fundamental knowledge it would be very difficult to make the most of the next steps.

> ✍ **Follow-up:** Interviewing.io also wrote a very well-consolidated [**article**](https://interviewing.io/guides/system-design-interview/part-two#12-fundamental-technical-system-design-concepts) on the 12 fundamental system design concepts that might also prove to be useful.

## Step 2: Understand the template

Time is not on your side when it comes to system design interviews. The expectation to understand a problem statement and translate it into a scalable design architecture with clearly laid out design choices meeting the interviewer’s standards in a 45 to 60 interview period *is anything but EASY***.** This is where the role of the template comes in.

> A template gives you a structure on how to approach design interviews in a structured manner, ensuring you don’t miss out on key information, make the right design choices, and successfully cover everything in a limited time.

There are multiple variations of this all over the internet (*they are mostly the same though*) but here is what I followed:

### Requirements

> The first step to any system design interview problem is to scope it. Requirements can be scope into 2 categories — functional and non-functional.

A problem statement, say “ *Design Instagram* ”, is very vague and has a very broad scope. Designing every feature that Instagram does is next to impossible in the short span of the interview. However, scoping it down to designing a photo-sharing app makes the task much more doable. Talking this through with your interviewer is important — especially because it helps you realize what their priorities are. Do they care about how user login and authentication is done, the followers/following feature, the comments section, reels, etc? It is easy to get sidetracked and knowing which part the interviewers care about primarily makes a lot of difference. An interviewer might be more interested in the photo-sharing aspect, while another might be more interested in features like comments or followers/friends. These are known as **functional requirements** and identifying them is the first piece of the puzzle.

The other aspect is **non-functional requirements**. These are criteria that define how a system should behave, rather than what it does. Is the system read-heavy or write-heavy? Are there any memory or latency constraints? Is strong consistency required or is eventual consistency acceptable? What are the security and privacy concerns (like authentication) we need to keep in mind? Is the system expected to be highly available? These are some examples of non-functional requirements and help determine what kind of technological trade-offs you would need to make in your architecture and database design. (*remember you read about them during step 1*).

### Back-of-the-envelope estimations

This is a very natural follow-up from the non-functional requirements. Now that you have understood your requirements — it is essential to quantify them. Some metrics to highlight here would be —

- **General daily usage and scale estimation**: This is usually measured in terms of **Daily active users (DAU)**. Other things to consider are the frequency of read/write operation per day. This also gives us a good understanding of if our system is **read-heavy** or **write-heavy**. Eg: For Instagram, estimations of its DAU, posts per day, posts seen per user every day, etc can be useful data points to capture.
- **Memory requirements**: This represents how much memory would be required by your system. This is highly affected by the **average number of writes and size of data written per write** made to your database. Having an estimate of at least one year’s worth of data storage requirements is a good idea to understand storage requirements.

> 📝 **Note:** For systems active for a small time, calculate the data stored for the active period should be sufficient. For example, if you are making an app for ticket booking for a charity event being held 2 months from today, most of your data would be coming for 2 months (and not 1 year).

- **Network Usage:** What is the average number of requests per second? What is the consumed bandwidth of our service per request? Knowing the peak requests per second gives us an estimation of the maximum load our system should be able to handle. It helps justify design choices involving databases (schema choices/replications/sharding etc), the usage of queues, and CDNs/cache in our design.

> ✍ **Follow-up:** A good read to understand back-of-the-envelope estimation in more depth is linked [**here**](https://medium.com/@saurabh.engg.it/software-system-design-back-of-envelope-calculations-8f2d9d0f4edd).

### API design and data representation

Before moving into the internal design and implementation, deciding how you represent your data as well as external-facing APIs would be the next step.

> This illustrates how you expose the functional requirements agreed on earlier from your system and how they should be consumed by your client.

It is a good place to introduce ideas such as pagination (for read-heavy APIs) or rate limits (for security) to your API. Choosing your API protocol (REST, gRPC, or graphQL) is another important choice made during this situation. Make sure to be consistent with how APIs in your protocol of choice are represented when mentioning the API design to your interviewer.

### High-level design

> This step covers presenting an outline of the overall system architecture, the different internal components of your system, and how they interact to serve each type of API.

Make sure to introduce all essential components to your design such as load balancers, different services, databases, queues, and caches to your system as needed. It is often difficult to avoid going too deep into your design choices here — but I would strongly recommend keeping this section concise and limited to painting an overall picture of how your system and API request flows. Only once you have defined your high-level design, start deep diving into your design choices (*covered in the next two steps*).

> **⚠️Warning:** System design interviews are a delicate time balancing act, and it is very easy to get stuck explaining one section of your design in deep details before you have painted a clear outline of how your design operates and meets the requirements.

### Database design

Database design is one of the most important highlights of your interview. It is most often the first thing you should deep dive into once you have gone through giving an overview of your design. Your schema and other design choices should be able to serve your initial back-of-the-envelope estimations, requirements, and data representation you have agreed upon earlier with the interviewer.

## Get Aritra Sen’s stories in your inbox

Join Medium for free to get updates from this writer.

**Some important things to address here are —**

- Database Schema
- Data replication and sharding strategy (if any)
- Object storage (if any images, audio, video, blobs, etc need to be stored)

> 📝 **Note:** This is a high-level design and no SQL queries, UML diagrams, etc are expected at this stage. Avoid wasting time by writing schemas in any standardized formats as such.

Choosing if you want a SQL or NoSQL database solution is important as well. Some factors you should consider are —

- Is your data structured/relational or unstructured/non-relational?
- Is your system required to be strongly consistent or does it need to be highly available?
- Is your system read-heavy or write-heavy?

For example, if you were designing a payment app handling outgoing and incoming transactions, being strongly consistent might be a requirement, and hence relational databases that follow ACID properties might be the correct type of databases to use here and base your design off.

> ✍ **Follow-up:** [**Here**](https://interviewing.io/guides/system-design-interview/part-three#3-1-1-relational-vs-non-relational) is a good read to understand choosing Relational vs/Non-relation databases during a system design interview.

### Detailed design choices

This last section covers justifying other design choices you have made in your overall architecture and going into details where required (similar to what you might have done earlier with databases). Authentication, scaling (load balancing), caching, serving data (eg: do you need CDNs), and data retention strategy are other aspects of your design that you should consider covering here. Also, no design is perfect. You should be able to talk through what trade-offs you took and why, as well as how you address concerns your interviewer presents. Also, this is where the interviewer might ask a lot of questions (*they would ask clarifying questions throughout the process usually*) regarding any unaddressed concerns they might have over your design or give follow-up/stretch requirements to add to your design (*if time permits which is rare*).

> **⚠️Warning:** Being able to defend your design choices is **NECESSARY**. You might have a *“* correct *”* design but if you can’t reason about it — its not really any good.

## Step 3: Practice the Obvious

For better or worse, system design interview problems are reasonably repetitive (*with small twists*). Being well-versed in solutions to common system design problems gives you a major competitive advantage. A lot of concepts you learn in one design problem are often transferable to other designs — giving you more confidence in your ability to ace your interviews.

If you are looking for good resources you can try these out.

- [**System Design Fight Club**](https://www.youtube.com/@SDFC) **(SDFC)** is my **TOP** recommendation from YouTube. They have a large variety of system design problems discussed over live Zoom streams that make it a great place to learn. The only downside is the videos are very long and raw/conversational, which might make it a little difficult to follow through if you are just starting with system design.
- [**Exponent**](https://www.youtube.com/@tryexponent) is a good YouTube channel for starters. They have short comprehensive solutions to a wide variety of problems. The videos are short and precise which makes them less time-consuming in general.
- [**“Jordan has no life**](https://www.youtube.com/watch?v=ugVwhsWslAc&list=PLjTveVh7FakKjb4UYzUazqBNNF-WGurXp) **”** is another excellent content creator in this space — one I followed a lot during my journey. I found the explanations much more detailed here and helped expand on what I had learned from watching bits of solutions on Exponent.
- For a bit more in-depth learning — [**Gaurav Sen (GKCS)**](https://www.youtube.com/@gkcs) would be an easy recommendation. I followed a lot of his content to learn concepts around Load balancing, and consistent hashing, and his [**system design playlist**](https://www.youtube.com/playlist?list=PLMCXHnjXnTnvo6alSjVkgxV-VH6EPyvoX) was great for understanding fundamental concepts.
- I read through the [**System Design Interview Vol. I**](https://www.amazon.com/System-Design-Interview-insiders-Second/dp/B08CMF2CQF) and [**Vol. II**](https://www.amazon.com/System-Design-Interview-Insiders-Guide/dp/1736049119) by Alex Xu. It was a really useful starting point — and combining the solutions in the book with content from YouTube was an easy recipe for understanding everything.

This step is fairly important since this is the first time we are solving questions using the fundamentals we have learned. Trying to fit your understanding of the problems into solutions that fit into the template we learned earlier would be a big step in ensuring success in your interviews.

Also try to look at multiple solutions for the same problem statement, since different sources would do it a bit differently helping you learn a lot more from one problem and understanding alternatives available.

> 💡 **Try this out:** Learn from your first five designs first. Next, start trying to implement the sixth problem based on what you have learned so far. Finally, look at the solution and improve your design accordingly. Continue doing this with the 7th, 8th, 9th question and so on. It will broaden your understanding of both — what you already practiced and the problem you are attempting currently.

## Step 4: Give Mock Interviews

There is always a difference between understanding system design solutions and being able to answer in an interview. Mocks prepare you for this. Regarding where you could get a mock interview setup, here are a few options I can recommend.

- [**Interviewing.io**](https://interviewing.io/): This is a great place for signing up for anonymous system design interviews.
- [**Topmate.io**](http://topmate.io/): A large network of Software Engineers from large tech companies offer many services like mock system design interviews here.
- [**LinkedIn**](http://linkedin.com/): Reaching out to individuals who offer mock interview services is also a good idea.
- **Fiverr/Upwork:** Reaching out to freelancer interviewers who have a full-time job with big tech companies could help you get some valuable practice.

Alternatively, if you are like me and are unable to get mock Interviews set up for any reason here are other resources that may be similarly useful.

- This [**breakdown of System Design Interviews by Alex Golec**](https://www.youtube.com/watch?v=oHyaRvnX2G8&t=3s) was a great watch and gave me a really good perspective on what to be mindful of during my System Design interviews.
- **Practice whiteboarding.** Drawing your system design outlines on a virtual whiteboard with a mouse is more difficult than it sounds. Practice your solutions on platforms like [**Excalidraw**](https://excalidraw.com/) to get comfortable with drawing system architectures. Doing this for the large number of problems you would practice in *Step 3* should ensure you are highly efficient during your interviews.
- **Watch a large number of mock interviews**. This helps to develop an understanding of how the interview would progress and what is the expectations from a candidate. This [**mock system design interview by engineers at Google**](https://www.youtube.com/watch?v=S1DvEdR0iUo) is a good starting point. Interviewing.io has a comprehensive list of [**mock interview replays**](https://interviewing.io/mocks?technical=system-design) that you can watch and learn from.
- [**System Design Fight Club**](https://www.youtube.com/@SDFC) **(SDFC):** They have a large number of live design solutions on the channel, which can be helpful.
- [**AI-Powered Mock Interview**](https://www.educative.io/mock-interview): Chat-based AI-powered interview tool that can be useful to polish your system design solutions better.

## Step 5: Company-specific preparations

Last but not least — It is important to fine-tune your preparations based on the company you are applying for. While system design interviews are pretty standard across most tech companies, they do have subtle differences. Learning about this can give your preparation a massive boost. While preparing for my interview with Meta, [*this article*](https://interviewing.io/guides/hiring-process/meta-facebook#meta) gave me a lot of insight regarding what to expect during different steps of my process. They seem to have [articles](https://interviewing.io/guides/hiring-process) for other FAANG companies as well which could help you with your preparations.

Preparing past interview questions for each company is **ABSOLUTELY NECESSARY**. Companies do tend to be repetitive with their System design questions and capitalizing on that should be the right step. Regarding where you can get past interview questions — my go-to resource has always been [**LeetCode Discuss**](https://leetcode.com/discuss/interview-question?currentPage=1&orderBy=hot&query=). Search with the company tag, and you will land on a gold mine of questions a company has been asking (*and also realize how often they get repeated)*.

**Thanks to anyone who read till the end.** **All the best for your System Design Interviews.** You can find me fairly active on [LinkedIn](https://www.linkedin.com/in/aritra-sen-a75009128/) (*profile linked here*) so feel free to reach out for any questions.

*📜* ***P.S.:*** *Here are a few other good sources I came across that helped me a lot during my preparation and might help you too.*

- [System design course by Karan Pratap Singh](https://github.com/karanpratapsingh/system-design)
- [A Senior Engineer’s Guide to the System Design Interview by interviewing.io](https://interviewing.io/guides/system-design-interview)
- [System Design Handbook](https://www.systemdesignhandbook.com/guides/system-design-interview/)
- [Grokking the Modern System Design](https://www.educative.io/courses/grokking-the-system-design-interview) by Educative (*this is a paid course but also has some useful free leasons you should check out)*

[![Aritra Sen](https://miro.medium.com/v2/resize:fill:96:96/1*t500UqEUbYulnu3X4DqQQg.jpeg)](https://medium.com/@sentalkssane?source=post_page---post_author_info--76d64689788b---------------------------------------)[1 following](https://medium.com/@sentalkssane/following?source=post_page---post_author_info--76d64689788b---------------------------------------)

A Googler talking about things he loves — Coding, Music and Coffee:)

## Responses (25)

Write a response[What are your thoughts?](https://medium.com/m/signin?operation=register&redirect=https%3A%2F%2Fmedium.com%2F%40sentalkssane%2Fa-beginners-guide-to-system-design-76d64689788b&source=---post_responses--76d64689788b---------------------respond_sidebar------------------)

```c
High-quality content
```

106

```c
Great read, Thanks for sharing.
```

29

```c
Thanks for sharing! There is a newer playlist from \`jordan has no life\`. I'm adding it here for anyone who's interested: https://www.youtube.com/playlist?list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ
```

21