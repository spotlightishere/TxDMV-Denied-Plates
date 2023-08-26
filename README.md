# TxDMV Denied Plates
The repositories [ca-license-plates](https://github.com/veltman/ca-license-plates) (and its subsequent [updated version](https://github.com/21five/ca-license-plates)) are datasets containing information about specialty plates flagged for review with quite a lot of... interesting review commentary. At the time, a [popular Twitter bot](https://botsin.space/@ca_dmv_bot) was posting absurd requests and decisions.

I was curious if Texas had similar information, so on January 3rd, 2023, I sent in a FOIA request to [TxDMV](https://www.txdmv.gov). In it, I requested approved and denied specialty plates - alongside any commentary - through the time period of January 1st, 2021 to December 31st, 2022. As Texas allows you to have three possible choices, I additionally asked for such to be included.

On January 18th, 2023, a surprisingly fast response was received, informing:
1. Texas cannot give out information about issued specialty license plates per the [DPPA](https://en.wikipedia.org/wiki/Driver's_Privacy_Protection_Act). :(
2. A lot of these applications are processed at a county level, so the state-level agency never sees possible choices. :(
3. There's no commentary recorded for declined plates - instead, reason codes are given. :(
4. It'd cost a small fortune to run a "custom query" to associate the denial reason codes with the actual plates. :(

:(

However, it was insisted that many license plates are self explanatory per the [reason codes](#reason-codes). On January 18th, 2023, slightly annoyed, I wrote back saying that the PDFs of only denied license plates would be acceptable.

On August 25, 2023, TxDMV reached out with the PDFs, citing a growing backlog for the delay. I'm just amazed they got back to me.

## Reason Codes
As defined in [Title 43, part 10, chapter 217, subchapter B, rule 217.27 ("Vehicle Registration Insignia")](https://texreg.sos.state.tx.us/public/readtac$ext.TacPage?sl=R&app=9&p_dir=&p_rloc=&p_tloc=&p_ploc=&pg=1&p_tac=&ti=43&pt=10&ch=217&rl=27), specialty license plates may be rejected if:

1. The alphanumeric pattern conflicts with the department's current or proposed regular license plate numbering system.
2. The director or the director's designee finds that the personalized alphanumeric pattern may be considered objectionable. An objectionable alphanumeric pattern may include words, [or] phrases, or slang in any language; phonetic, numeric, or reverse spelling; acronyms; patterns viewed in mirror image; or code that only a small segment of the community may be able to readily decipher. An objectionable pattern may be viewed as:
    1. A) indecent (defined as including a direct reference or connotation to a sexual act, sexual body parts, excreta, or sexual bodily fluids or functions. Additionally, the alphanumeric pattern "69" is prohibited unless used with the full year (1969) or in combination with a reference to a vehicle;
    2. B) vulgar, directly or indirectly (defined as profane, swear, or curse words);
    3. C) derogatory, directly or indirectly (defined as an expression that is demeaning to, belittles, or disparages any person, group, race, ethnicity, nationality, gender, or sexual orientation. "Derogatory" may also include a reference to an organization that advocates the expressions described in this subparagraph);
    4. D) a direct or indirect negative instruction or command directed at another individual related to the operation of a motor vehicle;
    5. E) a direct or indirect reference to gangs, illegal activities, implied threats of harm, or expressions that describe, advertise, advocate, promote, encourage, glorify, or condone violence, crime, or unlawful conduct;
    6. F) a direct or indirect reference to controlled substances or the physiological state produced by such substances, intoxicated states, or a direct or indirect reference that may express, describe, advertise, advocate, promote, encourage, or glorify such substances or states;
    7. G) a direct representation of law enforcement or other governmental entities, including any reference to a public office or position exclusive to government; or
    8. H) a pattern that could be misread by law enforcement.
3. The alphanumeric pattern is currently on a license plate issued to another owner.

## Generation
(I'm not a data scientist - I apologize! There are likely more procedural ways to do this, but I lack the knowledge.)

After struggling for several hours with a mixture of OCR/text parsing, regex, and hand-copying, I settled on using Excel as it supposedly could handle importing the PDFs and interpreting their tables directly. Unfortunately, I quickly found that it struggled with having multiple tables per page (as present within each PDF).

In order to correctly have Excel scrape data from the PDFs, the following command was utilized to split each page of the PDF into halves. Thank you to this [Super User answer](https://superuser.com/a/1000913)!

```sh
for current in *.pdf; do
  mutool poster -x 2 $current $(basename $current .pdf)_split.pdf
done
```

Each split PDF was then manually imported into Excel via its [PDF data connector](https://techcommunity.microsoft.com/t5/excel-blog/announcing-data-import-from-pdf-documents/ba-p/1569202). (Although `mutool` supports merging PDFs, importing all split tables at once has Excel easily exhaust my 32 GB RAM.)

A few custom steps were applied within Power Query to all tables:
- In order to accommodate only the left-hand side of the page,the "Month" column was dropped.
- In some circumstances, "Column 3" was created due to erroneous handling of certain license plates containing spaces. This column was merged with "Plate Selection" using a space as a separator.
- The total amount of license plate numbers present at the end of "Plate Selection" were manually removed (i.e. where Request Date was null).

# Conclusions
- Although my high-end gaming rig can handle Cyberpunk 2077 at highest graphics and build LLVM in about 20 minutes, it could not handle the full wrath of Excel.
- It's no wonder that [Excel can do ray-tracing](https://github.com/s0lly/Raytracer-In-Excel).
- We should all be wary of those who know how to use Excel.
