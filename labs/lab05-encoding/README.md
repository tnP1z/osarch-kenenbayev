# Lab 5 — Encoding forensics

Pair: none    Driver first half: Kenenbayev Kadirzhan
Machine: ASUS Zephyrus G14 GA403WR (host, Windows 11); `lab01-ubuntu` guest (Ubuntu 26.04.1 LTS) in VMware
Date: 12.11.2026

The diagnosis and the conversion were done in the Ubuntu guest; Excel and LibreOffice Calc ran on the Windows host.
The original was never opened in an editor: for viewing it in Excel and Calc I opened a copy, `view-broken.csv`.

## Diagnosis (before the fix)

The checksum of the original, taken before anything else and again after the fix — identical, so the original is untouched:

```
919027f1f8c71d85cba3f12aad0f6c927d4e0450cfb2a2d63378a69f14cab270  staff-broken.csv
```

**file says** (evidence/file-before.txt):

```
staff-broken.csv: Non-ISO extended-ASCII text, with CRLF line terminators
```

So: plain text, not Unicode, with bytes above 7F — some single-byte code page. `file` cannot say which one.

**First 16 bytes** (evidence/hexdump-before.txt):

```
00000000: d2e0 e12e b93b d4c8 ce3b c4ee ebe6 edee  .....;...;......
```

| Offset | Bytes | What they are |
|---|---|---|
| 00 | `D2` | Т — first character straight away: **no BOM** (a UTF-8 BOM would be `EF BB BF`) |
| 01–02 | `E0 E1` | а, б — Cyrillic letters, **one byte each**: a single-byte code page, not UTF-8 (which would be `D0 xx` pairs) |
| 03 | `2E` | `.` in ASCII |
| 04 | `B9` | № |
| 05 | `3B` | `;` — the separator is a **semicolon**, not a comma |
| 06–08 | `D4 C8 CE` | Ф, И, О — second column header |
| 09 | `3B` | `;` |
| 0A–0F | `C4 EE EB E6 ED EE` | Д, о, л, ж, н, о — start of the third header, “Должность” |

The line ending is not inside the first 16 bytes: the header line ends at offset `2C–2D` with `0D 0A`
(`xxd staff-broken.csv | head -20`, line `00000020`).

**Encoding: RK1048** (the Kazakh Cyrillic code page).
Evidence, in three steps:

1. It is not UTF-8: `iconv -f UTF-8 -t UTF-8 staff-broken.csv` stops with *illegal input sequence at position 0*.
2. I decoded the file with each single-byte candidate (evidence/candidates.txt). CP866 and KOI8-R give nonsense
   even for Russian words. CP1251, RK1048 and PT154 all give correct Russian — the Kazakh letters decide:

   | Candidate | Row 1001 | Row 1003 |
   |---|---|---|
   | CP1251 | **Ќ**асымова **Ј**лия Серік**ќ**ызы | Н**ў**рланов **Є**алымжан |
   | PT154 | Қасымова **Ј**лия Серікқызы | Н**ў**рланов **Ә**алымжан |
   | **RK1048** | **Қасымова Әлия Серікқызы** | **Нұрланов Ғалымжан Ерланұлы** |

   For example, byte `8D` is Қ in RK1048 but Ќ in CP1251, and `A3` is Ә in RK1048 but Ј in CP1251.
   PT154 differs from RK1048 on 14 of the 26 lines.
3. `iconv -f RK1048 -t UTF-8 staff-broken.csv > /dev/null && echo ok` prints `ok` — the whole file decodes, and
   reading it through I found every Kazakh letter correct: Ә, Ғ, Қ, Ң, Ө, Ұ, Ү, Һ, І.

**Line ending: CRLF (Windows)**, on every line.
`tr -cd '\r' < staff-broken.csv | wc -c` = **26** CR bytes, `wc -l staff-broken.csv` = **26** LF bytes,
26 lines — equal, so every line ends in `0D 0A`. `cat -A` shows `^M$` at the end of each line.

**BOM: absent** — the file starts with `D2`.

## What each application was doing wrong

**Excel** (double-click, evidence/view-broken.png): a CSV without a BOM is read in the Windows ANSI code page,
which on this Russian-locale machine is **CP1251**. CP1251 and RK1048 share all the Russian letters, so the Russian
names look right and only the Kazakh letters break: “Ќасымова”, “Јбенова”, “Нўрланов”. The columns are split
because the Windows list separator here is `;`. (The `#######` in the date column is only a too-narrow column.)

