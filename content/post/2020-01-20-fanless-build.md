+++

date = "2020-01-20T00:00:00+00:00"
description = "A fanless desktop computer build for gaming at 1200p."
title = "An i7-8700 and GTX 1060 based fanless gaming desktop"

+++

**TL;DR**:
As a silent computing ethusiast, I tell my story of delidding an Intel i7-8700 CPU, breaking it, delidding another one and then building a fanless gaming desktop with a GeForce GTX 1060 6GB in an HDPLEX H5 case.
The motherboard for the system is ASRock Fatal1ty Z370 Gaming-ITX/ac and Seasonic PRIME Titanium 600 is the power supply.

<!--more-->

I believe that a generic desktop PC build can be described by three independent qualities: physical size, performance and noise, which are often impractical or hard to decouple.
Want more performance?
Sure, but you might need a bigger case to fit more powerful hardware and more fans to circulate more air to cool the components down, thus causing more noise.
Price usually follows the other qualities, in the sense that the more extreme you want to be in any of the categories -- smaller size, more performance or less noise -- the more you'll have to pay.
Most often, in the realm of consumer hardware and sane prices, one can pick only one of the qualities for the PC to excel at.
Reasonably, most people go for performance, so an average gaming PC would be a mini or a mid tower, with a mid-range CPU, a high-end graphics card and a few fans on the case to provide a steady flow of fresh air.
Depending on the budget, an HDD is included in such a build for more storage space compared to an equally priced SSD and the power supply is very likely a unit with a fan for active cooling during load.
The count of components that make noise in such a mid-range build is easily more than four: CPU fan, graphics card fan or two, PSU fan, one or more case fans and an HDD.

I did not want my PC to be an average PC!
I wanted to try to maximize any of the three qualities, with the only strict condition of at least having enough performance to occasionally fluidly play a AAA game at 1920 by 1200 pixel resolution with at least high in-game video quality settings. 

Small computers exist -- RaspberryPi is one of the most popular small computers -- but going to that extreme would come at a significant performance penalty, therefore the only strict requirement would not be satisfied, so I opted not to chase the size extreme.

Maximizing performance was an option, but almost an endless money and time pit.
There is always another thing one can do for better performance.
How many graphics cards would be enough? Three? Four?
I would surely have to get a dual socket motherboard and overclock the CPUs for maximum performance.
Why stop at air cooling when custom liquid cooling loops give lower temperatures?
What kind of RAID setup should be used for maximum performance and how much storage space is enough?
Chasing the general performance extreme is difficult in practice, so I opted not to go after that one either.

We are left with the silence extreme.
That goal is very easy to pin down -- the PC should not be making any sound -- and non-exotic components that can satisfy that are available in retail.
The question to be answered was then, how much performance can be had in components so powerful, that they can not exceed the capacity of a passive cooling solution.

## Case and cooler selection

