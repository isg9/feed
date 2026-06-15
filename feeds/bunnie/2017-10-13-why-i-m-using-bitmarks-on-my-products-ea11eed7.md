---
title: Why Iâ€™m Using Bitmarks on my Products
url: https://www.bunniestudios.com/blog/2017/why-im-using-bitmarks-on-my-products/
published: "2017-10-13T14:32:55Z"
feed: bunnie
guid: https://www.bunniestudios.com/blog/?p=4981
---

# Why Iâ€™m Using Bitmarks on my Products

One dirty secret of hardware is that a profitable business isn’t just about design innovation, or even product cost reduction: it’s also about how efficiently one can move stuff from point A to B. This explains the insane density of hardware suppliers around Shenzhen; it explains the success of Ikea’s flat-packed furniture model; and it explains the rise of Amazon’s highly centralized, highly automated warehouses.

Unfortunately, reverse logistics – the system for handling returns & exchanges of hardware products – is not something on the forefront of a hardware startup’s agenda. In order to deal with defective products, one has to ship a product first – an all-consuming goal. However, leaving reverse logistics as a “we’ll fix it after we ship” detail could saddle the venture with significant unanticipated customer support costs, potentially putting the entire business model at risk.

This is because logistics are much more efficient in the “forward” direction: the cost of a centralized warehouse to deliver packages to an end consumer’s home address is orders of magnitude less than it is for a residential consumer to mail that same parcel back to the warehouse. This explains the miracle of Amazon Prime, when overnighting a pair of hand-knit mittens to your mother somehow costs you $20. Now repeat the hand-knit mittens thought experiment and replace it with a big-screen TV that has to find its way back to a factory in Shenzhen. Because the return shipment can no longer take advantage of bulk shipping discounts, the postage to China is likely more than the cost of the product itself!

Because of the asymmetry in forward versus reverse logistics cost, it’s generally not cost effective to send defective material directly back to the original factory for refurbishing, recycling, or repair. In many cases the cost of the return label plus the customer support agent’s time will exceed the cost of the product. This friction in repatriating defective product creates opportunities for unscrupulous middlemen to commit warranty fraud.

The basic scam works like this: a customer calls in with a defective product and gets sent a replacement. The returned product is sent to a local processing center, where it may be declared unsalvageable and slated for disposal. However, instead of a proper disposal, the defective goods “escape” the processing center and are resold as new to a different customer. The duped customer then calls in to exchange the same defective product and gets sent a replacement. Rinse lather repeat, and someone gets rich quick selling scrap at full market value.