**LibreOffice Calc** (evidence/view-broken-libre.png): its import dialog auto-detected **Кириллица
(Windows-1251)** — the same wrong guess as Excel, for the same reason: CP1251 is the most common Cyrillic code page
and Calc has no way to know the file is Kazakh. It detected the semicolon separator correctly. On this machine Excel
and Calc agree; on a non-Russian Windows, Excel would use CP1252 instead and show Latin garbage.

**cat** on Ubuntu: the terminal decodes everything as **UTF-8**. A single byte between `80` and `FF` is not valid
UTF-8 on its own (iconv confirmed this at position 0), so `cat` cannot show these bytes as letters at all.

## Fix

Script: [scripts/fix.sh](scripts/fix.sh), run from `~/labs/lab05-encoding` with `bash scripts/fix.sh`.

```bash
{
  printf '\xEF\xBB\xBF'
  iconv -f RK1048 -t UTF-8 evidence/staff-broken.csv | sed 's/\r$//'
} > evidence/staff-fixed.csv
```

Changed: **encoding RK1048 → UTF-8; line ending CRLF → LF; BOM added.**

**Why LF:** Excel and Calc read LF without any problem, and a script reading the file does not get a hidden `\r` at
the end of the last column.

**The BOM decision:**

| Reader | Without BOM | With BOM |
|---|---|---|
| Excel, double-click | reads it as CP1251 → UTF-8 bytes turn into garbage | reads it as UTF-8 → correct |
| LibreOffice Calc | usually guesses UTF-8, but it is a guess | detects UTF-8 for certain |
| `cat` in a UTF-8 terminal | correct | correct; the BOM is invisible, though my terminal drew a stray gap after the first letter |
| A script reading the first header | `Таб.№` | `\uFEFFТаб.№` — the first header carries an invisible character and must be stripped |

I kept the BOM: the requirement is that the file opens correctly everywhere, and Excel on a double-click is the
reader that cannot open UTF-8 without it. The cost is that any script reading the header has to strip the BOM first.

## Verification

**file after** (evidence/file-after.txt):

```
staff-fixed.csv: Unicode text, UTF-8 (with BOM) text
staff-fixed.csv: Unicode text, UTF-8 (with BOM) text

```

First bytes of the fixed file: `EF BB BF` (one BOM, not two), then `D0 A2` = Т as a UTF-8 pair.

Opened in: **Excel for Microsoft 365** (version 2610, build 16.0.20527.20150, 64-bit), **LibreOffice Calc 26.8.2.2**,
**cat on Ubuntu 26.04.1 LTS** — every name correct, Kazakh letters included, columns split — evidence/opens-correctly.png.

## What did not work the first time

1. **I ran a Windows command inside Ubuntu.** I typed the `scp.exe` command for copying `fix.sh` into the window
   where I was already connected to Ubuntu over SSH. Bash answered `C:WindowsSystem32OpenSSHscp.exe: command not
   found` — it does not know Windows paths and swallowed the backslashes. The prompt `studentk@lab01:~$` showed I
   was in Ubuntu. Fix: kept the SSH window for Ubuntu commands and opened a second PowerShell window for commands
   that run on Windows.
2. **A typo in the terminal:** one `cat` was mistyped and gave `cat: command not found` (visible in
   opens-correctly.png); I retyped it.

## Evidence

- [evidence/staff-broken.csv](evidence/staff-broken.csv) — the original from Canvas, unchanged (checksum above)
- [evidence/staff-fixed.csv](evidence/staff-fixed.csv) — the converted file: UTF-8 with BOM, LF
- [evidence/file-before.txt](evidence/file-before.txt) — `file` on the original: extended-ASCII, CRLF
- [evidence/hexdump-before.txt](evidence/hexdump-before.txt) — the first 16 bytes of the original
- [evidence/candidates.txt](evidence/candidates.txt) — the file decoded with five code pages; only RK1048 gets the Kazakh letters right
- [evidence/file-after.txt](evidence/file-after.txt) — `file` on the fixed file: UTF-8 with BOM
- [evidence/view-broken.png](evidence/view-broken.png) — a copy of the original in Excel: Kazakh letters broken
- [evidence/view-broken-libre.png](evidence/view-broken-libre.png) — Calc's import dialog guessing Windows-1251
- [evidence/opens-correctly.png](evidence/opens-correctly.png) — the fixed file in Excel, Calc and `cat`, the same names in all three
