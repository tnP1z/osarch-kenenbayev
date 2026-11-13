# Ticket 2 — Kitting out the new office

## 1. What was asked

> From: Daniyar K. (Director)
> Subject: computers for the new office
>
> We are opening the second office and need four computers — accountant,
> sales, web developer, designer. Budget 3,000,000 tenge for everything.
> Everyone says we should just buy the same four machines, is that right?
> Make sure they last a few years. Monitors included.

## 2. What was actually needed

**Where the request is wrong.**

- **“The same four machines” — no.** The four people carry completely different loads. Four identical machines
  would either be too weak for the designer or waste money on the accountant. In my builds the cheapest machine
  costs 527 560 ₸ and the most expensive 1 138 531 ₸ — more than twice as much — and both are the right size for
  their job. Spread evenly, 750 000 ₸ each would pay for power the accountant never uses and leave the designer
  without a colour-accurate monitor.
- **“Budget for everything” hides a large fixed cost.** Four licensed Windows 11 Pro licences cost 411 960 ₸ —
  14 % of the budget before a single component is bought. Keys sold online for about 15 000 ₸ are not licences,
  so I did not count them as an option.
- **“Last a few years” collides with this year's memory prices.** The current platform needs DDR5, which costs
  more than twice as much as DDR4 (304 990 ₸ against 139 990 ₸ for 32 GB). The budget does not cover DDR5 for
  the two machines that need 32 GB, so “lasting” here means *enough memory and storage for four years*, not
  *the newest platform*. See builds.md, last section.

**What the request left out, and what I assumed instead of an answer.**

| Not said | What I would have asked | What I assumed |
|---|---|---|
| Desktops or laptops | “Will anyone work from home or travel?” | Desktops: “monitors included” points that way, and a desktop gives more per tenge |
| What software each person uses | “Which programs, and how heavy are the files?” | Accountant: 1C, Excel, bank client. Sales: CRM in a browser, video calls. Developer: IDE, Docker, Node.js. Designer: Adobe apps and Figma, colour work for print and web |
| How long is “a few years” | “When do you expect to replace them?” | Four years |
| Which Windows edition | “Will the office have a domain or central management?” | Windows 11 **Pro**: BitLocker, Remote Desktop and domain join, which Home lacks |
| Keyboards, mice, headsets, webcams | “Do you have peripherals already?” | Not available; a basic keyboard and mouse for everyone, and a headset and webcam for sales, whose job is calls |
| Who assembles the machines | “Do you have IT staff, or should the shop assemble them?” | I assemble them — so assembly costs time, not money |
| Is 3 000 000 ₸ a hard limit | “Can it move if a requirement does not fit?” | A hard limit; the result is 2 980 351 ₸ |

## 3. What I did

1. Defined the load and the bottleneck for each role (builds.md).
2. Priced every component at one vendor that delivers across Kazakhstan, **shop.kz**, on **14.11.2026**. One item
   came from **itmag.kz**: the colour-accurate monitor was out of stock at shop.kz. Every price is in
   [budget.csv](budget.csv) with its source and date.
3. Checked part compatibility in PCPartPicker (evidence/pcpartpicker-links.md). PCPartPicker does not list
   Kazakhstan prices, so it was used only for compatibility, never for prices.
   It found one real risk: the A520M S2H board in the accountant's and the sales machines supports the
   Ryzen 5 5600GT only from **BIOS F13**. A board from an older batch would not start with this processor. So the
   order has to ask shop.kz to confirm the BIOS version or update it before delivery — the developer's and
   designer's B550 boards raised no such warning.
4. Hit the budget problem, compared DDR5 against DDR4, and moved all four machines to AM4 + DDR4 (builds.md).
5. Priced comparable pre-built machines for question 2.

| Role | Main parts | Total |
|---|---|---|
| Accountant | Ryzen 5 5600GT, 16 GB DDR4, 1 TB NVMe, 24″ FHD IPS | 527 560 ₸ |
| Sales manager | Ryzen 5 5600GT, **32 GB** DDR4, 1 TB NVMe, 24″ FHD IPS, headset, webcam | 638 430 ₸ |
| Web developer | Ryzen 7 5700G (8 cores), 32 GB DDR4 on a 4-slot board, 1 TB NVMe, 27″ QHD IPS | 675 830 ₸ |
| Designer | Ryzen 7 5700X, 32 GB DDR4, 1 TB NVMe, **RTX 3060 12 GB**, **27″ ProArt, 100 % sRGB** | 1 138 531 ₸ |
| **Total** | | **2 980 351 ₸** (19 649 ₸ left) |

The 19 649 ₸ left is less than 1 % of the budget. Prices in Kazakhstan move within weeks, so before ordering the
list has to be re-checked; if something has gone up, the first place to cut is the developer's monitor (a 24″ Full
HD instead of the 27″ QHD saves about 35 000 ₸).

### Question 1. Which machine will be upgraded first, in eighteen months, and why?