Similarly, [high-quality counterfeits](https://www.bunniestudios.com/blog/?p=208) can sap profits from companies. [Clones of products](https://www.bunniestudios.com/blog/?page_id=1022) are typically produced using cut-rate or recycled parts but sold at full price. What happens when customers then find quality issues with the clone? That’s right – they call the authentic brand vendor and ask for an exchange. In this case, the brand makes zero money on the customer but incurs the full cost of supporting a defective product. This kind of warranty fraud is pandemic in smart phones and can cost producers many millions of dollars per year in losses.

[![](https://bunniestudios.com/blog/images/microsd_1.jpg)](https://www.bunniestudios.com/blog/?page_id=1022)

*High-quality clones, like the card on the left, can cost businesses millions of dollars in warranty fraud claims.*

Serial numbers help mitigate these problems, but it’s easy to guess a simple serial number. More sophisticated schemes tie serial numbers to silicon IDs, but that necessitates a system which can reliably download the serialization data from the factory. This might seem a trivial task but for a lot of reasons – from failures in storage media to human error to poor Internet connectivity in factories – it’s much harder than it seems to make this happen. And for a startup, losing an entire lot of serialization data due to a botched upload could prove fatal.

As a result, most hardware startups ship products with little to no plan for product serialization, much less a plan for reverse logistics. When the first email arrives from an unhappy customer, panic ensues, and the situation is quickly resolved, but by the time the product arrives back at the factory, the freight charges alone might be in the hundreds of dollars. Repeat this exercise a few dozen times, and any hope for a profitable run is rapidly wiped out.

I’ve wrestled with this problem on and off through several startups of my own and finally landed on a solution that looks promising: it’s reasonably robust, fraud-resistant, and dead simple to implement. The key is the [bitmark](https://bitmark.com/) – a small piece of digital data that links physical products to the blockchain.

[![](https://cdn-images-1.medium.com/fit/c/72/72/1*OUNO2fxFLUR1nUYsVTXlsA.png)](https://bitmark.com)

Most people are familiar with blockchains through Bitcoin. Bitcoin uses the blockchain as a public ledger to prevent double-spending of the same virtual coin. This same public ledger can be applied to physical hardware products through a bitmark. Products that have been bitmarked can have their provenance tracked back to the factory using the public ledger, thus hampering cloning and warranty fraud – the physical equivalent of double-spending a Bitcoin.

One of my most recent hardware startups, [Chibitronics](https://chibitronics.com) has teamed up with Bitmark to develop an end-to-end solution for Chibitronics’ newest microcontroller product, the [Chibi Chip](https://chibitronics.com/lovetocode).

[![](https://chibitronics.com/wp-content/uploads/2017/05/03_code.jpg)](https://chibitronics.com/lovetocode)

As an [open hardware](https://github.com/chibitronics) business, we welcome people to make their own versions of our product, but we can’t afford to give free Chibi Chips to customers that bought cut-rate clones and then report them as defective for a free upgrade to an authentic unit. We’re also an extremely lean startup, so we can’t afford the personnel to build a full serialization and reverse logistics system from scratch. This is where Bitmark comes in.

Bitmark has developed a turn-key solution for serialization and reverse logistics triage. They issue us bitmarks as lists of unique, six-word phrases. The six-word phrases are less frustrating for users to type in than strings of random characters. We then print the phrases onto labels that are stuck onto the back of each Chibi Chip.

![](https://bunniefoo.com/bunnie/bitmark-example.jpg)

*Bitmark claim code on the back of a Chibi Chip*

We release just enough of these pre-printed labels to the factory to run our authorized production quantities. This allows us to trace a bitmark back to a given production lot. It also prevents [“ghost shifting”](https://www.bunniestudios.com/blog/?p=2037) – that is, authorized factories producing extra bootleg units on a midnight shift that are sold into the market at deep discounts. Bitmark created a website for us where customers can then claim their bitmarks, thus registering their product and making it eligible for warranty service. In the event of an exchange or return, the product’s bitmark is updated to record this event. Then if a product fails to be returned to the factory, it can’t be re-claimed as defective because the blockchain ledger would evidence that bitmark as being mapped to a previously returned product. This allows us to defer the repatriation of the product to the factory. It also enables us to use unverified third parties to handle returned goods, giving us a large range of options to reduce reverse logistics costs.

Bitmark also plans to roll out a site where users can verify the provenance of their bitmarks, so buyers can check if a product’s bitmark is authentic and if it has been previously returned for problems before they buy it. This increases the buyer’s confidence, thus potentially boosting the resale value of used Chibi Chips.

For the cost and convenience of a humble printed label, [Bitmark](https://bitmark.com/) enhances control over our factories, enables production lot traceability, deters cloning, prevents warranty fraud, enhances confidence in the secondary market, and gives us ample options to streamline our reverse logistics.

Of course, the solution isn’t perfect. A printed label can be peeled off one product and stuck on another, so people could potentially just peel labels off good products and resell the labels to users with broken clones looking to upgrade by committing warranty fraud. This scenario could be mitigated by using tamper-resistant labels. And for every label that’s copied by a cloner, there’s one victim who will have trouble getting support on an authentic unit. Also, if users are generally lax about claiming their bitmark codes, it creates an opportunity for labels to be sparsely duplicated in an effort to ghost-shift/clone without being detected; but this can be mitigated with a website update that encouraging customers to immediately register their bitmarks before using the web-based services tied to the product. We also have to exercise care in handling lists of unclaimed phrases because, until a customer registers their bitmark claim phrase in the blockchain, the phrases have value to would-be fraudsters.

But overall, for the cost and convenience, the solution outperforms all the other alternatives I’ve explored to date. And perhaps most importantly for hardware startups like mine that are short on time and long on tasks, printing bitmarks is simple enough for us to implement that it’s hard to justify doing anything else.

*Disclosure: I am a technical advisor and shareholder of Bitmark.*
