---
layout: post
title: "All Scrabble Hooks for Three letter Words"
category: tutorial
tags: [python]
---
{% include JB/setup %}


I'm a big Scrabble fan, and I was recently looking for a list of all the hooks for the three-letter words. For two letter words, this list ("the 2-to-make-3 list") is in the book *Everything Scrabble* and easy to find [on the web](http://www.wolfberg.net/scrabble/wordlists/OWL2/twos-to-threes.html). But I wasn't able to find an up-to-date ([OWL2](http://www.scrabbleplayers.org/w/Official_Tournament_and_Club_Word_List)) version of the 3-to-make-4 list. So I decided to make one for myself and share it here.

It's easy to find a list of all the allowed three-letter and four-letter words at sites such as [scrabutility](http://scrabutility.com/index.php). I've saved these word lists as text files [threes.txt](/static/threes.txt) and [fours.txt](/static/fours.txt). Then to create the list, I wrote some python code.

{% highlight py3 %}
# Read files to create list of three-letter words and list of four-letter words
def linestolist(filename):
	with open(filename) as f:
		return [x.strip('\n') for x in f.readlines()]
threes = linestolist('threes.txt')
fours = linetolist('fours.txt')

# For each three letter word, see which letters can go before it and after it
letters = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
before = ["" for x in range(len(threes))]
after = ["" for x in range(len(threes))]
for i in range(len(threes)):
	for l in letters:
		if l+threes[i] in fours:
			before[i] += l
		if threes[i]+l in fours:
			after[i] += l
z = zip(before, threes, after)

# Write the result as a CSV file
import csv
with open('threehooks.csv','w') as out:
    csv_out=csv.writer(out)
    csv_out.writerow(['before','word', 'after'])
    for row in z:
        csv_out.writerow(row)
{% endhighlight %}

I know the code isn't as computationally efficient as could be because it doesn't take into account the fact that the word files are sorted. But over-optimizing such a computationally small task isn't an efficient use of my time.


## The List

The list is is shown below; you can also download it as a [nicely-formatted pdf](/static/three-to-make-four.pdf).

| Front Hooks | Word | Back Hooks |
|--------:|:-------:|:--------|
| | AAH | S |
| B | AAL | S |
| BK | AAS |  |
| B | ABA | S |
|  | ABO | S |
| CDFGJKLNSTW | ABS |  |
| BG | ABY | ES |
| DFLMPRT | ACE | DS |
| FPT | ACT | AS |
|  | ADD | S |
| DF | ADO | S |
| BCDFGLMPRTW | ADS |  |
|  | ADZ | E |
| BCDGNRWY | AFF |  |
| DHRW | AFT |  |
| GRS | AGA | RS |
| CGMPRSW | AGE | DERS |
| DS | AGO | GN |
| BDFGHJLMNRSTWYZ | AGS |  |
| H | AHA |  |
|  | AHI | S |
| ADH | AHS |  |
| CLMPQRS | AID | ES |
| BFHJKMNPRSTVW | AIL | S |
| M | AIM | S |
| CFGKLMPRSTVW | AIN | S |
| FHLMPVW | AIR | NSTY |
| DR | AIS |  |
| BGW | AIT | S |
| GNT | ALA | ENRS |
|  | ALB | AS |
| BDGHKMPRSTVW | ALE | CEFS |
| BCFGHLMPSTW | ALL | SY |
| PS | ALP | S |
| ABDGPS | ALS | O |
| HMS | ALT | OS |
| GLM | AMA | HS |
| KR | AMI | ADENRS |
| CDGLRSTV | AMP | S |
|  | AMU | S |
| KMN | ANA | LS |
| BHLRSW | AND | S |
| BCFGJKLMPSVW | ANE | SW |
| BR | ANI | LS |
| CHPRW | ANT | AEIS |
| MWZ | ANY |  |
| CGJNRT | APE | DRSX |
| C | APO | DS |
|  | APP | S |
| R | APT |  |
| BCDG | ARB | S |
| MN | ARC | HOS |
| BCDFHMPRTWY | ARE | AS |
| BZ | ARF | S |
| BCDHLMNPSW | ARK | S |
| BFHW | ARM | SY |
| BCEGJLMOPTVW | ARS | E |
| CDFHKMPTW | ART | SY |
| BCDFGHLMPRSW | ASH | Y |
| BCMT | ASK | S |
| GHRW | ASP | S |
| BLMPST | ASS |  |
| BCDFGHLMPRST | ATE | S |
| BMW | ATT |  |
| JW | AUK | S |
| FJKL | AVA |  |
| CEFGHLNPRSW | AVE | RS |
|  | AVO | SW |
|  | AWA | Y |
|  | AWE | DES |
| BPWY | AWL | S |
| DFLMPSY | AWN | SY |
|  | AXE | DLS |
|  | AYE | S |
| BCDFGHJKLMNPRSWY | AYS |  |
|  | AZO | N |
|  | BAA | LS |
|  | BAD | ES |
|  | BAG | S |
|  | BAH | T |
|  | BAL | DEKLMS |
|  | BAM | S |
|  | BAN | DEGIKS |
|  | BAP | S |
| K | BAR | BDEFKMNS |
| AO | BAS | EHKST |
|  | BAT | EHST |
|  | BAY | S |
| A | BED | SU |
|  | BEE | FNPRST |
|  | BEG | S |
|  | BEL | LST |
|  | BEN | DEST |
| O | BES | T |
| A | BET | AHS |
| O | BEY | S |
|  | BIB | BS |
|  | BID | EIS |
|  | BIG | S |
|  | BIN | DEST |
|  | BIO | GS |
| IO | BIS | EK |
| O | BIT | EST |
|  | BIZ | E |
|  | BOA | RST |
|  | BOB | S |
|  | BOD | ESY |
|  | BOG | SY |
|  | BOO | BKMNRST |
|  | BOP | S |
| A | BOS | HKS |
|  | BOT | AHST |
|  | BOW | LS |
|  | BOX | Y |
|  | BOY | OS |
|  | BRA | DEGNSTWY |
|  | BRO | OSW |
|  | BRR | R |
|  | BUB | OSU |
|  | BUD | S |
|  | BUG | S |
|  | BUM | FPS |
|  | BUN | ADGKNST |
|  | BUR | ABDGLNPRSY |
|  | BUS | HKSTY |
| A | BUT | EST |
|  | BUY | S |
| A | BYE | S |
| A | BYS |  |
| S | CAB | S |
| S | CAD | EIS |
| S | CAM | EOPS |
| S | CAN | EST |
|  | CAP | EHOS |
| S | CAR | BDEKLNPRST |
| S | CAT | ES |
|  | CAW | S |
|  | CAY | S |
|  | CEE | S |
|  | CEL | LST |
|  | CEP | ES |
|  | CHI | ACDNPST |
|  | CIG | S |
|  | CIS | T |
|  | COB | BS |
|  | COD | AES |
|  | COG | S |
|  | COL | ADESTY |
| I | CON | EIKNSY |
|  | COO | FKLNPST |
| S | COP | ESY |
|  | COR | DEFKMNSY |
|  | COS | HSTY |
| S | COT | ES |
| S | COW | LSY |
|  | COX | A |
|  | COY | S |
|  | COZ | Y |
| E | CRU | DSX |
| S | CRY |  |
|  | CUB | ES |
| S | CUD | S |
|  | CUE | DS |
| S | CUM |  |
| S | CUP | S |
|  | CUR | BDEFLNRST |
| S | CUT | ES |
|  | CWM | S |
|  | DAB | S |
|  | DAD | AOS |
|  | DAG | OS |
| O | DAH | LS |
|  | DAK | S |
|  | DAL | ES |
|  | DAM | ENPS |
|  | DAN | GKS |
|  | DAP | S |
|  | DAW | KNST |
|  | DAY | S |
|  | DEB | ST |
|  | DEE | DMPRST |
|  | DEF | ITY |
|  | DEL | EFILST |
|  | DEN | EISTY |
|  | DEV | AS |
|  | DEW | SY |
|  | DEX | Y |
|  | DEY | S |
|  | DIB | S |
|  | DID | OY |
|  | DIE | DLST |
|  | DIF | FS |
|  | DIG | S |
|  | DIM | ES |
|  | DIN | EGKOST |
|  | DIP | ST |
|  | DIS | CHKS |
| AE | DIT | AESZ |
|  | DOC | KS |
|  | DOE | RS |
|  | DOG | ESY |
| I | DOL | ELST |
|  | DOM | ES |
| U | DON | AEGS |
| O | DOR | EKMPRSY |
| AU | DOS | EST |
|  | DOT | EHSY |
|  | DOW | NS |
|  | DRY | S |
|  | DUB | S |
|  | DUD | ES |
|  | DUE | LST |
|  | DUG | S |
|  | DUH |  |
|  | DUI | T |
|  | DUN | EGKST |
|  | DUO | S |
|  | DUP | ES |
|  | DYE | DRS |
| BDFGHLNPRSTWY | EAR | LNS |
| BFHMNPST | EAT | HS |
| B | EAU | X |
|  | EBB | S |
|  | ECU | S |
|  | EDH | S |
| BFGMPRTWZ | EDS |  |
| GKLMPRSW | EEK |  |
| FHKPRSTW | EEL | SY |
| T | EFF | S |
| KR | EFS |  |
| DHLRW | EFT | S |
| TY | EGG | SY |
| S | EGO | S |
| DLP | EKE | DS |
| GHMVWY | ELD | S |
| DPS | ELF |  |
| Y | ELK | S |
| BCDFHJMSTWY | ELL | S |
| H | ELM | SY |
| BCDEGMST | ELS | E |
| DFHMS | EME | SU |
| FGHMR | EMS |  |
|  | EMU | S |
| BFLMPRSTVW | END | S |
|  | ENG | S |
| BDFGHKLPTWY | ENS |  |
| AJNP | EON | S |
| SV | ERA | S |
| CDFHMPSW | ERE |  |
| B | ERG | OS |
| FHKT | ERN | ES |
|  | ERR | S |
| HS | ERS | T |
| CFJLMN | ESS |  |
| BFGMSZ | ETA | S |
| BHMT | ETH | S |
| N | EVE | NRS |
|  | EWE | RS |
|  | EYE | DNRS |
|  | FAB | S |
|  | FAD | EOS |
|  | FAG | S |
|  | FAN | EGOS |
| A | FAR | DELMOT |
|  | FAS | HT |
|  | FAT | ES |
|  | FAX |  |
| O | FAY | S |
|  | FED | S |
|  | FEE | BDLST |
|  | FEH | S |
|  | FEM | ES |
|  | FEN | DS |
|  | FER | EN |
|  | FES | ST |
|  | FET | AES |
|  | FEU | DS |
|  | FEW |  |
|  | FEY |  |
|  | FEZ |  |
|  | FIB | S |
|  | FID | OS |
|  | FIE | F |
|  | FIG | S |
|  | FIL | AELMOS |
|  | FIN | DEKOS |
|  | FIR | EMNS |
|  | FIT | S |
|  | FIX | T |
|  | FIZ | Z |
|  | FLU | BESX |
|  | FLY |  |
|  | FOB | S |
|  | FOE | S |
|  | FOG | SY |
|  | FOH | N |
|  | FON | DST |
|  | FOP | S |
|  | FOR | ABDEKMT |
|  | FOU | LR |
|  | FOX | Y |
|  | FOY | S |
|  | FRO | EGMW |
|  | FRY |  |
|  | FUB | S |
|  | FUD | S |
|  | FUG | SU |
|  | FUN | DKS |
|  | FUR | LSY |
|  | GAB | SY |
| E | GAD | IS |
|  | GAE | DNS |
|  | GAG | AES |
| E | GAL | AELS |
| O | GAM | ABEPSY |
|  | GAN | EG |
|  | GAP | ESY |
| A | GAR | BS |
| A | GAS | HPT |
|  | GAT | ES |
|  | GAY | S |
| A | GED | S |
| AO | GEE | DKSZ |
|  | GEL | DST |
|  | GEM | S |
|  | GEN | ESTU |
|  | GET | AS |
|  | GEY |  |
|  | GHI | S |
|  | GIB | ES |
|  | GID | S |
|  | GIE | DNS |
|  | GIG | AS |
| A | GIN | KS |
|  | GIP | S |
|  | GIT | ES |
|  | GNU | S |
|  | GOA | DLST |
|  | GOB | OSY |
|  | GOD | S |
|  | GOO | DFKNPS |
|  | GOR | EMPY |
| E | GOS | H |
|  | GOT | H |
|  | GOX |  |
|  | GOY | S |
|  | GUL | FLPS |
|  | GUM | S |
|  | GUN | KS |
|  | GUT | S |
|  | GUV | S |
|  | GUY | S |
|  | GYM | S |
|  | GYP | S |
| CS | HAD | EJ |
| T | HAE | DMNST |
| S | HAG | S |
| S | HAH | AS |
|  | HAJ | IJ |
| CSW | HAM | ES |
| C | HAO |  |
| CW | HAP | S |
|  | HAS | HPT |
| CGKPSTW | HAT | EHS |
| CST | HAW | KS |
| CS | HAY | S |
|  | HEH | S |
| AT | HEM | EPS |
| TW | HEN | ST |
|  | HEP |  |
|  | HER | BDELMNOS |
| S | HES | T |
| KW | HET | HS |
| CPSTW | HEW | NS |
|  | HEX |  |
| TW | HEY |  |
| C | HIC | K |
| CW | HID | E |
|  | HIE | DS |
| SW | HIM | S |
| CSTW | HIN | DST |
| CSW | HIP | S |
| ACGKPT | HIS | NST |
| CSW | HIT | S |
|  | HMM |  |
|  | HOB | OS |
| S | HOD | S |
| S | HOE | DRS |
| S | HOG | GS |
| CP | HON | EGKS |
| CSW | HOP | ES |
| MR | HOS | ET |
| PS | HOT | S |
| CDS | HOW | EFKLS |
| A | HOY | AS |
| C | HUB | S |
|  | HUE | DS |
| CT | HUG | ES |
|  | HUH |  |
| C | HUM | PS |
| S | HUN | GHKST |
| W | HUP |  |
| BPS | HUT | S |
|  | HYP | EOS |
| BDFLMNPRSV | ICE | DS |
| LRW | ICH | S |
| DHKLMNPRSTW | ICK | Y |
|  | ICY |  |
| ABFGKLMRVY | IDS |  |
| BDJMRT | IFF | Y |
| DKR | IFS |  |
| M | IGG | S |
| BMS | ILK | AS |
| BDFGHJKMNPRSTVWYZ | ILL | SY |
| GJLPSW | IMP | IS |
| DFGJKLMOPRSW | INK | SY |
| JL | INN | S |
| ABDFGHJKLPRSTWYZ | INS |  |
| CLP | ION | S |
| CDFHLMSTW | IRE | DS |
| BDKM | IRK | S |
| J | ISM | S |
| ABDFGHKLNPSTWZ | ITS |  |
| JT | IVY |  |
|  | JAB | S |
|  | JAG | GS |
|  | JAM | BS |
| A | JAR | LS |
|  | JAW | S |
|  | JAY | S |
| A | JEE | DPRSZ |
|  | JET | ES |
|  | JEU | X |
|  | JEW | S |
|  | JIB | BES |
|  | JIG | S |
| D | JIN | KNSX |
|  | JOB | S |
|  | JOE | SY |
|  | JOG | S |
|  | JOT | AS |
|  | JOW | LS |
|  | JOY | S |
|  | JUG | AS |
|  | JUN | K |
|  | JUS | T |
|  | JUT | ES |
|  | KAB | S |
|  | KAE | S |
|  | KAF | S |
| OS | KAS |  |
| IS | KAT | AS |
| O | KAY | OS |
|  | KEA | S |
|  | KEF | S |
| S | KEG | S |
|  | KEN | OST |
| S | KEP | IST |
|  | KEX |  |
|  | KEY | S |
|  | KHI | S |
| S | KID | S |
|  | KIF | S |
| AS | KIN | ADEGKOS |
| S | KIP | S |
|  | KIR | KNS |
| S | KIS | ST |
| S | KIT | EHS |
|  | KOA | NS |
|  | KOB | OS |
|  | KOI | S |
|  | KOP | HS |
|  | KOR | AES |
|  | KOS | S |
|  | KUE | S |
|  | KYE | S |
| BFS | LAB | S |
|  | LAC | EKSY |
| CG | LAD | ESY |
| CFS | LAG | S |
| BCFGS | LAM | ABEPS |
| CFS | LAP | S |
| A | LAR | DIKS |
| A | LAS | EHST |
| BFPS | LAT | EHISU |
|  | LAV | AES |
| BCFS | LAW | NS |
| F | LAX |  |
| CFPS | LAY | S |
| FIOP | LEA | DFKLNPRS |
| BFGPS | LED |  |
| AFG | LEE | KRST |
| G | LEG | S |
|  | LEI | S |
|  | LEK | ESU |
| AO | LES | ST |
| B | LET | S |
|  | LEU | D |
|  | LEV | AOY |
| FIP | LEX |  |
| FG | LEY | S |
|  | LEZ |  |
| G | LIB | S |
| S | LID | OS |
| P | LIE | DFNRSU |
| B | LIN | EGKNOSTY |
| BCFS | LIP | AES |
|  | LIS | PT |
| AFS | LIT | ESU |
| BGS | LOB | EOS |
| BCFS | LOG | EOSY |
|  | LOO | FKMNPST |
| CFGPS | LOP | ES |
| BCPS | LOT | AHIS |
| ABFGPS | LOW | ENS |
|  | LOX |  |
| GPS | LUG | ES |
| AGPS | LUM | APS |
|  | LUV | S |
| F | LUX | E |
|  | LYE | S |
|  | MAC | EHKS |
|  | MAD | ES |
|  | MAE | S |
|  | MAG | EIS |
|  | MAN | AEOSY |
|  | MAP | S |
|  | MAR | ACEKLST |
| A | MAS | AHKST |
|  | MAT | EHST |
|  | MAW | NS |
|  | MAX | I |
|  | MAY | AOS |
|  | MED | S |
|  | MEG | AS |
|  | MEL | DLST |
|  | MEM | EOS |
| AO | MEN | DOU |
|  | MET | AEH |
| S | MEW | LS |
|  | MHO | S |
|  | MIB | S |
| E | MIC | AEKS |
| AI | MID | IS |
|  | MIG | GS |
|  | MIL | DEKLOST |
|  | MIM | E |
| AE | MIR | EIKSY |
| A | MIS | EOST |
|  | MIX | T |
|  | MOA | NST |
|  | MOB | S |
|  | MOC | KS |
|  | MOD | EIS |
| S | MOG | S |
|  | MOL | ADELSTY |
|  | MOM | EIS |
|  | MON | KOSY |
|  | MOO | DLNRST |
|  | MOP | ESY |
|  | MOR | AENST |
|  | MOS | HKST |
|  | MOT | EHST |
|  | MOW | NS |
|  | MUD | S |
| S | MUG | GS |
|  | MUM | MPSU |
|  | MUN | IS |
| AE | MUS | EHKST |
| S | MUT | EST |
|  | MYC | S |
|  | NAB | ES |
|  | NAE |  |
| S | NAG | S |
|  | NAH |  |
|  | NAM | E |
|  | NAN | AS |
| KS | NAP | AES |
| GS | NAW |  |
|  | NAY | S |
|  | NEB | S |
| K | NEE | DMP |
|  | NEG | S |
|  | NET | ST |
| AK | NEW | ST |
| S | NIB | S |
| A | NIL | LS |
|  | NIM | S |
| S | NIP | AS |
| KSU | NIT | ES |
|  | NIX | EY |
| KS | NOB | S |
|  | NOD | EIS |
| S | NOG | GS |
|  | NOH |  |
|  | NOM | AES |
|  | NOO | KN |
|  | NOR | IM |
| O | NOS | EHY |
| KS | NOT | AE |
| EKS | NOW | ST |
|  | NTH |  |
| S | NUB | S |
|  | NUN | S |
| AGO | NUS |  |
|  | NUT | S |
| L | OAF | S |
| S | OAK | SY |
| BHRS | OAR | S |
| BCDGM | OAT | HS |
| S | OBA | S |
| LR | OBE | SY |
|  | OBI | AST |
| CLS | OCA | S |
| CS | ODA | HS |
|  | ODD | S |
| BCLMNR | ODE | AS |
| BCGHMNPRSTY | ODS |  |
| DFGHJNRTVW | OES |  |
| BCDT | OFF | S |
| CLST | OFT |  |
|  | OHM | S |
| BC | OHO |  |
| O | OHS |  |
| BCFMNRST | OIL | SY |
|  | OKA | SY |
| CHJMPSTWY | OKE | HS |
| BCFGHMSTW | OLD | SY |
| BCDHJMPRSTV | OLE | AOS |
| DMNPRST | OMS |  |
| BCDGHLNPSTZ | ONE | S |
| M | ONO | S |
| CDEFHIMPSTW | ONS |  |
| P | OOH | S |
| BCFHLMRST | OOT | S |
| CDHLMNPRT | OPE | DNS |
| BCFHKLMOPSTW | OPS |  |
|  | OPT | S |
| BFHKMST | ORA | DL |
| FS | ORB | SY |
| T | ORC | AS |
| BCDFGKLMPSTWY | ORE | S |
| CDKMT | ORS |  |
| BFMPSTW | ORT | S |
| DHLNPR | OSE | S |
| L | OUD | S |
| DFHLPSTY | OUR | S |
| BGLPRT | OUT | S |
| N | OVA | L |
| HLY | OWE | DS |
| BCFHJY | OWL | S |
| DGLMST | OWN | S |
|  | OXO |  |
| BDFP | OXY |  |
|  | PAC | AEKSTY |
|  | PAD | IS |
| O | PAH |  |
| O | PAL | ELMPSY |
| S | PAM | S |
| S | PAN | EGST |
|  | PAP | AS |
| S | PAR | ADEKRST |
| SU | PAS | EHST |
| S | PAT | EHSY |
|  | PAW | LNS |
|  | PAX |  |
| S | PAY | S |
|  | PEA | GKLNRST |
| S | PEC | HKS |
| AOS | PED | S |
| E | PEE | DKLNPRS |
|  | PEG | S |
|  | PEH | S |
| O | PEN | DST |
|  | PEP | OS |
| A | PER | EIKMPTV |
| AO | PES | OT |
|  | PET | S |
| S | PEW | S |
|  | PHI | SZ |
|  | PHT |  |
|  | PIA | LNS |
| ES | PIC | AEKS |
|  | PIE | DRS |
|  | PIG | S |
| S | PIN | AEGKSTY |
|  | PIP | ESY |
|  | PIS | HOS |
| S | PIT | AHSY |
|  | PIU |  |
|  | PIX | Y |
|  | PLY |  |
| A | POD | S |
|  | POH |  |
|  | POI | S |
|  | POL | ELOSY |
|  | POM | EOPS |
|  | POO | DFHLNPRS |
|  | POP | ES |
| S | POT | S |
|  | POW | S |
|  | POX | Y |
|  | PRO | ADFGMPSW |
| S | PRY |  |
|  | PSI | S |
|  | PST |  |
|  | PUB | S |
| S | PUD | S |
|  | PUG | HS |
|  | PUL | AEILPS |
| S | PUN | AGKSTY |
|  | PUP | ASU |
| S | PUR | EILRS |
| O | PUS | HS |
|  | PUT | STZ |
|  | PYA | S |
|  | PYE | S |
|  | PYX |  |
|  | QAT | S |
|  | QIS |  |
| A | QUA | DGIY |
| BGOT | RAD | S |
| BCDF | RAG | AEGIS |
|  | RAH |  |
|  | RAI | ADLNS |
|  | RAJ | A |
| CDGPT | RAM | IPS |
| BG | RAN | DGIKT |
| CFTW | RAP | EST |
| BE | RAS | EHP |
| BDFGP | RAT | EHOS |
| BCD | RAW | S |
|  | RAX |  |
| BDFGPT | RAY | AS |
|  | REB | S |
|  | REC | KS |
| BCI | RED | DEOS |
| BDFGPT | REE | DFKLS |
| T | REF | ST |
| D | REG | S |
|  | REI | FNS |
|  | REM | S |
| P | REP | OPS |
| AIOT | RES | HT |
| FT | RET | ES |
|  | REV | S |
| P | REX |  |
|  | RHO | S |
| A | RIA | LS |
| CD | RIB | S |
| AGI | RID | ES |
|  | RIF | EFST |
| BFGPT | RIG | S |
| BGPT | RIM | ESY |
| BG | RIN | DGKS |
| DGT | RIP | ES |
|  | ROB | ES |
| C | ROC | KS |
| PT | ROD | ES |
| F | ROE | S |
| FP | ROM | PS |
| GT | ROT | AEILOS |
| BCFGPTV | ROW | S |
| DG | RUB | ESY |
| GT | RUE | DRS |
| DFT | RUG | AS |
| ADG | RUM | PS |
|  | RUN | EGST |
| B | RUT | HS |
|  | RYA | S |
|  | RYE | S |
|  | SAB | ES |
|  | SAC | KS |
|  | SAD | EI |
|  | SAE |  |
|  | SAG | AEOSY |
|  | SAL | ELPST |
|  | SAP | S |
|  | SAT | EI |
|  | SAU | L |
|  | SAW | NS |
|  | SAX |  |
|  | SAY | S |
| A | SEA | LMRST |
|  | SEC | ST |
|  | SEE | DKLMNPRS |
|  | SEG | OS |
|  | SEI | FS |
|  | SEL | FLS |
|  | SEN | DET |
| U | SER | AEFS |
|  | SET | AST |
|  | SEW | NS |
|  | SEX | TY |
|  | SHA | DGHMTWY |
|  | SHE | ADSW |
|  | SHH |  |
| A | SHY |  |
|  | SIB | BS |
|  | SIC | EKS |
|  | SIM | APS |
|  | SIN | EGHKS |
|  | SIP | ES |
|  | SIR | ES |
| P | SIS |  |
|  | SIT | EHS |
|  | SIX |  |
|  | SKA | GST |
|  | SKI | DMNPST |
|  | SKY |  |
|  | SLY |  |
|  | SOB | AS |
|  | SOD | AS |
|  | SOL | ADEIOS |
|  | SOM | AES |
|  | SON | EGS |
|  | SOP | HS |
|  | SOS |  |
|  | SOT | HS |
|  | SOU | KLPRS |
|  | SOW | NS |
|  | SOX |  |
|  | SOY | AS |
|  | SPA | EMNRSTYZ |
| E | SPY |  |
|  | SRI | S |
|  | STY | E |
|  | SUB | AS |
|  | SUE | DRST |
|  | SUK | S |
|  | SUM | OPS |
|  | SUN | GKNS |
|  | SUP | ES |
|  | SUQ | S |
|  | SYN | CE |
| S | TAB | SU |
|  | TAD | S |
|  | TAE | L |
| S | TAG | S |
|  | TAJ |  |
|  | TAM | EPS |
|  | TAN | GKS |
|  | TAO | S |
| A | TAP | AES |
| S | TAR | ENOPST |
| EU | TAS | KS |
| S | TAT | ES |
|  | TAU | ST |
|  | TAV | S |
| S | TAW | S |
|  | TAX | AI |
|  | TEA | KLMRST |
|  | TED | S |
|  | TEE | DLMNS |
|  | TEG | GS |
|  | TEL | AELS |
|  | TEN | DST |
| S | TET | HS |
| S | TEW | S |
|  | THE | EMNWY |
|  | THO | U |
|  | THY |  |
| EO | TIC | KS |
|  | TIE | DRS |
|  | TIL | ELST |
|  | TIN | EGSTY |
|  | TIP | IS |
|  | TIS |  |
|  | TIT | IS |
|  | TOD | SY |
|  | TOE | ADS |
|  | TOG | AS |
| A | TOM | BES |
|  | TON | EGSY |
|  | TOO | KLMNT |
| AS | TOP | EHIOS |
|  | TOR | ACEINORSTY |
| S | TOT | ES |
| S | TOW | NSY |
|  | TOY | OS |
|  | TRY |  |
|  | TSK | S |
| S | TUB | AES |
|  | TUG | S |
| EP | TUI | S |
| S | TUN | AEGS |
|  | TUP | S |
|  | TUT | SU |
|  | TUX |  |
|  | TWA | EST |
|  | TWO | S |
| S | TYE | ERS |
| JK | UDO | NS |
| PSV | UGH | S |
| CDJNP | UKE | S |
| LS | ULU | S |
| M | UMM |  |
| BDHJLMPRST | UMP | S |
| BDFGHMNPRST | UNS |  |
|  | UPO | N |
| CDPSTY | UPS |  |
| BC | URB | S |
| BCNST | URD | S |
| BCDT | URN | S |
| B | URP | S |
| FMR | USE | DRS |
|  | UTA | S |
| BCJLM | UTE | S |
| BCGHJMNOPRT | UTS |  |
|  | VAC | S |
|  | VAN | EGS |
|  | VAR | ASY |
| K | VAS | AET |
|  | VAT | SU |
|  | VAU | S |
|  | VAV | S |
|  | VAW | S |
|  | VEE | PRS |
|  | VEG |  |
|  | VET | OS |
|  | VEX | T |
|  | VIA | L |
| A | VID | ES |
|  | VIE | DRSW |
|  | VIG | AS |
|  | VIM | S |
|  | VIS | AE |
|  | VOE | S |
| A | VOW | S |
|  | VOX |  |
|  | VUG | GHS |
| O | VUM |  |
| S | WAB | S |
|  | WAD | EISY |
| T | WAE | S |
| S | WAG | ES |
| HS | WAN | DEKSTY |
| S | WAP | S |
|  | WAR | DEKMNPSTY |
| T | WAS | HPT |
| ST | WAT | ST |
|  | WAW | LS |
|  | WAX | Y |
| AS | WAY | S |
|  | WEB | S |
| AO | WED | S |
| AT | WEE | DKLNPRST |
|  | WEN | DST |
|  | WET | S |
|  | WHA | MPT |
|  | WHO | AMP |
|  | WHY | S |
| ST | WIG | S |
| T | WIN | DEGKOSY |
| IY | WIS | EHPST |
| T | WIT | EHS |
|  | WIZ |  |
|  | WOE | S |
|  | WOG | S |
|  | WOK | ES |
|  | WON | KST |
|  | WOO | DFLS |
| S | WOP | S |
| T | WOS | T |
| S | WOT | S |
|  | WOW | S |
| A | WRY |  |
|  | WUD |  |
|  | WYE | S |
|  | WYN | DNS |
| A | XIS |  |
|  | YAG | IS |
| A | YAH |  |
| K | YAK | S |
|  | YAM | S |
|  | YAP | S |
| K | YAR | DEN |
|  | YAW | LNPS |
|  | YAY | S |
|  | YEA | HNRS |
|  | YEH |  |
| E | YEN | S |
|  | YEP | S |
| ABDEKLOPRTW | YES |  |
|  | YET | IT |
|  | YEW | S |
|  | YID | S |
| APT | YIN | S |
|  | YIP | ES |
|  | YOB | S |
|  | YOD | HS |
|  | YOK | ES |
|  | YOM |  |
|  | YON | DI |
|  | YOU | RS |
|  | YOW | ELS |
|  | YUK | S |
|  | YUM |  |
|  | YUP | S |
|  | ZAG | S |
|  | ZAP | S |
|  | ZAS |  |
|  | ZAX |  |
|  | ZED | S |
|  | ZEE | S |
|  | ZEK | S |
|  | ZEP | S |
|  | ZIG | S |
|  | ZIN | CEGS |
|  | ZIP | S |
|  | ZIT | IS |
|  | ZOA |  |
|  | ZOO | MNS |
|  | ZUZ |  |
{: class="table"}