**The web developer's — memory, from 32 GB to 64 GB.** His load grows fastest: every new project adds containers,
services and browser tabs that all stay open, and 32 GB is already the floor for that work today. The build is
prepared for it: the B550 board has two free slots, so the upgrade is two more 16 GB DDR4 modules (139 990 ₸ at
today's price) and ten minutes with a screwdriver, with nothing thrown away. Second in line is the designer, also
for memory, for the same reason.

### Question 2. The same four as pre-built machines

Closest pre-built equivalents at shop.kz on 14.11.2026, system units only (no monitors or peripherals):

| Role | Pre-built | Windows included | Price | My build, system unit with Windows 11 Pro |
|---|---|---|---|---|
| Accountant | [HP OmniDesk M02-0016ci](https://shop.kz/offer/kompyuter-hp-omnidesk-desktop-m02-0016ci-e1xt1ea/) — Core i5, 16 GB, **512 GB** SSD | Windows 11 Home SL | 539 990 ₸ | 466 470 ₸ |
| Sales | [HP OmniDesk AI M03-0020ci](https://shop.kz/offer/kompyuter-hp-omnidesk-desktop-ai-m03-0020ci-e1xt3ea/) — Core Ultra 5, 32 GB, 1 TB | Windows 11 Home SL | 799 990 ₸ | 524 660 ₸ |
| Web developer | [HP OmniDesk AI M03-0019ci](https://shop.kz/offer/kompyuter-hp-omnidesk-desktop-ai-m03-0019ci-e1xt2ea/) — Core Ultra 7, 32 GB, 1 TB | Windows 11 Home SL | 889 990 ₸ | 579 850 ₸ |
| Designer | [WW-KZ, shop.kz's own build](https://shop.kz/offer/kompyuter-igrovoy-ww-kz-amd-ryzen-5-7500f-rtx-5060-ti-8-gb-1-tb-ssd-32-gb-ddr-5/) — Ryzen 5 7500F, RTX 5060 Ti **8 GB**, 32 GB DDR5, 1 TB | none — Windows 11 Pro added, +102 990 ₸ | 899 190 + 102 990 = 1 002 180 ₸ | 900 060 ₸ |
| **Total** | | | **3 232 150 ₸** | **2 471 040 ₸** |

The three HP machines come with **Windows 11 Home Single Language**, not Pro: no BitLocker, no Remote Desktop
host, no domain join, and one interface language only. Bringing them to Pro would cost more on top of the prices
above, so the real gap is larger than the table shows.

**What the company gains by buying pre-built:** one 12-month warranty per machine from one company, machines assembled and
tested before delivery, nobody's working day spent on assembly, and newer platforms (Core Ultra, DDR5) with a
longer future.

**What it loses:** about **761 000 ₸** (+31 %) on the system units alone, before upgrading three of them to Pro — with the monitors added, the pre-built
option does not fit the budget at all, and the first thing to go would be the designer's colour-accurate monitor.
It also loses the right proportions: the accountant's pre-built machine has only a 512 GB SSD, below the 1 TB
requirement, and the designer's has 8 GB of video memory instead of 12. Branded office machines also use their
own power supplies and cases, which makes later upgrades harder.

### Question 3. Who complains about performance within a month, and about what?

**The web developer, about memory.** On a day when he runs the containers for two projects, the IDE, a build and
two browsers at once, 32 GB fills up, Windows starts swapping, and the machine freezes for seconds at a time — it
will feel like “the computer is slow”, though the processor will be mostly idle. That complaint is exactly what
question 1 answers: the upgrade path is already built in.

## 4. Evidence

- [budget.csv](budget.csv) — every component: role, model, price in tenge, source URL, date taken
- [builds.md](builds.md) — the four builds: load, bottleneck, trade-off, total; and the DDR4/DDR5 decision
- [evidence/vendor-quotes/](evidence/vendor-quotes/) — a screenshot of every vendor page with the price and the date (14.11.2026); every price matched budget.csv:

  | File | Item | Price |
  |---|---|---|
  | 01.png | Ryzen 5 5600GT | 95 990 ₸ |
  | 02.png | Ryzen 7 5700G | 115 590 ₸ |
  | 03.png | Ryzen 7 5700X | 82 990 ₸ |
  | 04.png | Deepcool AG400 G2 | 11 300 ₸ |
  | 05.png | Deepcool AK400 G2 | 20 100 ₸ |
  | 06.png | Gigabyte A520M S2H | 41 900 ₸ |
  | 07.png | Gigabyte B550M GAMING X WIFI6 | 74 700 ₸ |
  | 08.png | Lexar 8 GB DDR4 | 40 900 ₸ |
  | 09.png | Geil Pristine 32 GB DDR4 kit | 139 990 ₸ |
  | 10.png | WD Blue SN5100 1 TB | 96 990 ₸ |
  | 11.png | Samsung 990 1 TB | 121 600 ₸ |
  | 12.png | GamerStorm PF600D V2 | 26 300 ₸ |
  | 13.png | PCCooler C3 B310 BK | 11 990 ₸ |
  | 14.png | PCCooler MA100 MESH BK | 17 990 ₸ |
  | 15.png | Windows 11 Pro OEM | 102 990 ₸ |
  | 16.png | Philips 24E1N1200A | 52 100 ₸ |
  | 17.png | ASUS ProArt PA278CV (itmag.kz) | 229 481 ₸ |
  | 18.png | Logitech MK120 | 8 990 ₸ |
  | 19.png | Logitech 960 headset | 12 690 ₸ |
  | 20.png | Logitech Brio 300 | 39 990 ₸ |
  | 21.png | WD Green SN3000 1 TB | 94 200 ₸ |
  | 22.png | Palit RTX 3060 12 GB | 289 200 ₸ |
  | 23.png | GamerStorm PQ650G | 50 500 ₸ |
  | 24.png | AOC Q27B36X | 86 990 ₸ |
- [evidence/pcpartpicker-links.md](evidence/pcpartpicker-links.md) — the four builds in PCPartPicker: the parts are compatible, with one BIOS warning and the substitutes listed
- [evidence/peer-check.md](evidence/peer-check.md) — another person checked prices and compatibility from my documents

## 5. What I would do differently

I would price every component at **two** vendors, not one. Using only shop.kz kept the evidence simple, but I
found that the colour-accurate monitor was out of stock only when I got to it, and a second vendor might have been
cheaper on the expensive memory kits — the line where the whole budget was decided. I would also add an
uninterruptible power supply for the accountant's machine: a power cut during a 1C operation can damage the
database, and that risk costs more than the UPS.
