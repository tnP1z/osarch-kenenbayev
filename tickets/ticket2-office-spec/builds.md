# Four builds for the second office

All prices in tenge, taken on **14.11.2026**, almost all from one vendor, **shop.kz** (Белый Ветер, delivers across
Kazakhstan); the designer's monitor is from **itmag.kz** (in stock, delivers across Kazakhstan). Line items and
links: [budget.csv](budget.csv).

Every machine meets the hard requirements: at least 16 GB of memory, a 1 TB SSD, a licensed Windows 11 Pro and a
monitor. They also share one platform — AMD AM4 with DDR4 memory — for a reason explained at the end.

| Role | Total | Rows in budget.csv |
|---|---|---|
| Accountant | 527 560 ₸ | 2–11 |
| Sales manager | 638 430 ₸ | 12–23 |
| Web developer | 675 830 ₸ | 24–33 |
| Designer | 1 138 531 ₸ | 34–44 |
| **All four** | **2 980 351 ₸** | **19 649 ₸ under the 3 000 000 ₸ budget** |

---

## Build 1 — Accountant

**Load:** 1C:Enterprise, Excel, a browser with the bank client and the tax portal, all day, rarely more than two
or three of them at once. Nothing here is heavy; what matters is that the machine never loses the 1C database
and never stops.

**Bottleneck:** none of the components is under real pressure. The closest thing to a bottleneck is storage
response for the 1C database, which an NVMe SSD removes.

**Trade-off:** the cheapest build of the four. A 6-core Ryzen 5 5600GT with integrated graphics (no video card),
16 GB as two 8 GB modules so the memory runs in dual channel, and a 24-inch Full HD monitor. The A520 board has
only two memory slots and runs the SSD at PCIe 3.0 instead of 4.0 — irrelevant for this load, and it saved about
33 000 ₸ against a B550 board.

**Total:** 527 560 ₸ (budget.csv, rows 2–11)

## Build 2 — Sales manager

**Load:** a CRM in the browser with dozens of tabs, Excel, messengers, and several hours of video calls a day —
often all at once.

**Bottleneck:** memory, and the call hardware. A browser with many tabs plus a video call easily fills 16 GB,
and then Windows starts swapping in the middle of a call with a customer. The processor is mostly idle.

**Trade-off:** the same processor and board as the accountant, but **32 GB** of memory, plus a USB headset and a
Full HD webcam. A desktop has no camera or microphone of its own, so without these two items the machine cannot do
its main job. The money for them came from not buying a stronger
processor this role would never use.

**Total:** 638 430 ₸ (budget.csv, rows 12–23)

## Build 3 — Web developer

**Load:** an IDE, a local web server, Node.js builds, Docker containers with a database, and several browsers for
testing — at the same time, every day.

**Bottleneck:** memory first, processor cores second. Containers, the IDE and the browsers each hold memory for as
long as they run; builds use every core they are given.

**Trade-off:** an 8-core Ryzen 7 5700G (integrated graphics, so no video card) and 32 GB of memory, on a **B550**
board instead of A520: it has **four** memory slots, so the planned upgrade to 64 GB means adding two modules
instead of replacing these. A 27-inch QHD monitor, because more code and a browser fit side by side. To pay for
the B550 board I took a WD Blue SSD instead of a Samsung 990 (−24 610 ₸) — for a developer the difference is not
noticeable.

**Total:** 675 830 ₸ (budget.csv, rows 24–33)

## Build 4 — Designer

**Load:** Photoshop, Illustrator and Figma with large files, some video in Premiere, and colour work that ends up
in print and on the company's website.

**Bottleneck:** two of them — the **monitor** and the **graphics card**. If the screen shows colours wrongly, every
other component is wasted: the designer cannot see what the customer will see. Adobe applications also use the
video card for filters, previews and export.

**Trade-off:** almost 40 % of the whole budget goes to this one machine, and most of that to the two bottlenecks:
an ASUS ProArt PA278CV (100 % sRGB, factory calibrated, ΔE < 2) and a GeForce RTX 3060 with **12 GB** of video
memory. I chose the older RTX 3060 12 GB over the newer RTX 5060 8 GB (324 990 ₸): it is cheaper and has more
video memory, which matters more for large Photoshop files than raw speed. The processor is the Ryzen 7 5700X,
which has no integrated graphics — not needed with a video card — and was the cheapest 8-core chip available.
The B550 board gives the card a PCIe 4.0 slot and leaves two memory slots free.

**Total:** 1 138 531 ₸ (budget.csv, rows 34–44)

---

## Why all four are AM4 with DDR4

This is the biggest decision in the document, and it was forced by memory prices in November 2026:

| Memory | Price at shop.kz, 14.11.2026 |
|---|---|
| 32 GB DDR4-3200 (2 × 16 GB) | **139 990 ₸** |
| 32 GB DDR5-6000 (2 × 16 GB) | **304 990 ₸** |

DDR5 costs more than twice as much. My first version put the developer and the designer on the current AM5
platform with DDR5 (Ryzen 7 9700X); it came to **3 421 861 ₸ — 421 861 ₸ over budget**. Moving all four machines
to AM4 with DDR4 brought the total under 3 000 000 ₸ without dropping a single hard requirement.

**What it costs:** AM4 is the previous platform. In four years nobody will put a new processor into these boards,
and DDR4 will not move to a future platform. Memory and storage can still be upgraded; the processor cannot be
upgraded in any meaningful way. One shared platform also has an advantage for a small company: the same drivers,
the same spare parts, and any module can move between three of the four machines.
