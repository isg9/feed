---
title: CSDN Interview Translation
url: https://www.bunniestudios.com/blog/2013/csdn-interview-translation/
published: "2013-07-04T05:07:28Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=3234
---

# CSDN Interview Translation

I recently did an interview for CSDN.net, which stands for “China Software Developer Network”, or more colloquially, “Programmer Magazine”, about open source hardware, and what it means to be a hacker.

The [interview itself is in Chinese](http://www.csdn.net/article/2013-07-03/2816095) ( [print version](http://bunniefoo.com/bunnie/csdn-201307.pdf)), but I thought I’d post the English translation here for non-Chinese readers.

![](http://bunniestudios.com/blog/images/csdn-screenshot.png)

\[biographical details and photo omitted from this translation. Caption to the bio photo reads: “My PhD advisor in college, Dr. Tom Knight, had a profound impact upon me. I often asking myself “what would Tom do?” when I am stuck or lost, and the answer I come back with is usually the right one.”\]

**About Open-source Hardware and the Maker movement**

**CSDN**: The Maker and Open hardware movements attract a lot of attention recent years, Chris Anderson wrote a book Makers, Paul Graham called it The Hardware Renaissance. From your perspective and observation, how do you think about this movement will affect ordinary people, developers and our IT industry?

**bunnie**: This movement, as it may be, is more of a symptom than a cause, in my opinion. Let’s review how we got here today.

In 1960, for all practical purposes there was only hardware, and it was all open. When you bought a transistor radio, it had its schematic printed in the back. If it broke, you had to fix it yourself. It was popular to buy kits to make your own radios.

Around 1980-1990, the personal computer revolution began. Computers started to become powerful enough to run software that was interesting and enabling.

From 1990-2005, Moore’s Law drove computers to be twice as fast, and have twice as much memory every 1.5-2 years. All that mattered in this regime is the software; unless you could afford to fab a chip in the latest technology, it wasn’t worth it to make hardware, because by the time you got the components together a new chip was out that made your design look slow. It also wasn’t worth it to optimize software. By the time you made your software optimized, you could have bought a faster computer and run the old software even faster. What mattered was features, convenience, creativity. “Making” fell out of fashion: you had to ship code or die, there was no time to make.

From 2005-2010, computers didn’t get much faster in terms of MHz, but they got smaller. Smartphones were born. Everything became an app, and everything is becoming connected.

From about 2010-now, we find that Moore’s Law is slowing down. This slowdown is rippling through the innovation chain. PCs are not getting faster, better, or cheaper in a meaningful way; basically we buy a PC just to replace a broken one, not because it’s so much better. It’s a little bit too early to tell, but smartphones may also be solidifying as a platform: the iPhone5 is quite similar to the iPhone4; the Samsung phones are also looking pretty similar across revisions.

How to innovate? how to create market differentiation? With Moore’s Law slowing down, it’s now possible to innovate in hardware and not have your innovation look slow because a new chip came out. You have steady platforms (PCs, smartphones, tablets) which you can target your hardware ideas toward. You don’t have to necessarily be fabbing chips just to have an advantage. We are now sifting through our past, looking for niches that were overlooked during the initial fast-paced growth of technology. Even an outdated smartphone motherboard can look amazing when you put it in quadcopters, satellites, HVAC systems, automobiles, energy monitoring systems, health monitoring systems, etc.

Furthermore, as humans we fundamentally feel differently toward “real” things as opposed to “virtual” things. As wonderful as apps are, a human home consists of more than a smartphone, a food tray, a bed and a toilet. We still surround ourselves with knick-knacks, photos of friends, physical gifts we give each other on special occasions. I don’t think we will ever get to the point where getting a “virtual” teddy bear app will be as valued as getting a “real” teddy bear.

As a result, there will always be a place for people to make hardware that fills this need for tangible goods. This hardware will merge more technology and run more software, but in the end, there is a space for Makers and hardware startups, and that space is just getting bigger now that hardware technology is stabilizing.

**CSDN**: Compared with the past, Arduino and Raspberry Pi’s appearance seems to reduce the threshold of doing hardware design, what do you think this will effect hardware product industry? Do you think these platforms will bring real leaps and bounds in the industry? Or to make really innovative hardware product, what do we need?

**bunnie**: Arduino and RPi serve specific market niches.

Arduino’s key contribution is the reduction of computation to a easy-to-use physical form. It was made first and foremost by designers and artists, and less so by technologists. This unique perspective on technology turned out to be very powerful: it turns out people who aren’t programmers or hardware designers also want to access hardware technology.

Some very moving and deep interactive art pieces have been made using the Arduino, allowing hardware to transcend menial control applications into something that changes your mood or makes you think about life differently. I think Arduino is just the first step toward taking the “tech” out of technology and letting every day people not just use technology, but create with it. There will be other platforms, for sure.

Raspberry Pi is a very inexpensive embedded hardware reference module; it is cheap enough that for many applications, it’s just fine to buy the RPi as-is and you don’t have to design your own hardware. I think there will also be other followers in their footsteps. The nice thing about RPi for hardware professionals is that instead of buying a reference design and then having to spin your own board, you can just buy the RPi and ship it in the product. For people who have relatively low-volume products, this makes sense.

As an ongoing trend, I see product design becoming more feasible at low volumes. There will still be the million-unit blockbusters for things like smartphones and coffee makers, but there will also be a new market for 1k-10k of something, but with a much higher margin. These small run products will be developed and sold by teams of just one or two people, so that the profit on such a small run of product is still a good living for the individuals. The key to the success of these products is that they are highly customized and help solve the exact problem a small group of users have, and because of this the users are willing to pay

more for it.

**CSDN**: When new concept or technologies first appear, they always cause a lot of optimism discussion, but most of them, only take a long time development, will really affect our lives. When we are talking about Marker or Open hardware movement, are we too optimistic? For the average person, if there are common misunderstandings about this field?

**bunnie**: Yes, it does take a long time for technology to really change our lives.

The Maker movement, I think, is less about developing products, and more about developing people. It’s about helping people realize that technology is something man-made, and because of this, every person has the power to control it: it just takes some knowledge. There is no magic in technology. Another way to look at it is, we can all be magicians with a little training.

Open Hardware is more of a philosophy. The success or failure of a product is largely disconnected with whether the hardware is open or closed. Closing hardware doesn’t stop people from cloning or copying, and opening hardware doesn’t mean that bad ideas will be copied simply because they are open. Unlike software, hardware requires a supply chain, distribution, and a network of relationships to build it at a low cost. Because of this, hardware being open or closed is only a small part of the equation for hardware, and mostly it’s a question of how much you want to involve end users or third parties to modify or interoperate with your product.

**CSDN**: When we look at the future of open source hardware, could we analogy it with the open source software industry that we have already seen (many commercial companies also support open source software, many software companies to provide support for the open source software for living)? What are the difference between them?

**bunnie**: I don’t think the analogy is quite the same. In software, the cost to copy, modify, and distribute is basically zero. I can download a copy of linux, and run “make” and have the same high-quality kernel running on my desktop as runs on top-end servers and supercomputers.

In hardware, there is a real cost to copy hardware. And the cost of the parts, the factories and skilled workers used to build them, the quality control procedures, and the process to build are all important factors in how the final hardware cost, looks, feels, and performs. Simply giving someone a copy of my schematics and drawings doesn’t mean they can make exactly my product. Even injection molding has art to it: if I give the same CAD drawing to two tooling makers, the outcome can be very different depending on where the mold maker decides to place the gates, the ejector pins, the cooling for the mold, the mold cycle time, temperature, etc.

And then there is the distribution channel, reverse logistics, financing, etc.; even as the world becomes more efficient at logistics, you will never be able to buy a TV as easily as you can download the movies that you watch on the same TV.

**CSDN**: What kind of business model do you think is ideal for open source hardware company? Could you give an example?

**bunnie**: One of my key theories behind open source hardware is that hardware, at least at the level of schematics and PCB layout, is “essentially open”. This is because for a relatively small amount of money you can pay services to extract the detail required to copy a PCB design. Therefore an asymptotic assumption is that once you have shipped hardware, it can be copied.

If you can accept this assumption, then not releasing schematics and PCB layouts will not stop people from copying your goods. It will be copied if someone wants to copy it. So it makes no difference to copying whether or not you have shared your design files.

However, if you do share your design files, it does make a difference to a different and important group of people. There are other businesses and individual innovators who could use your design files to design accessories, upgrades, or third party enhancements that rely upon your product.

Thus, in a “limiting case”, sharing your design files doesn’t change the situation on copying, but does improve your opportunity for new business relationships. Therefore, the practical suggestion is to just share the design files, using the guidelines of the open source hardware licenses to help reserve a few basic rights and protections.

Clearly, there are some hardware strategies that are not compatible with the idea of open hardware. If your sole value to the consumer is your ability to make stand-alone hardware, and you have no strategic advantage in terms of cost, then you would like to keep your plans secret to try to delay the low-cost copiers for as long as you can.

However, we are finding today that the most innovative products are not just a piece of hardware, but they also involve software and services. Open hardware business models work better when they are in such hybrid products. In many cases, consumers are willing to pay annuity revenue in some form (e.g. subscriptions, advertising, upsells, accessories, royalties and upgrades) for many types of products. In fact, it is most profitable to just collect these fees and not involve yourself in the hardware manufacturing portion; and it is much easier to control access to an ongoing service than it is to the plans of a piece of hardware.

Thus, if you are running an on-line service that is coupled to your hardware, open hardware makes a lot of sense: if your on-line service is profitable, letting other people copy the hardware, sell it, and then add more users to your on-line service simply means you get more revenue without more risk in the production of hardware.

**CSDN**: We know you often come to China and know a lot about this country. China’s software technology is not advanced, do you think its current position — the world factory center will help it improve its overall level of technology? How can this country change to a design, research and development focus place but not just a manufacturing center? What is China missing?

**bunnie**: I wouldn’t say I know much about China. I know a little bit about one small corner of China in one specific area — hardware manufacturing. If there is one thing I *do* know about China, it’s that it is a very big country, with many different kind of people, and a long history that I am only beginning to understand.

However, the history of high technology is almost entirely contained within my life span, so I can comment on the relationship between high technology and people, from which we can derive some perspective about China.

The first observation is that every technology power-house today started with manufacturing. The US started as simple colonies of Britain, mining ores, trapping furs and farming cotton and tobacco. Over time, the US had steel mills and linen production. It wasn’t until the early 1900’s before the US really started to rise in developing original technology, and not until the mid 1900’s before things really took off.

The same goes for Japan. They started in manufacturing, copying many of the US made goods. In fact the first cars and radios they made were, if you believe the historical accounts, not so good. It took the US and Japan decades to go from a manufacturing-based economy to a service-based economy.

If you compare that to China, the electronics manufacturing industry started there maybe only 20 years ago, at most, and China is just turning the corner from being a manufacturing-oriented economy to one that can do more design and software technology. I believe this is a natural series of events: some portion of entry-level workers will eventually become technicians, then some technicians become designers, then some designers become successful entrepreneurs.

In terms of concrete numbers, if you have 10 million factories workers, maybe 1% of them will learn enough from their job such that after a few years they can become technicians. This gives you 100,000 technicians. After a few years of technician work, maybe 1% will gain enough skill to become original designers. This gives you 1,000 designers. These experienced, grass-roots designers become the core of your entrepreneurial economy, and from there the economy begins to transform.

From a thousand companies, you will eventually winnow down to just a handful of global brand companies. This whole process takes a decade or two, and I believe we are currently witnessing China going through the final phase of the transformation, where we now have a lot of people in Shenzhen who have the experience of manufacturing, the wisdom to do design, and now are just starting to apply their talent to innovation and original product design. The next decade will be an exciting one for China’s technology industry, if the current policies on economic and intellectual development stay roughly on course.

This pattern applies primarily to hardware or hardware-dominated products. Software products have a similar pattern, but I believe there are unique cultural aspects that can give the West an advantage in software design. In hardware, if a process is not efficient or producing low yield, one can easily identify the root cause and produce direct physical evidence of the problem. Hardware problems, in essence, are indisputable.

In software, if code is not efficient or poorly written, it’s very hard to identify the exact problem that causes it. One can see the evidence of programs crashing or running slowly, but there is no broken wire or missing screw you can simply hold up and show everyone to show why the software is broken. Instead, you need to sit down with your collaborators and review a complex design, consider many opinions, and ultimately you need to identify a problem which is ultimately due to a bad decision made by an individual, and nothing more than that. All software APIs are simply constructs of human opinions: nothing more.

Asian cultures have a strong focus on guanxi, reputation, and respect for the elders. The West tends to be more rebellious and willing to accept outsiders as champions, and they have less respect for the advice of elders. As a result, I think it’s very culturally difficult in an Asian context to discuss code quality and architectural decisions. The field of software itself is only 30 years old, and older, more experienced engineers are also the most out of date in terms of methodology and knowledge. In fact, the young engineers often have the best ideas. However, if it’s culturally difficult for the young engineers to challenge the decisions of the elder engineers, you end up with poorly architected code, and you have no hope to be competitive.

It’s not hopeless to overcome these obstacles, but it requires a very strong management philosophy to enforce the correct incentives and culture. The workers should be rewarded fairly for making correct decisions, and there can be no favorites based upon friendship, relationship, or seniority. Senior engineers and managers must see a real financial reward for accepting their mistakes, instead of saving face by forcing junior engineers to code patches around their bad high level decisions. Usually, this alignment is achieved in US contexts by sharing equity in a company among the engineers, so that the big payout only comes if the company as a whole survives, regardless of the ego of the individual.

**CSDN**: What do you think the role (or relationship) of individual maker and commercial company in the future? And as individual maker may not only compete with commercial companies, but also will compete with other makers in the future, what are the factors critical to the success of a product?

**bunnie**: As a general trend, MOQs are reducing and innovation is getting closer to the edge. So I think commercial companies will see more competition from Makers, especially as the logistics industry transforms itself into a API that can plug directly into websites.

At the end of the day, the most critical factors to success will still be how much value the consumer will perceive from the product. A part of this is related to superior features and good product quality, but also an important part of this is in the presentation to the consumer and how clearly the benefits are explained. As a result, it’s important to make sure the product is visually appealing, easy to use, and to create marketing material that clearly explains the benefit of the product to the customer. This is often a challenge for individual makers, because their talent is normally in the making of the product’s technical value, but less so on the marketing and sales value. Makers who can master both will have an edge over makers who focus specifically on offering just a technical value.

**About Hardware Hackers**

**CSDN**: You have participated in the development process of many products, what is your personal goal? In this long period, what is the greatest pleasure?

**bunnie**: I would like to make people happy by building things that improve their life in some way. The greatest pleasure is to see someone enjoying something you have made because you have improved their life in some small way. Sometimes your product is solving a big problem for them. Other times it is more whimsical and happiness comes from fun or beauty. But either way, you are helping another person.

That is what is important to me. One thing I have learned in the past few years is that money beyond a certain level doesn’t make me any happier. This makes me difficult to work with, because it’s hard for people to just hire my service by offering me a lot of money. Instead, they need to convince me that the activity will somehow also make people happy.

Another important goal for me is to just understand how the world works. I have a natural curiosity and I want to learn and understand all kinds of things. The universe has a lot of patterns to it, and sometimes you will find seemingly unrelated pieces fitting together just like magic. Discovering these links and seeing the world fit together like a big jigsaw puzzle is very profound and satisfying.

**CSDN**: Which project you focus on recently? Why you choose them?

**bunnie**: I haven’t really been focused, actually. Recently, I have intentionally been \*not\* focused. If you focus on one idea for too long, many good innovative ideas just pass you by.

One of the hardest parts of being an entrepreneur or innovator is to have the patience to review many ideas, know when to say no to bad ideas, and then cultivate a few good ideas at the same time, and to accept many changes to your concept along the way. This process can take months, if not years.

I have a project to build my own laptop, but it’s not meant to be a business. It’s a subject meant to encourage personal growth, to challenge my abilities and learn things that I don’t know about the entire computer design process. I have a project to reverse engineer the firmware in SD cards. It’s also not a business, it’s meant to satisfy my curiosity. I have a project to learn how companies in Shenzhen build such amazing phones for such a low price. If I can learn some of their techniques, then hopefully I can apply this knowledge to create new and compelling products that make people happy. I have a project to build flexible circuit stickers. It will hopefully help introduce more people into using technology in their everyday life. It will also challenge my understanding of manufacturing processes, as I must develop a few new processes that don’t exist today to make the stickers robust and inexpensive enough for every day use. I have a project to help advise new entrepreneurs and innovators, and help get them started on their adventures. I may not have the best ideas, or all the talent needed to make a business, but if I can help teach others to fly, then maybe that can make up for my inability to find success in business.

So, I have many projects, and no focus. Maybe someday I will find one to focus on — sometimes it is also important to focus — but as of today, I haven’t found what I’m looking for.

**CSDN**: Failure tends to give people more experience, could you talk about the not-so-successful projects you have participated, or if you’ve ever seen other failed projects that inspired you.

**bunnie**: My life is a story of failures. The only thing I have done repeatedly and reliably is fail. However, I have two rules when handling failure: (1) don’t give up and (2) don’t make the same mistake twice. If you follow these rules, eventually, you will find a success after many failures.

Actually, I have an interview that focuses on one of my recent failures. You can [read it here](http://blog.makezine.com/2012/04/30/makes-exclusive-interview-with-andrew-bunnie-huang-the-end-of-chumby-new-adventures/).

**CSDN**: Your book Hacking the Xbox has been published ten years, for people who want to learn reverse engineering, or want to become a hardware hacker today, whether these experiences and skills has still apply?

**bunnie**: I’d like to think that the core principles covered in the book are still relevant today. The Xbox is simply an example of how to do things, but the approach described in the book and the techniques are applicable to a broad range of problems.

For the Chinese audience, I have found the mobile phone repair manuals to be quite interesting to read. I have bought a couple of them, and even though I can’t read Chinese well, I still find them pretty interesting. Sometimes their descriptions on the theory of electronics are not completely accurate, but practically speaking they are good enough, and it’s a quick way to get started while learning immediately useful skills in repairing phones.

There’s also a Chinese magazine, 无线电, which I have found to be quite good. If you get started building the projects in there, I think you will learn very quickly.

**CSDN**: For users, the new Xbox One has more stringent restrictions, what do you think about this? Do you have interesting to explore this black box and upgrade your book?

**bunnie**: I haven’t done much work on video game consoles in a while — there is a whole new generation of console hackers who are excited to explore them, and I’m happy for that. As for the Xbox One security, I’m sure it is one of the most secure systems built. They did a very good job on the Xbox360, and I know some of the Xbox One security team members and they have a very solid understanding of the principles needed to build secure hardware. It should be very hard to crack.

That being said, I’m glad I have no desire to buy or use one. I think it would become very frustrated with their use policies and restrictions very quickly.

**CSDN**: There are a lot of controversy about if electronic devices should have a lock to prevent user rooting, what do you think about this? Whether there is a contradiction between ensure the safety of user and make user get complete control of their device?

**bunnie**: I believe users should “own” their hardware, and “owning” means having the right to modify, change, etc. including root access rights. If the company has a concern about users being unsafe, then it’s easy enough to include an “opt-out” where users can simply select an electronic waiver form, and give up their support and warranty right to gain access to their own machine. Most people who care to root their machine are already smarter than the phone support they would be calling inside the company, so anyways it’s not a problem.

The laws have changed to make it illegal to do some rooting activities, even to hardware that you bought and own. I think this reduction in our natural rights of ownership is dangerous and can lead to consumers being put in an unfair situation, and it also discourages consumers to explore and learn more about the technology they have become so dependent upon.

**CSDN**: The integration of hardware is increasing, do you think doing hardware hacking is getting more and more difficult or do you worry about the hardware hackers to extinct? How could we change this situation?

**bunnie**: The increasing integration has been true for a long time — from the TX-0 which just used transistors, to the Apple II which used TTL ICs, to the PCs which used controller chipsets, to the mobile phones which have just a single SoC now. It does make it harder to hack some parts, but there is always opportunities at the system integration level.

In other words, I still think there is art in hardware, just the level at which hardware hackers have to work gets higher every day, and this is a good thing, because it means our hacks are also getting more powerful with time as well.

**CSDN**: Your book is dedicated to Aaron Swartz, could you talk about why do you think the hacker spirit is important in our era today.

**bunnie**: The hacker spirit is the ultimate expression of human problem solving ability. It’s about the ability to see the world for what it is, and not the constructs and conventions that society puts in place. A brick is not just used to make buildings, it can be a doorstop, a weapon, a paperweight, a heating ballast, or it can be ground up and used for soil. Hackers question convention through the lens of doing what’s most practical and correct for the situation at hand: they see things for what they are, and not by the labels put on them. Sometimes their methods are not always harmonious, as hackers often times prioritize doing the right thing over being nice or playing by the rules.

I find the more difficult situations become, the more pervasive and stronger the hacker spirit becomes among common people. I see evidence of this around the world. It is linked to the human will to survive and to thrive. I think it’s important for a society to culture and tolerate the hacker spirit. Not everyone has it, but the few who do have it help make society more resilient and survivable in hard times.

**CSDN**: Do you have other words you would like to share with Chinese readers?

**bunnie**: Recently, I was reading some comments on a Chinese web forum, and it seems that many Chinese regard the term “Shanzhai” as a negative term. This was surprising to me, because as an outsider, I feel that the Shanzhai have done a lot of very interesting and useful innovation. I think in English, we have a similar problem. The term “hacker” in English started as a good term, but over time became associated with many kinds of negative acts. Recently the term “Maker” was coined to distinguish between the positive and negative aspects of hackers (I still call myself a hacker because I still adhere to the traditional definition of the word).

It may be easier to explain the innovation happening in China, if in Chinese a similar linguistic bifurcation could happen. I had recently proposed referring to the innovative, open aspects of what the Shanzhai do as “gongkai” (公开), referring to their method of sharing design files. Significantly, the term 开放 as used in 开放源代码 I feel doesn’t quite apply, because it refers to a specific Western-centric legal aspect of being open, which is not applicable to the methods engaged in the Chinese ecosystem.

However, the fact that China has found its own way to share IP, unique from the Western system, doesn’t mean it’s bad. I think in fact, it’s quite interesting and I’m very curious to see where it goes. Since I see positive value in some of the methods that the Shanzhai use, I’d propose using the more positive/generic term “gongkai” to describe the style of IP sharing commonly engaged in China.

But then again, who am I to say — I’m not a native Chinese speaker, and maybe there is a much better way to address the situation.
