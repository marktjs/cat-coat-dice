# Cat Coat Dice v1.0 — Genetics Correctness Review & Correction Advice

**Scope:** All genes, alleles, dominance relationships, eye-color logic, explanatory text, and genotype probabilities in [`cat-coat-dice.html`](cat-coat-dice.html) (live: https://marktjs.github.io/cat-coat-dice/) and [`README.md`](README.md).

**Verdict:** Most of the basic genetics is right: the X-linked orange gene (*ARHGAP36*), *TYRP1*, *MLPH*, *ASIP*, the *TYR* colorpoint series, the dilution names, orange being epistatic to non-agouti, and silver/smoke/cameo naming. But there are **three structural errors** (the two *KIT* loci, the tabby "series", and the FERV1 allele assignment), **several mislabels**, and **probability tables that are not actually Hardy–Weinberg consistent** even though the code says they are.

Severity legend: 🔴 factually wrong / biologically impossible · 🟠 misleading or oversimplified · 🟡 minor / naming / optional

---

## 🔴 1. `W` and `Ws` are modeled as two independent loci, but they are alleles of the same *KIT* gene

**Where:** `LOCUS_OPTIONS.whiteGenotype` + `LOCUS_OPTIONS.domWhiteGenotype` (html ~L1116–1125), `LOCUS_PROBS` (~L1202–1211), the gene switcher, and gene cards 9 and 10.

**Problem:** David et al. (2014, *G3*) showed that wild type, the FERV1 full-length element, and the FERV1 LTR-only insertion form **one triallelic marker in *KIT* intron 1** (Chr B1). Each cat therefore carries exactly **two** *KIT* alleles. The current model lets a cat be `Ws/Ws` **and** `W/W` at once (four *KIT* alleles), which is impossible. All 734,832 numbered combinations include these impossible pairings.

**Fix:** Merge the two loci into a single *KIT* locus with three alleles: `W` > `Ws` > `w+`.

| *KIT* genotype | Phenotype |
| :--- | :--- |
| `w+/w+` | No white |
| `Ws/w+` | Low–medium white (bib, paws, muzzle; variable) |
| `Ws/Ws` | Medium–high white (bicolor → harlequin/van; variable) |
| `W/w+`, `W/Ws`, `W/W` | Solid white (W is dominant/epistatic to everything) |

That gives **6 genotypes** instead of 3 × 3 = 9 independent pairs.

---

## 🔴 2. FERV1 allele assignment is reversed in README and in my earlier explanation

**Where:** [README.md L55–56, L81–82](README.md)

**Problem:** The README says W is the *full-length (~7.1 kb)* insertion and Ws is the *truncated (~662 bp LTR)* insertion, citing *PLoS Genetics*. The primary source says the opposite:

> "A full-length (7125 bp) FERV1 element is associated with **white spotting**, whereas a FERV1 long terminal repeat (LTR) is associated with all **Dominant White** individuals." — David *et al.* 2014, *G3: Genes|Genomes|Genetics* 4:1881–1891.

**Fix:**
- `Ws` = full-length **7,125 bp** FERV1 insertion.
- `W` = FERV1 **LTR-only** insertion (a few hundred bp; secondary sources quote ~620 bp).
- Change the citation from *PLoS Genetics* to ***G3* (2014)**.

---

## 🔴 3. The horse comparison is wrong ("unlike lethal white in horses")

**Where:** html L1066 (`kit-note`), html L2182 (gene card 10), README L54 ("Frame Overo / Dominant White").

**Problem:** Equine **Lethal White Foal Syndrome / Frame Overo is caused by *EDNRB*, not *KIT***, so it is not a counterpart to cat `W/W`. It is true that horse *KIT* dominant-white alleles and many mouse *Kit* `W` alleles are thought to be homozygous lethal, sterile, or anemic.

**Fix:** Change the text to something like: *"Unlike many mouse* Kit *W alleles and some horse* KIT *dominant-white alleles (presumed homozygous-lethal), feline W/W is viable, because the cat mutation is a regulatory insertion in intron 1 rather than a loss of the coding sequence."* Remove "Frame Overo" from the README.

---

## 🔴 4. Tabby, Spotted, and Ticked are treated as one allelic series (`Ta > Sp > Mc > mc`)

**Where:** `LOCUS_OPTIONS.tabbyPattern` (`Mc/Mc, Mc/mc, mc/mc, Sp/Sp, Sp/mc, Ta/Ta, Ta/mc`), gene card 6 (L2144), and README row 6.

**Problem:** This is the outdated pre-2012 "Abyssinian allele" model. Molecular work has since separated these:

| Trait | Gene | Chromosome | Alleles | Notes |
| :--- | :--- | :--- | :--- | :--- |
| Mackerel vs. Classic | ***Taqpep*** (= *LVRN*) | **A1** | `Mc` > `mc` (often written `TaM` > `Tab`) | Kaelin et al. 2012 |
| Ticked | ***DKK4*** | **B1** | `TiA` (dominant) vs. `Ti+` | Kaelin et al. 2021. **Epistatic to *Taqpep*.** `TiA/Ti+` shows residual leg/tail/neck bars; `TiA/TiA` is nearly stripeless |
| Spotted | **No gene identified** | — | Hypothetical modifier(s) that break mackerel/classic stripes into spots | Should not be presented as a confirmed `Sp` allele |

Genotypes like `Ta/mc` or `Sp/mc` mix alleles of different genes into one pair. Also, a ticked cat should still carry an underlying `Mc/–` or `mc/mc` genotype that it can pass on.

**Fix:** Split the single locus into:
- **Locus 6a, *Taqpep* (A1):** `Mc/Mc`, `Mc/mc`, `mc/mc`
- **Locus 6b, *DKK4* Ticked (B1):** `Ti+/Ti+`, `TiA/Ti+`, `TiA/TiA` (epistatic to 6a)
- **Spotted:** optional. Either make it a labeled "polygenic/unidentified modifier" toggle or drop it from the genotype count.

Also rename the card from "LVRN / DKK4" to two cards, and stop calling ticked `Ta`.

---

## 🔴 5. Eye-color label bug: `W/w` cats can be labeled "KIT W/W"

**Where:** `buildCatFromState`, html L1353–1359:

```js
if (domWhiteGenotype === "W/W" || whiteGenotype === "Ws/Ws" || colorpointMode === "Pointed") {
  eyeDescription = "Two Sky-Blue Eyes (KIT W/W)";
```

**Problem:** A `W/w` cat that is also `Ws/Ws` or pointed is shown as "(KIT W/W)". Under fix #1 the `W/w + Ws/Ws` case disappears. The pointed case should say "(TYR cs)" or "(KIT W + TYR cs)".

**Fix:** Build the label from the cause that actually applies, not a hard-coded `W/W` string.

---

## 🟠 6. Eye color is deterministic, but in real cats it is probabilistic

**Where:** `buildCatFromState` (L1350–1369), `describeCat` (L1404–1415, L1506–1508), kit-note (L1066), gene cards 9 and 10, README §1B Layer 7 and §2A.

**Problems:**
- **`W/w` → always odd-eyed, and `W/W` → always blue-eyed is not supported.** David et al. describe *incomplete penetrance* for iris hypopigmentation and deafness in all W carriers. A dominant-white cat of any W genotype can have **two blue, one blue (odd), or two copper/gold/green eyes**. No published data shows that W/W is deterministically blue-eyed or W/w deterministically odd-eyed.
- **`Ws/Ws` → always odd-eyed** is also overstated. Most high-white bicolor/van cats have normally pigmented eyes, and only some are odd- or blue-eyed.
- The `describeCat` odd-eye sentence (L1507) always blames "Homozygous high white spotting (KIT Ws/Ws)", even when the cause is W.

**Fix (recommended):** For `W/–` and `Ws/Ws`, draw eye color from a distribution instead of fixing it. For example, illustratively: W/–: blue 40% / odd 20% / copper-gold-green 40%; Ws/Ws: blue 5% / odd 10% / normal 85%. Label these numbers "illustrative", since there are no good population estimates. Or keep it deterministic but change the wording to "**can** cause" / "**increased chance of**". Add a note that blue eyes in white cats are linked to **congenital sensorineural deafness** (missing melanocytes in the stria vascularis), not caused by W/W specifically.

---

## 🟠 7. Default eye colors are wrong for Sepia and oversimplified for normal coats

- **Burmese Sepia `cb/cb`** currently falls through to "Emerald Green". Burmese sepia cats have **gold/yellow** eyes. Add a Sepia branch with a gold iris (`#eab308`).
- **Standard pigmented coats** are all shown as "Emerald Green". In non-pointed, non-white cats, eye color is **polygenic** and ranges from green through hazel and yellow to copper. Suggest a neutral "Gold/Green (polygenic)" label, or a random draw from {green, hazel, gold, copper} that is marked as not genotype-determined.
- **Albino:** see #9.

---

## 🟠 8. "Hardy–Weinberg" probabilities are not Hardy–Weinberg consistent

**Where:** comment at html L1143 ("Realistic Hardy-Weinberg diploid genotype probabilities") and `LOCUS_PROBS`.

These loci are internally consistent (genotype frequencies match an allele frequency): *TYRP1*, *MLPH*, *ASIP*, and `Ws`. These are not:

| Locus | Listed | Hardy–Weinberg check |
| :--- | :--- | :--- |
| *TYR* | C/C 0.49, C/cb 0.10, C/cs 0.16, C/c 0.03, cb/cb 0.05, cs/cs 0.09, c/c 0.01 … | C/C = 0.49 means f(C) = 0.70, so f(cb) ≈ 0.071, f(cs) ≈ 0.114, f(c) ≈ 0.021 (sum only 0.91). Expected cb/cb ≈ 0.005 (listed 10× higher), cs/cs ≈ 0.013 (7× higher), c/c ≈ 0.0005 (22× higher) |
| Inhibitor | i/i 0.86, I/i 0.11, I/I 0.03 | I/I = 0.03 means f(I) ≈ 0.17, so expected I/i ≈ 0.29, not 0.11 |
| *KIT* W | w/w 0.94, W/w 0.05, W/W 0.01 | W/W = 0.01 means f(W) = 0.10, so expected W/w = 0.18, w/w = 0.81 |
| Tabby | Mc/Mc 0.20, Mc/mc 0.28, mc/mc 0.24, … | Not derived from any allele frequencies |

**Fix:** Define **allele frequencies** per locus and compute genotype probabilities as p², 2pq, q² (and for multi-allele loci, pᵢ² and 2pᵢpⱼ). The code then becomes self-consistent automatically. Also say on the page that the numbers are a **teaching "dice pool"**, not real population frequencies. For example, `b`/`bl` at 1/3 and 1/6 and `cb`/`cs` at about 10% each are far higher than in random-bred cats.

---

## 🟠 9. Albino allele is oversimplified

**Where:** `TYR` options (`c/c`), eye label "Pink-Blue Albino Eyes".

**Problem:** Cats have **two** albino *TYR* alleles (Imes et al. 2006): **`ca`** gives a **blue-eyed albino**, and **`c`** gives a **pink-eyed (full) albino**. Dominance is `C` > `cb` = `cs` > `ca` > `c`. "Pink-blue" mixes the two.

**Fix:** Either add `ca` (giving 15 *TYR* genotypes instead of 9), or keep one albino allele and label it clearly as "`c` (pink-eyed albino)" with a pale pink/red iris. Also, `cs/c` probably gives a pointed cat with paler points; that is acceptable but worth a note.

---

## 🟠 10. Klinefelter (XXY) frequency is overstated about 25×

**Where:** `LOCUS_PROBS.chromosomes` (`"XXY": 0.008`), gene card 1 (L2104, "0.8%"), README row 1.

**Problem:** Male tortoiseshell/calico cats occur in roughly **1 in 3,000** tortie/calico cats, and most of them are XXY or chimeras/mosaics. Putting 0.8% of all rolls into XXY makes male calicos far too common.

**Fix:** Lower XXY to about 0.03–0.05%, or keep 0.8% as a deliberate "game boost" and label it that way. Also mention **chimerism** (XX/XY) as the other common cause of male torties.

---

## 🟠 11. Silver `I/I` is mislabeled "Wide-Band"

**Where:** html L2042 (`hint: "Wide-Band Silver"`), L2161 (`"homozygous Wide-Band I/I"`).

**Problem:** **Wide-band** is a separate, hypothesized trait (golden/"shaded" depth of ticking; *CORIN* variants are linked to golden/"sunshine" coats). It is not homozygous Inhibitor. The Inhibitor gene itself is still **unidentified**, so do not present it as a named "Melanin Inhibitor gene".

**Fix:** Change `I/I` to "Homozygous Silver/Smoke (I/I)" and add "(gene not yet identified)" to card 8 and README row 8.

---

## 🟡 12. Smaller wording and naming fixes

| Item | Current | Suggested |
| :--- | :--- | :--- |
| "Sex-linked Black" (`X^o`) | Presented almost as its own black gene | Say "`o` = wild-type **non-orange** allele". Black itself comes from *TYRP1* `B`, and there is no separate sex-linked black gene. The note at L1061 mostly says this already; keep the wording consistent everywhere. |
| `X^O` mechanism | "blocks dark eumelanin synthesis" | Add: caused by a ~5 kb deletion that raises *ARHGAP36* expression in melanocytes, switching them to pheomelanin (Kaelin et al. 2025; Sasaki et al. 2025). |
| XXY display | `X^O/X^o/Y` built by appending `/Y` | Fine, but write it as `X^O X^o Y` to avoid suggesting three alleles at one locus. |
| Dilute tortie names | e.g. "Dilute Lilac Tortoiseshell" | Standard names: Blue-Cream, Lilac-Cream, Fawn-Cream. |
| Tortie + low white `Ws/w+` | Called "Calico" | Breed standards: low white = "Tortie-and-White"; Calico usually means substantial white (`Ws/Ws`-like). |
| `Ws/w` pointed | "& White (Snowshoe)" | OK, but Snowshoe also needs a specific white pattern; "Pointed & White" is safer. |
| `a/a` card | "keeps MC1R continuously active" | Correct in spirit (no ASIP antagonism of MC1R). Fine. |
| Chromosome numbers | *TYRP1* D4, *MLPH* C1, *ASIP* A3, *TYR* D1, *KIT* B1 | ✅ Correct. Add *Taqpep* A1 and *DKK4* B1 when splitting tabby. |

---

## 🟡 13. Optional color genes not yet included (for "all color genes" completeness)

| Gene / Locus | Alleles | Effect |
| :--- | :--- | :--- |
| ***MC1R* (Extension)** | `E`, `e` (Amber, Norwegian Forest), `er` (Russet, Burmese) | Eumelanin gradually turns to red/amber with age |
| ***CORIN*** | wild type vs. golden/"sunshine" variants | Golden undercoat (Siberian "sunshine", golden chinchilla) |
| ***KIT* gloving** | `wg` (two linked missense variants) | Birman white gloves |
| **Salmiak** | deletion downstream of *KIT* | Salmiak pattern |
| ***ASIP* `APb`** | Charcoal (from Asian leopard cat) | Bengal charcoal |
| ***TYR* Mocha `cm`** | Burmese mocha | Further colorpoint allele |
| **Dilute modifier `Dm`** | hypothetical | "Caramel" on dilutes |

---

## Recommended corrected locus model

| # | Locus | Genotypes |
| :--- | :--- | :--- |
| 1–2 | Sex + *ARHGAP36* (X) | 8 (as now) |
| 3 | *TYRP1* | 6 |
| 4 | *MLPH* | 3 |
| 5 | *ASIP* | 3 |
| 6a | *Taqpep* (A1) | 3 |
| 6b | *DKK4* Ticked (B1) | 3 |
| 7 | *TYR* | 9 (or 15 with `ca`) |
| 8 | Inhibitor | 3 |
| 9 | ***KIT* (merged W/Ws/w+)** | **6** |

$$8 \times 6 \times 3 \times 3 \times 3 \times 3 \times 9 \times 3 \times 6 = 629{,}856 \text{ genotype combinations}$$

(With the blue-eyed albino `ca` added to *TYR*: 1,049,760. With an optional 2-state Spotted modifier, double either number.)

After restructuring, update `LOCUS_OPTIONS`, `LOCUS_KEYS_ORDERED`, `LOCUS_PROBS`, `rollCatState`, `buildCatFromState`, the `precomputeAllProbabilities` groups, the switcher and gene cards, all "734,832" strings, and the README tables. Then re-verify that the genotype count matches and the probabilities still sum to 1.

---

## Priority order for fixes

1. 🔴 #2: fix the README FERV1 reversal and the citation (text only, quick).
2. 🔴 #3: fix the horse/*EDNRB* comparison (text only, quick).
3. 🔴 #5 and 🟠 #11: fix the eye-label bug and the "Wide-Band" mislabel (one-line fixes).
4. 🔴 #1: merge *KIT* into one locus (structural).
5. 🔴 #4: split *Taqpep* / *DKK4*, and make Spotted a labeled modifier (structural).
6. 🟠 #6–#10: probabilistic eye color, Sepia eye color, Hardy–Weinberg-derived frequencies, albino alleles, XXY rate.
7. 🟡 #12–#13: naming polish and optional extra genes.

---

## References

- David VA *et al.* (2014). Endogenous retrovirus insertion in the *KIT* oncogene determines White and White spotting in domestic cats. ***G3*** 4(10):1881–1891.
- Kaelin CB *et al.* (2012). Specifying and sustaining pigmentation patterns in domestic and wild cats (*Taqpep*). *Science* 337:1536–1541.
- Kaelin CB, McGowan KA, Barsh GS (2021). Developmental genetics of color pattern establishment in cats (*DKK4*, Ticked). *Nature Communications* 12:5127.
- Imes DL *et al.* (2006). Albinism in the domestic cat is associated with a *tyrosinase* (*TYR*) mutation. *Animal Genetics* 37:175–178.
- Lyons LA *et al.* (2005). *Tyrosinase* mutations associated with Siamese and Burmese patterns in the domestic cat. *Animal Genetics* 36:119–126.
- Ishida Y *et al.* (2006). A homozygous single-base deletion in *MLPH* causes the dilute coat color phenotype in the domestic cat. *Genomics* 88:698–705.
- Eizirik E *et al.* (2003). Molecular genetics and evolution of melanism in the cat family (*ASIP*). *Current Biology* 13:448–453.
- Kaelin CB *et al.* (2025) and Sasaki H *et al.* (2025). *ARHGAP36* deletion underlies X-linked Orange in cats. *Current Biology*.
- Peterschmitt M *et al.* (2009). Amber (*MC1R*) in Norwegian Forest cats. *Animal Genetics* 40:547–552.

