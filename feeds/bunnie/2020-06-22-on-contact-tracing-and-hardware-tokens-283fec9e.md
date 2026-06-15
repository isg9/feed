---
title: On Contact Tracing and Hardware Tokens
url: https://www.bunniestudios.com/blog/2020/on-contact-tracing-and-hardware-tokens/
published: "2020-06-22T14:25:12Z"
feed: bunnie
guid: https://www.bunniestudios.com/blog/?p=5820
---

# On Contact Tracing and Hardware Tokens

Early in the COVID-19 pandemic, I was tapped by the European Commission to develop a privacy-protecting contact tracing token, which you can read more about at the [Simmel project home page](https://simmel.betrusted.io). And very recently, Singapore has announced the deployment of a TraceTogether token. As part of their launch, I was [invited to participate in a review of their solution](https://www.facebook.com/Vivian.Balakrishnan.Sg/posts/10156902734466207). The urgency of COVID-19 and the essential challenges of building supply chains means we are now in the position of bolting wheels on a plane as it rolls down the runway. As with many issues involving privacy and technology, this is a complicated and nuanced situation that cannot be easily digested into a series of tweets. Thus, over the coming weeks I hope to offer you my insights in the form of short essays, which I will post here.

Since I was only able to spend an hour with the TraceTogether token so far, I’ll spend most of this essay setting up the background I’ll be using to evaluate the token.

**Contact Tracing**

The basic idea behind contact tracing is simple: if you get sick, identify your close contacts, and test them to see if they are also sick. If you do this fast enough, you can contain COVID-19, and most of society continues to function as normal.

However, from an implementation standpoint, there are some subtleties that I struggled to wrap my head around. Dr. Vivian Balakrishnan, the Minister-in-charge of the Smart Nation Initiative, briefly stated at our meeting on Friday that the Apple/Google Exposure Notification system did not reveal the “graph”. In order to help myself understand the epidemiological significance of extracting the contact graph, I drew some diagrams to illustrate contact tracing scenarios.

Let’s start by looking at a very simple contact tracing scenario.

[![](https://bunniestudios.com/blog/images/c19-contact-example.png)](https://bunniestudios.com/blog/images/c19-contact-example.png)

In the diagram above, two individuals are shown, *Person 1* and *Person 2*. We start Day 1 with *Person 1* already infectious yet only mildly symptomatic. *Person 1* comes in contact with *Person 2* around mid-day. *Person 2* then incubates the virus for a day, and becomes infectious late on Day 2. *Person 2* may not have any symptoms at this time. At some future date, *Person 2* infects two more people. In this simple example, it is easy to see that if we can isolate *Person 2* early enough, we could prevent at least two future exposures to the virus.

Now let’s take a look at a more complicated COVID-19 spread scenario with no contact tracing. Let’s continue to assume *Person 1* is a carrier with mild to no symptoms but is infectious: a so-called “ [super spreader](https://www.scmp.com/news/world/united-states-canada/article/3049933/how-one-man-spread-coronavirus-singapore-uk-alps)”.

[![](https://bunniestudios.com/blog/images/c19-no-tracing.png)](https://bunniestudios.com/blog/images/c19-no-tracing.png)

The above graphic depicts the timelines of 8 people over a span of five days with no contact tracing. *Person 1* is ultimately responsible for the infection of several people over a period of a few days. Observe that the incubation periods are not identical for every individual; it will take a different amount of time for every person to incubate the virus and become infectious. Furthermore, the onset of symptoms is not strongly correlated with infectiousness.

Now let’s add contact tracing to this graph.

[![](https://bunniestudios.com/blog/images/c19-contact-tracing.png)](https://bunniestudios.com/blog/images/c19-contact-tracing.png)

The graphic above illustrates the same scenario as before, but with the “platonic ideal” of contact tracing and isolation. In this case, *Person 4* shows symptoms, seeks testing, and is confirmed positive early on Day 4; their contacts are isolated, and dozens of colleagues and friends are spared from future infection. Significantly, digging through the graph of contacts also allows one to discover a shared contact of *Person 4* and *Person 2*, thus revealing that *Person 1* is the originating asymptomatic carrier.

There is a subtle distinction between “contact tracing” and “contact notification”. Apple/Google’s “Exposure Notification” system only perform notifications to the immediate contacts of an infected person. The significance of this subtlety is hinted by the fact that the protocol was originally named a “Privacy Preserving Contact Tracing Protocol”, but renamed to the more accurate description of [“Exposure Notification” in late April](https://www.vox.com/recode/2020/4/24/21234420/apple-google-contact-tracing-exposure-notification-update).

To better understand the limitations of exposure notification, let’s consider the same scenario as above, but instead of tracing out the entire graph, we only notify the immediate contacts of the first person to show definite symptoms – that is, *Person 4*.

[![](https://bunniestudios.com/blog/images/c19-exposure-notification.png)](https://bunniestudios.com/blog/images/c19-exposure-notification.png)

With exposure notification, carriers with mild to no symptoms such as *Person 1* would get misleading notifications that they were in contact with a person who tested positive for COVID-19, when in fact, it was actually the case that *Person 1* gave COVID-19 to *Person 4*. In this case, *Person 1* – who feels fine but is actually infectious – will continue about their daily life, except for the curiosity that everyone around them seems to be testing positive for COVID-19. As a result, some continued infections are unavoidable. Furthermore, *Person 2* is a hidden node from *Person 4*, as *Person 2* is not within *Person 4*’s set of immediate notification contacts.

In a nutshell, Exposure Notification alone cannot determine causality of an infection. A full contact “graph”, on the other hand, can discover carriers with mild to no symptoms. Furthermore, it has been well-established that a significant fraction of COVID-19 infections show mild or no symptoms for extended periods of time – these are not “rare” events. These individuals are infectious but are well enough to walk briskly through crowded metro stations and eat at hawker stalls. Thus, in the “local context” of Singapore, asymptomatic carriers can seed dozens of clusters in a matter of days if not hours, unlike less dense countries like the US, where infectious individuals may come in contact with only a handful of people on any given day.

The inability to quickly identify and isolate mildly symptomatic super-spreaders motivates the development of the local TraceTogether solution, which unlocks the potential for “full graph” contact tracing.

**On Privacy and Contact Tracing**

Of course, the privacy implications of full-graph contact tracing are profound. Also profound are the potential health risks and loss of life absent full-graph contact tracing. There’s also a proven solution for containing COVID-19 that involves no sacrifice of privacy: an extended Circuit-Breaker style lockdown. Of course, this comes at the price of the economy.

Of the three elements of privacy, health, or economy, it seems we can only pick two. There is a separate and important debate about which two we should prioritize, but that is beyond the context of this essay. For the purpose of this discussion, let’s assume contact tracing will be implemented. In this case, it is incumbent upon technologists like us to try and come up with a compromise that can mitigate the privacy impact while facilitating public policy.

Back in early April, Sean ‘xobs’ Cross and I were contacted by the [European Commission’s NGI program via NLnet to propose a privacy-protecting contact tracing hardware token](https://nlnet.nl/project/Simmel/). The resulting proposal is called “ [Simmel](https://simmel.betrusted.io)”. While not perfect, the salient privacy features of Simmel include:

1. **Strong isolation of user data**. By disallowing sensor fusion with the smartphone, there is zero risk of GPS or other geolocation data being leaked. It is also much harder to do metadata-based attacks against user privacy.
2. **Citizens are firmly in control**. Users are the physical keeper of their contact data; no third-party servers are involved, until they volunteer their data to an authority by surrendering the physical token. This means in an extreme case, a user has the option of physically destroying their token to erase their history.
3. **Citizens can temporarily opt-out**. By simply twisting the cap of the token, users can power the token down at any time, thus creating a gap in their trace data ( *note: this feature is not present on the first prototypes*).
4. **Randomized broadcast data**. This is a protocol-level feature which we recommend to defeat the ability for third parties (perhaps an advertising agency or a hostile government) from piggy backing on the protocol to aggregate user locations for commercial or strategic benefit.

**Why a Hardware Token?**

But why a hardware token? Isn’t an app just better in so many ways?

At our session on Friday, the TraceTogether token team stated that Singapore needs hardware tokens to better serve two groups: the underprivileged, and iPhone users. The underprivileged can’t afford to buy a smartphone; and iPhone users can only run Apple-approved protocols, such as their Exposure Notification service (which does not enable full contact tracing). In other words, iPhone users, like the underprivileged, also don’t own a smartphone; rather, they’ve bought a phone that can only be used for Apple-sanctioned activities.

Our Simmel proposal makes it clear that I’m a fan of a hardware token, but for reasons of privacy. It turns out that apps, and smartphones in general, are bad for user privacy. If you genuinely care about privacy, you would leave your smartphone at home. The table below helps to illustrate the point. A red X indicates a known plausible infraction of privacy for a given device scenario.

[![](https://bunniestudios.com/blog/images/c19-privacy-table.png)](https://bunniestudios.com/blog/images/c19-privacy-table.png)

The tracing token (as proposed by Singapore) can reveal your location and identity to the government. Nominally, this happens at the point where you surrender your token to the health authorities. However, in theory, the government could deploy tens of thousands of TraceTogether receivers around the island to record the movement of your token in real-time. While this is problematic, it’s relevant to compare this against your smartphone, which typically broadcasts a range of unique, unencrypted IDs, ranging from the IMEI to the wifi MAC address. Because the smartphone’s identifiers are not anonymized by default, they are potentially usable by anyone – not just the government – to identify you and your approximate location. Thus, for better or for worse, the design of the TraceTogether token does not meaningfully change the status quo as far as “big infrastructure” attacks on individual privacy.

Significantly, the tracing token employs an anonymization scheme for the broadcast IDs, so it should not be able to reveal anything about your location or identity to third parties – only to the government. Contrast this to the SafeEntry ID card scanner, where you hand over your ID card to staff at SafeEntry kiosks. This is an arguably less secure solution, as the staff member has an opportunity to read your private details (which includes your home address) while scanning your ID card, hence the boxes are red under “location” and “identity”.

Going back to the smartphone, “typical apps” – say, Facebook, Pokemon Go, Grab, TikTok, Maps – are often installed with most permissions enabled. Such a phone actively and routinely discloses your location, media, phone calls, microphones, contacts, and NFC (used for contactless payment and content beaming) data to a wide variety of providers. Although each provider claims to “anonymize” your data, it has been well-established that so much data is being published that it is [virtually a push of a button to de-anonymize that data](https://www.nytimes.com/interactive/2019/12/19/opinion/location-tracking-cell-phone.html). Furthermore, your data is subject to surveillance by several other governments, thanks to the broad power of governments around the world to lawfully extract data from local service providers. This is not to mention the ever-present risk of malicious actors, exploits, or deceptive UI techniques to convince, dupe, or coerce you to disclose your data.

Let’s say you’re quite paranoid, and you cleverly put your iPhone into airplane mode most of the time. Nothing to worry about, right? Wrong. For example, in airplane mode, the [iPhone still runs its GPS receiver and NFC](https://www.macobserver.com/analysis/ios-airplane-mode-gps-nfc/). An independent analysis I’ve made of the iPhone also reveals [occasional, unexplained blips on the wifi interface](https://assets.pubpub.org/aacpjrja/AgainstTheLaw-CounteringLawfulAbusesofDigitalSurveillance.pdf).

To summarize, here are the core arguments for why a hardware token offers stronger privacy protections than an app:

**No Sensor Fusion**

The data revealed by a hardware token is strongly limited by its inability to perform “sensor fusion” with a smartphone-like sensor suite. And even though I was only able to spend an hour with the device, I can say with a high degree of confidence that the TraceTogether token has little to no capability beyond the requisite BLE radio. Why do I say this? Because physics and economics:

• **Physics**: more radios and sensors would draw more power. Ever notice how your phone’s battery life is shorter if location services are on? If the token is to last several months on such a tiny battery, there simply is not enough power available to operate much more than the advertised BLE functions.

• **Economics**: more electronics means more cost. The publicly disclosed tender offering places a cap on the value of parts at S$20, and it essentially has to be less than that because the producer must also bear their development cost out of the tender. There is little room for extraneous sensors or radios within that economic envelope.

[![](https://bunniestudios.com/blog/images/c19-tt-token-battery.jpg)](https://bunniestudios.com/blog/images/c19-tt-token-battery.jpg)

*Above: the battery used in the TraceTogether token. It has a capacity of 1000mAh. The battery in your smartphone has a capacity of around 3x of this, and requires daily charging.*

The economics argument is weaker than the physics argument, because the government could always prepare a limited number of “special” tokens to track select individuals at an arbitrary cost. However, the physics argument still stands – no amount of money invested by the government can break the laws of physics. If Singapore could develop a mass-manufacturable battery that can power a smartphone sensor suite for months in that form factor – well, let’s just say the world would be a very different place.

**Citizen Hegemony over Contact History**

Assuming that the final TraceTogether token doesn’t provide a method to repurpose the Bluetooth Low-Energy (BLE) radio for data readout (and this is something we hope to confirm in a future hackathon), citizens have absolute hegemony over their contact history data, at least until they surrender it in a contact tracing event.

As a result the government is, perhaps inadvertently, empowering citizens to rebel against the TraceTogether system: one can always crush their token and “opt-out” of the system (but please remove the battery first, otherwise you may burn down your flat). Or perhaps more subtly, you can “forget your token at home”, or carry it in a metallized pouch to block its signal. The physical embodiment of the token also means that once the COVID-19 pandemic is under control, destroying the token definitively destroys the data within it – unlike an app, where too often uninstalling the app simply means an icon is removed from your screen, but some data is still retained as a file somewhere on the device.

In other words, a physical token means that an earnest conversation about privacy can continue in parallel with the collection of contact tracing data. So even if you are not sure about the benefit of TraceTogether today, carrying the token allows you to defer the final decision of whether to trust the government until the point where you are requested to surrender your token for contact trace extraction.

If the government gets caught scattering BLE receivers around the island, or an errant token is found containing suspicious circuitry, the government stands to lose not just the trust of the people, but also access to full-graph contact tracing as citizens and residents dispose of tokens *en masse*. This restores a certain balance of power, where the government can and will be held accountable to its social contract, even as we amass contact tracing data together as a whole.

**Next Steps**

When I was tapped to independently review the TraceTogether token, I told the government that I would hold no punches – and surprisingly, they still invited me to the introductory session last Friday.

This essay framed the context I will use to evaluate the token. “Exposure notification” is not sufficient to isolate mildly symptomatic carriers of COVID-19, whereas “full graph” contact tracing may be able to make some headway against this problem. The good news is that the introduction of a physically embodied hardware token presents a safer opportunity to continue the debate on privacy while simultaneously improving the collection of contact tracing data. Ultimately, deployment of a hardware token system relies upon the compliance of citizens, and thus it is up to our government to maintain or earn our trust to manage our nation’s best interests throughout this pandemic.

I look forward to future hackathons where we can really dig into what’s running inside the TraceTogether token. Until then, stay safe, stay home when you can, and when you must go outside, wear your mask!

PS: You should also check out [Sean ‘xobs’ Cross’ teardown of the TraceTogether token!](https://xobs.io/trace-together-token-teardown/)