The build was driven by the case and/or cooler selection, because heat disippation is usually the limiting factor in passive PC builds.
My first idea was simply using a mid or a full tower case with aftermarket passive CPU and GPU coolers.
A monstrous CPU cooler that can handle a lot of watts is the [Nofan CR-95C](https://web.archive.org/web/20190810142642/http://www.nofancomputer.com/eng/products/CR-95C.php) and is the default choice for many fanless builds.
There are fewer products on the market for passive graphics card cooling, so the only choice was the [Arctic Accelero S3](https://web.archive.org/web/20190810143618/https://www.arctic.ac/us_en/accelero-s3.html) which has reached end of life.
An alternative would be mounting a CPU cooler on the graphics card, but that solution is hacky and comes without any guarantees of a successful fit.
Disheartened, I started looking for PC cases which do the cooling, by connecting to the CPU and the GPU with heatpipes.

Sadly for me, the passive cooling market is a niche, but there are a handful of companies offering what I wanted.
Mid-2018, when I was looking to make my purchase, the three companies that were in the business of making passively cooled cases and had them readily available were Akasa, Streacom and HDPLEX.
Akasa focuses on NUCs and I couldn't achieve adequate gaming performance with a fanless NUC.
Streacom and HDPLEX seem to have a lot of HTPC cases, where the PSU would have to be external and where there was no room for a graphics card, so those are not applicable either.
The only rare case with enough room to accommodate a graphics card and an internal PSU was the [HDPLEX H5](https://web.archive.org/web/20190810165721/https://www.hdplex.com/hdplex-h5-fanless-computer-case.html), so it was more or less my only choice.
An alternative was the [Streacom DB4](https://web.archive.org/web/20190810170047/https://streacom.com/products/db4-fanless-chassis/), but the HDPLEX H5 has nominally more powerful cooling, so I went with it.

Worth a mention is the [Calyos NSG S0](https://web.archive.org/web/20190810170234/https://www.kickstarter.com/projects/1489140137/nsg-s0-worlds-first-fanless-chassis-for-high-perfo/) that was still up for preorder when I was looking to buy.
However, it turned out that Calyos scammed people who preordered or mismanaged the funds and ended up without a working product, so I was lucky to dodge a bullet there.
Lesson learned about preorders!
According to a few Reddit threads, part of the Calyos team went on to found MonsterLabo which at the time of writing (late 2019) offers an interesting fanless case, [The First](https://web.archive.org/web/20190810170555/https://www.monsterlabo.com/the-first-1) and just the cooler used in the The First, called [The Heart](https://web.archive.org/web/20191201214100/https://www.monsterlabo.com/page-d-articles/the-heart).
Based on the easily discoverable reviews and as the name implies, it really seems like a company's first attempt at a PC case.
Despite being nominally more capable than the HDPLEX H5, cooling-wise, I would love to see it having proper back panel and front panel IO and feeling like a finished product without pass-through cables.

## Component selection

Once the HDPLEX H5 case was picked, the next step was selecting the most powerful components that fit inside, while still not being too powerful to cool.
In other words, I was looking for the most energy efficient components.

### PSU

Since I intended to have gaming loads on the PSU, I had to select a very efficient unit, such that not a lot of energy is wasted into heat.
Low efficiency at idle loads is not a problem, because the total power turned into heat at those loads is still small and handled easily.
I had a rough idea of the maximum power the system could draw as it is limited by how much of that power can be dissipated by the case: nominally 95W CPU and 95W GPU, so a 190W total plus a few dozen watts for the rest of the system, and it was obvious that even the lowest power PSU would be enough.
ATX units, as opposed to the smaller SFX units, are generally more efficient, so I went with the most efficient consumer power supply available on the market at the time, the [600W Seasonic PRIME Titanium Fanless](https://web.archive.org/web/20190810173455/https://seasonic.com/prime-titanium-fanless).
Even if that unit wasn't fanless, it would have likely used a fanless mode for low power loads, as the passive coolers in it could successfully dissipate that small amount of heat.
Therefore, such a high-end pick is an overkill from a power standpoint, but could be greener overall, depending on the exact load and the efficiency curve of the unit.

### Motherboard

The choice of an ATX power supply limited the size of the motherboard that would fit in the case to mITX, according to the HDPLEX H5 specifications.
I did not have rigorous criteria for the motherboard selection or features I wanted, other than going for an Intel-based mITX platform, because Intel CPUs were more energy efficient at the time than AMD CPUs.
Looking at different options, I went for the one having a lot of voltage regulators, because each of them gets less heat the more of them there are, and having a beefy-looking passive cooler on top of the motherboard chips.
I selected the [ASRock Fatal1ty Z370 Gaming-ITX/ac](https://web.archive.org/web/20190623200756/https://www.asrock.com/mb/Intel/Fatal1ty%20Z370%20Gaming-ITXac/).
I might have selected a different motherboard had the reviews included more information that I was looking for across a wider selection of products, but, retrospectively, the choice was not bad.
Not necessarily the best, but not bad by any means.

### CPU

Overclocking was out of the question, because CPUs that fit the available power at nominal performance were readily available.
However, I still wanted the highest-end CPU I could get, so I went for the Intel i7-8700 (non-K).
I planned to delid the CPU and keep it undervolted for less power consumption, but in hindsight, I should have gone with the K CPU version, due to potentially better binning, thus enabling more undervolting.
Furthermore, I believe it is better picking a more powerful CPU than the case can handle and then lowering its voltage and clock to achieve maximum performance without thermal throttling, as opposed to buying a less powerful CPU and leaving unused thermal headroom.
Of course, had the CPU cost been an issues, the latter option might have been the preferred one.

### Graphics card

Same ideas that apply to the CPU selection apply to the graphics card selection too.
I wanted to pick the most powerful card that fit in the thermal budget with a preferred overshoot rather than an undershoot, when it comes to performance and power consumption.
The 6GB GTX 1060 was a clear winner and when picking the manufacturer, I wanted to get one with the best VRM design.
I could not find too much information and didn't know whether the Eastern European market has the same products as Central or Western Europe, so other factors were size and price as well.
Finally, I picked a reasonably priced entry-level Gigabyte GTX 1060 and planned an undervolt.
I did not go for the Mini version as I was afraid the smaller design would have less VRMs, nor did I go for the biggest one as I didn't need the extra fans on the stock cooler and I could not fit it in the case.

### SSD

Power efficiency and supporting NVMe were the only characteristics that I cared about when picking the SSD.
I picked the most power efficient NVMe SSD according to the [The SSD Review](https://web.archive.org/web/20190810181020/http://www.thessdreview.com/featured/mydigitalssd-sbx-m-2-nvme-ssd-review/5/) at the time, the [512GB MyDigitalSSD SBX](https://web.archive.org/web/20190810181146/https://mydigitalssd.com/pcie-m2-ngff-ssd.php#80mm-sbx-m2).


## Delidding, damaging and delidding the CPU again

Various review and overclocking websites revealed that by default even high-end 8th gen Core i7 CPUs used thermal paste, not solder, for interfacing the die with the heatspreader (the lid).
Using solder or a better-than-default thermal interface would increase heat transfer and provide better cooling, so a lot of enthusiasts started delidding the CPUs and putting the cooler directly against the die or using liquid metal thermal interface between the die and the heatspreader.

I didn't want to buy one of the purpose-made delidding tools, so I have used a monkey wrench to take the heatspreader off.

![Components used in the delidding process: the CPU in the center, the blade to clean leftovers from the glue and the monkey wrench.](https://luka.strizic.info/fig-fanless-desktop/delidding-01.jpg)

In spite of a successful trial run with an i3-8100 CPU, I did not properly orient the i7-8700 CPU so I managed to split a few layers of the PCB where the wrench was touching it, just as the heatspreader came off. The next image shows the proper orientation. The wrong one was, as you may imagine, 90 degrees off in the plane of the CPu.

![Proper delidding alignment with a monkey wrench.](https://luka.strizic.info/fig-fanless-desktop/delidding-02.jpg)

You can imagine the horror on my face when I realized what I had just done, but I was very suprised to find out that the CPU wasn't entirely destroyed and actually still worked!
The damage is clearly visible in the following images.

![CPU damage from improper delidding.](https://luka.strizic.info/fig-fanless-desktop/delidding-03.jpg)

![CPU damage from improper delidding.](https://luka.strizic.info/fig-fanless-desktop/delidding-04.jpg)

![CPU damage from improper delidding.](https://luka.strizic.info/fig-fanless-desktop/delidding-05.jpg)

![CPU damage from improper delidding.](https://luka.strizic.info/fig-fanless-desktop/delidding-06.jpg)

Despite the damage, the CPU had nominal performance and thermals.
All the features that I tested worked ok, with the exception that one memory channel was unavailable and the RAM in the corresponding memory slot on the motherboard was not seen by the OS or the BIOS.
According to the CPU pinout that I found on WikiChip, the pins directly beneath the split part of the layers are DIMM 1 pins, so that exact DIMM slot not working was not too big big a suprise.
[This Coffee Lake pin diagram](https://web.archive.org/web/20191218221353/https://en.wikichip.org/w/images/a/a4/coffee_lake_pin_diagram.png) is quite low definition, making it hard to read which pin is connected to what, but [this Skylake / Kaby Lake pin diagram](https://web.archive.org/web/20191218221708/https://en.wikichip.org/w/images/9/9f/skylake_pin_diagram.png) shares much of the same connections and is higher definition.
The pins that I damaged are at the upper edge in the diagrams.

My motherboard had only two DIMM slots and I was not ready to give one up due to a faulty CPU.
I sold the faulty CPU that was otherwise working fine for a really low price with full disclosure of the mishap and bought another i7-8700 with an important delidding lesson kept in mind.

Second delidding attempt on the fresh i7-8700 with the monkey wrench went perfectly fine, because the orientation was correct this time.

![A delidded Core i7-8700 CPU.](https://luka.strizic.info/fig-fanless-desktop/delidding-07.jpg)

Then, [Thermal Grizzly Conductonaut](https://web.archive.org/web/20190810185355/http://thermal-grizzly.com/en/products/26-conductonaut-en) was applied and the heatspreader put back with generic high-heat silicone used as the glue. The CPU PCB exposes four pins under the lid, which are not connected to anything according to a quick Google search. Just in case, I put a heat-resistant tape over them. The images show quite a messy re-lidding process, but I cleaned it up a bit before putting the lid back on and removed the excesses liquid metal.

![Thermal grizzly Conductonaut applied to the CPU die and silicone applied to the PCB.](https://luka.strizic.info/fig-fanless-desktop/delidding-08.jpg)

![Re-lidded Core i7-8700.](https://luka.strizic.info/fig-fanless-desktop/delidding-09.jpg)

## Assembly

The case packaging is quite compact as can be seen in the following images.

![Packaging of the HDPLEX H5.](https://luka.strizic.info/fig-fanless-desktop/case-01.jpg)

![Packaging of the HDPLEX H5.](https://luka.strizic.info/fig-fanless-desktop/case-02.jpg)

![Packaging of the HDPLEX H5.](https://luka.strizic.info/fig-fanless-desktop/case-03.jpg)

![Packaging of the HDPLEX H5.](https://luka.strizic.info/fig-fanless-desktop/case-04.jpg)

![Packaging of the HDPLEX H5.](https://luka.strizic.info/fig-fanless-desktop/case-05.jpg)

Sadly, heatpipes for the GPU have to be bought separately.
The PCIe riser cable, also bought separately, seems like a high-quality unit, whomever it is sourced from.

![Packaging of the GPU unit for the HDPLEX H5.](https://luka.strizic.info/fig-fanless-desktop/case-06.jpg)

![Packaging of the GPU unit for the HDPLEX H5.](https://luka.strizic.info/fig-fanless-desktop/case-07.jpg)

![Packaging of the GPU unit for the HDPLEX H5.](https://luka.strizic.info/fig-fanless-desktop/case-08.jpg)

Assembly was relatively painless but quite involved.
It was clear at all points what I needed to do next, but it took a while to put all the pieces in their place and make sure that the heatpipes are properly aligned and well-pasted.
I am not mentioning this as a negative thing, due to what the HDPLEX is trying to accomplish here.

The cooling block for the CPU came together like a sandwich.
First, the lower part of the block gets mounted, like any other CPU cooler.

![Assembly of the CPU heatsink.](https://luka.strizic.info/fig-fanless-desktop/assembly-01.jpg)

Second, the right places where the heatpipes go need to be pasted both on the case wall and the CPU block.
I'm not sure that the small ball on a stick used for pasting helps too much, but I gave it a shot.

![Assembly of the CPU heatsink.](https://luka.strizic.info/fig-fanless-desktop/assembly-02.jpg)

![Assembly of the CPU heatsink.](https://luka.strizic.info/fig-fanless-desktop/assembly-03.jpg)

![Assembly of the CPU heatsink.](https://luka.strizic.info/fig-fanless-desktop/assembly-04.jpg)

Finally, the small radiator goes on top of the attached heatpipes to keep them in place.
I've used way too much thermal paste for the upper and lower parts of the sandwich.
Most of it can be seen flowing out in the half-assembled case.

![Assembly of the CPU heatsink.](https://luka.strizic.info/fig-fanless-desktop/assembly-05.jpg)

![Assembled CPU heatsink and the back of the half-assembled case.](https://luka.strizic.info/fig-fanless-desktop/assembly-06.jpg)

Next image shows the same thing, but from the front, where you can also see the backside of the power button on the front panel.

![Assembled CPU heatsink, the front of the half-assembled case and the power button.](https://luka.strizic.info/fig-fanless-desktop/assembly-07.jpg)

Once more, same thing, but from the top.

![Assembled CPU heatsink and half-assembled case from the top.](https://luka.strizic.info/fig-fanless-desktop/assembly-08.jpg)

Same process was applied for the GPU portion, so I don't have detailed shots of that, just the position of the GPU with its block on.
The motherboard and the PSU can be seen to the left.

![Graphics card with its block on placed in the case.](https://luka.strizic.info/fig-fanless-desktop/assembly-09.jpg)

Just behind the graphics card are the messy wires and cables which can't be put anywhere else.

![Wires and cables in the case which can't be hidden anywhere.](https://luka.strizic.info/fig-fanless-desktop/assembly-10.jpg)

Once the heatpipes were connected to the walls and the respective components, the case came together.
I made sure to leave some room for convection through the holes in the top cover, so my display was put on four stands.
That's when I also realized that I rotated the power button by 90 degrees.
My guess is that it is supposed to spell H as in HDPLEX.

![Assembled case and a display on top of it on my desk.](https://luka.strizic.info/fig-fanless-desktop/assembly-11.jpg)


I have used [Thermal Grizzly Kryonaut](https://web.archive.org/web/20190810200005/http://thermal-grizzly.com/en/products/16-kryonaut-en) between the heatpipes and other cooling elements.

The case will achieve its main goal: it will cool the CPU and the GPU passively up to 95W each, just perhaps not under long-term maximum load.
However, I do have a list of grievances:

 - Cable management was the hardest part of the assembly (isn't it always?) and that's the weakest point of the whole experience in my opinion.
   The case doesn't leave any room for cable routing and feels quite low-end in that regard.
   Making the case taller with proper routing options beneath the motherboard would be a huge plus, I believe.
 - Having more openings on top for additional convection could perhaps lower the temperature by a degree or two.
 - Leaving more horizontal space around USB sockets for wider USB plugs would be useful.
 - More space around the 3.5 mm audio socket would make it much easier to plug devices in and out, which is quite troublesome now or downright impossible for people with larger fingers.
 - The USB 2.0 and USB 3.0 sockets are oriented differently. USB 2.0 is upside down.
 - I find ATX power supply mounting unconvincing. It doesn't seem secure to me and it worries me a bit when transporting the case.
 - The power button requires manual assembly as well as manual tuning of the travel depth in four places. Also, it can be easily shorted on the other side, inside the case.
 - One of the edges of the top panel looks chipped away.

None of the things that I listed affect performance, which I am quite happy with, but overall it shows that the attention to detail just isn't there or that I'm not the target audience.
I believe that if HDPLEX took these things into account and improved them in the next iteration, they'd make a really awesome case attractive to slightly broader audiences.

## Undervolting

I've used the Intel Xtreme Tuning Utility to change the voltage offset on the CPU and would test the stability with Prime95.
The lowest stable offset that I've found is -50mV on my sample of the CPU, without sacrificing any clock.
I have left the load-line calibration at level 5, which compensates for voltage droop the least.
Testing with level 1 showed that a lot more voltage is applied with the same VID at load, which is the opposite of what I want.
To make the change permanent, I have entered it into UEFI.
Furthermore, I have had to limit the long term CPU power to 90 watts to make the CPU not throttle with Prime95 alone.

The clock and voltage of the GPU can not be controlled directly like the CPU's, with an offset, and varies a lot depending on the thermal headroom.
I have used MSI Afterburner to overclock the GPU by an additional 150 MHz, but have also lowered the Power Limit to 95%, so that in practice same clocks are achieved with less power, thus effectively undervolting the GPU.
I've left the memory clock the same, as there is no active or passive cooling on the memory chips.
I did not have a lot of success in modifying the frequency-voltage curve, as the GUI is not friendly and the settings would have a hard time sticking across reboots because they would stick to specific values.
To find those values, that are stable under load, I've tested with FurMark.

## Thermal performance

Long term full load testing with Prime95 and FurMark inevitably leads to thermal throttling.
Long term gaming loads, naturally, do not cause any thermal throttling.
It should be noted that the case gets very hot to the touch during long term load.
I don't think that posting hard numbers here makes sense, as I can't perform the tests in a controlled environment.
A ballpark is that during multi-hour gaming loads the CPU and GPU would go up to 80 degrees Celsius and during synthetic load they would reach 95 degrees Celsisus in around 30 minutes.

![Prime95 and FurMark testing.](https://luka.strizic.info/fig-fanless-desktop/thermal-01.jpg)

During a multi-hour testing session the WiFi card on the motherboard died.
It could not be detected in Windows nor Linux, as if it was disconnected.
I have tried using it in other machines where it did not work either.
I have tried using another WiFi card attached to the motherboard and it did work, which clearly determined the culprit.
When I contacted ASRock support to make sure something else wasn't the issues, the agent was very helpful and promptly sent me a new WiFi card by mail, which they did not owe me at all.

![New WiFi card that ASRock sent me.](https://luka.strizic.info/fig-fanless-desktop/wifi-01.jpg)

The WiFi card is connected to the motherboard by an M.2 slot, so it is trivial to change it.

![New WiFi card next to the old one still plugged into the motherboard.](https://luka.strizic.info/fig-fanless-desktop/wifi-02.jpg)


## Alternatives and future

Acquiring and assembling all the components takes time and effort, and many things can go wrong.
I had a lot of fun and learned a lot, but if someone wants a plug-and-play fanless solution, this definitely isn't the way to go.
In that case, [Compulab's Airtop](https://web.archive.org/web/20190810201308/https://fit-iot.com/web/) is the way to go.
Their machines use higher performance components than what I selected for my build, but they cool them successfully under full load with their patented case.

At time of writing this post, Zen 2 based Ryzen processors were recently released and are more energy efficient than what Intel has to offer.
Zen 2 CPUs would, thus, be a better choice for a fanless computer.
My only current concern about the Zen 2 platform is the fan on the X570 chipset on many motherboards due to the high-power PCIe 4.0 capability.
Over time, power use of the chipset will surely be decreased and using a non-X570 motherboard with passive cooling is also a viable option.

Also at the time of writing, the [Turemetal UP10](https://web.archive.org/save/https://www.fullysilentpcs.com/product/turemetal-up10/) was just released and it seems to be able to handle significantly more power than the HDPLEX H5 -- double the power easily, according to published videos.
I am considering switching to that case and leveling up my desktop, but will wait for a decent review first.
