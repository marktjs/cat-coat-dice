# 🎲 Cat Coat Dice — Realistic Feline Coat Genetics Simulator

🏷️ **Version:** 1.1  
🌐 **Live Web App:** https://marktjs.github.io/cat-coat-dice/  
📦 **GitHub Repository:** https://github.com/marktjs/cat-coat-dice  
📄 **Source File:** `cat-coat-dice.html`

**Cat Coat Dice** is a self-contained, interactive feline coat genetics simulator and exact analytical probability calculator. Players can roll the genetic dice to generate random domestic cats (*Felis catus*), jump directly to any of the **629,856 numbered diploid genotype combinations (`#1` – `#629,856`)**, or manually toggle molecularly validated diploid alleles across **10 canonical coat color and pattern loci** directly beneath the cat illustration.

---

## 1. Webpage Design & Architecture

The simulator is built as a **zero-dependency, single-file web application** (`cat-coat-dice.html`) combining semantic HTML5, responsive CSS Grid/Flexbox, procedural SVG illustration, and an exact analytical Hardy–Weinberg probability engine.

### A. Bijective Genotype Combination Numbering (`#1` to `#629,856`)
Every possible diploid state across all 10 confirmed feline loci is assigned a unique, deterministic **Genotype Combination Number** via `encodeGenotypeNumber(state)` and `decodeGenotypeNumber(num)`:
$$\text{Total Diploid Combinations} = 8_{\text{Sex + X}^{O/o}} \times 6_{\text{TYRP1}} \times 3_{\text{MLPH}} \times 3_{\text{ASIP}} \times 3_{\text{Taqpep}} \times 3_{\text{DKK4}} \times 9_{\text{TYR}} \times 3_{\text{Inhibitor}} \times 6_{\text{KIT}} = 629,856$$
- **Interactive Genotype `#` Jump Box (`#comboJumpInput`):** Type any number from `1` to `629856` in the summary header to decode and inspect that exact 10-locus genotype.
- **Per-Phenotype Genotype Counts (`COAT_COMBO_COUNT_MAP`):** Each of the **1,335 distinct visual coat phenotypes** displays both its exact roll probability and how many of the 629,856 diploid genotypes produce that phenotype (summing to all `629,856 / 629,856` genotypes and `100.00%` total probability).

### B. Layered Procedural SVG Cat Renderer (`renderCatSVG`)
Instead of static image sprites, the cat is drawn dynamically using a multi-layered vector graphics pipeline inside a `240×240` SVG canvas:
1. **Silhouette Clipping Masks (`#cuteCatBodyClip` & `#cuteCatTailClip`):**
   - Separate vector `<clipPath>` definitions for the sitting torso/head/ears and the curled tail ensure that complex coat patterns (tortoiseshell patches, tabby stripes, tuxedo bibs, and piebald spots) hug the cat's contours cleanly.
   - Separating the tail clip path from the body allows temperature-sensitive colorpoint cats (Siamese, Mink, Sepia) and high-white piebald cats (`Ws/Ws`) to keep a distinctly pigmented, striped tail while the torso remains pale or white.
2. **Composited Genetic Coat Layers (Bottom-to-Top):**
   - **Layer 1 — Base Coat & Temperature Shading:** Fills the torso with the computed eumelanin/pheomelanin shade, silver undercoat (`I/i`, `I/I`), or temperature-sensitive pale body (`cs/cs`, `cs/c`, `cb/cs`, `cb/cb`).
   - **Layer 2 — Smoke Undercoat Radial Gradient (`#smokeUndercoat`):** Renders a silvery-white root glow on non-agouti smoke cats (`a/a` with `I/i` or `I/I`).
   - **Layer 3 — Colorpoint Extremities:** Applies dark pigment to cooler body regions (ears, central face mask, front paws, and tail) when temperature-sensitive *TYR* alleles (`cs`, `cb`) are active.
   - **Layer 4 — Tortoiseshell / Calico Mosaic Patches:** Overlays asymmetric orange/cream X-inactivation patches (`X^O / X^o`) across the head, back, and tail.
   - **Layer 5 — Tabby Pattern Geometry (`renderTabbyOverlay`):** Draws the classic forehead `"M"` mark, cheek swirls, tail rings, and torso patterns governed by ***Taqpep* (`LVRN`, Chr A1)** (**Mackerel** vertical stripes vs. **Classic** blotched whorls), **epistatic *DKK4* (Chr B1)** (**Ticked** Abyssinian agouti flecking—distinguishing clean homozygous `TiA/TiA` from residual leg-bar heterozygous `TiA/Ti+`), or the interactive **Polygenic Spotted Modifier** (**Spotted** oval rosettes).
   - **Layer 6 — White Spotting (`Ws` at *KIT*):** Adds either a low-white tuxedo chest bib, white muzzle, and white mittens (`Ws/w+`) or a high-white piebald/Van coat with a facial blaze (`Ws/Ws`).
   - **Layer 7 — Kawaii Facial Features & Realistic Eye Genetics:** Crisp dark outlines, pink inner ears, rosy blush cheeks, whiskers, a pink triangle nose with a `:3` smile, and dual-highlight eyes whose left and right iris colors reflect *KIT*, *TYR*, and polygenic eye pigmentation:
     - **Sky-Blue Eyes (`#38bdf8`) & Odd-Eyed Heterochromia (`#38bdf8` + `#eab308`):** Expressed with incomplete penetrance in Dominant White (`W/w+`, `W/Ws`, `W/W`) and high-white piebald (`Ws/Ws`) cats, and consistently in Siamese pointed (`cs/cs`, `cs/c`) cats.
     - **Aquamarine Eyes (`#06b6d4`):** Expressed in Tonkinese Mink (`cb/cs`) cats.
     - **Golden-Yellow Eyes (`#eab308`):** Expressed in Burmese Sepia (`cb/cb`) cats.
     - **Pink-Red Albino Eyes (`#fb7185`):** Expressed in recessive Tyrosinase-negative Pink-Eyed Albino (`c/c`) cats.
     - **Gold / Green / Copper Eyes (Polygenic):** Expressed in standard pigmented coats, with an interactive modifier bar to preview stochastic *KIT* penetrance or custom iris colors.

### C. Cellphone & Mobile Browser Optimizations
- **Sticky Live Mini-Cat Header (`#stickyCatBar`):** Uses an `IntersectionObserver` on the primary cat preview card so that when a mobile user scrolls down into the 10-locus gene switcher or the probability directory, a blurred glassmorphic top bar pins to the screen with a live `44×44` mini SVG cat, the coat name, `Genotype #`, sex, probability badge, and a quick `🎲 Roll` button.
- **Zero Horizontal Overflow (`390×844` Audited):** All CSS Grid containers use `minmax(0, 1fr)` with wrapping genotype pills, ensuring `scrollWidth === innerWidth` across small phone viewports.

---

## 2. The Feline Coat Genetics Behind the Simulator

### A. Merged Tri-Allelic *KIT* Locus (`W > Ws > w+`), FERV1 Insertions & Incomplete Penetrance
1. **Why `W` and `Ws` Are Alleles of One *KIT* Locus (David et al., *G3* 2014):**
   - Both **Dominant White (`W`)** and **White Spotting (`Ws`)** map to the exact same region in **intron 1 of the *KIT* proto-oncogene on feline chromosome B1** (David et al. 2014, *G3: Genes|Genomes|Genetics* 4:1881–1891).
   - **`Ws` (White Spotting):** Caused by the insertion of a **full-length (7,125 bp) feline endogenous retrovirus (FERV1)** element into *KIT* intron 1.
   - **`W` (Dominant White):** Caused by a **~620 bp solo long terminal repeat (LTR)** of the same FERV1 element (generated by homologous recombination between the two LTRs of the full-length insertion).
   - Together with wild-type **`w+`**, they form a single **tri-allelic locus** (`W > Ws > w+`) with **6 possible diploid genotypes**: `w+/w+`, `Ws/w+`, `Ws/Ws`, `W/w+`, `W/Ws`, and `W/W`.
2. **Why `W/W` Is Viable in Cats (Unlike Many Mouse *Kit* and Horse *KIT* Alleles):**
   - Unlike many murine *Kit* (`W`) loss-of-function mutations and several equine *KIT* dominant-white alleles (which are embryonic lethal, sterile, or anemic when homozygous—note that equine Frame Overo Lethal White Foal Syndrome is caused by a different gene, *EDNRB*), the feline FERV1 insertion disrupts a tissue-specific DNase I hypersensitive regulatory site in *KIT* intron 1 without knocking out the *KIT* coding sequence needed for hematopoiesis and gametogenesis. Consequently, **homozygous `W/W` cats are viable**.
3. **Incomplete Penetrance for Blue / Odd Eyes & Sensorineural Deafness:**
   - All `W` genotypes (`W/w+`, `W/Ws`, `W/W`) have complete penetrance for a pure white coat, but **incomplete penetrance** for iris hypopigmentation (two sky-blue eyes, one blue and one yellow/amber/green eye, or two pigmented eyes) and congenital sensorineural deafness (due to missing melanocytes in the cochlear *stria vascularis*), with penetrance highest in `W/W` homozygotes. High-white spotting (`Ws/Ws`) similarly increases the probability of odd or blue eyes.

### B. Split Tabby Loci: *Taqpep* (Chr A1) vs. Epistatic *DKK4* Ticked (Chr B1)
- **Primary Tabby Pattern (*Taqpep* / *LVRN*, Chr A1):** Determines whether non-ticked agouti cats display **Mackerel** vertical tiger stripes (`Mc/Mc`, `Mc/mc`, often written `Ta^M`) or **Classic/Blotched** bullseye whorls (`mc/mc`, `Ta^b`) (Kaelin et al. 2012, *Science*).
- **Ticked Agouti Epistasis (*DKK4*, Chr B1):** Encodes Dickkopf 4, a Wnt signaling antagonist in a reaction–diffusion pre-patterning system (Kaelin et al. 2021, *Nature Communications*). Dominant **`Ti^A`** (`TiA/Ti+` or `TiA/TiA`) is **epistatic to *Taqpep***, suppressing torso stripes while the cat still carries and transmits its underlying `Mc`/`mc` genotype (`TiA/Ti+` retains residual leg/tail bars; `TiA/TiA` is nearly stripeless).
- **Spotted Tabby (Polygenic Modifier):** No single Mendelian `Sp` gene has been identified; spotted tabbies arise from modifier loci that break *Taqpep* mackerel bars into oval rosettes. The simulator provides an interactive **Spotted Modifier** toggle above the gene switcher.

### C. X-Linked Non-Orange (`X^o`) vs. Autosomal Black (*TYRP1* `B`)
- **The X-Chromosome Switch (*ARHGAP36*, Chr X):** A ~5 kb deletion on the X chromosome (`X^O`) causes ectopic expression of *ARHGAP36* in melanocytes, switching pigment synthesis to red/yellow **pheomelanin** (Kaelin et al. 2025; Sasaki et al. 2025). The wild-type **non-orange allele (`X^o`)** permits dark **eumelanin** synthesis.
- **The Autosomal Eumelanin Gene (*TYRP1*, Chr D4):** Whenever a melanocyte has an active `X^o` allele, *TYRP1* sets the eumelanin granule structure: **Black (`B`) > Chocolate (`b`) > Cinnamon (`bl`)**.

---

### Complete 10-Locus Reference Table (Hardy–Weinberg Teaching Pool)

All autosomal diploid genotype probabilities are derived analytically ($p_i^2$ and $2p_i p_j$) from explicit teaching-pool allele frequencies:

| # | Locus & Symbol | Molecular Gene & Chr | Diploid Genotypes & Dominance | Teaching-Pool Allele Frequencies & HWE Genotype Probabilities |
| :--- | :--- | :--- | :--- | :--- |
| **1** | **Sex Chromosomes** | Karyotype (`X`, `Y`) | `XX` (Female), `XY` (Male), `XXY` (Klinefelter Male; note `XX/XY` chimerism also causes male torties) | `XX`: 49.98%<br>`XY`: 49.98%<br>`XXY`: 0.04% (~1 in 2,500) |
| **2** | **X-Linked Orange vs. Non-Orange** (`O` Locus) | *ARHGAP36* (Chr X) | `X^O` (~5 kb deletion, Orange) co-dominant with wild-type `X^o` (Non-Orange) via X-inactivation | $f(X^o)=0.50,\; f(X^O)=0.50$<br>**XX / XXY:** `o/o` 25%, `O/o` 50%, `O/O` 25%<br>**XY:** `o/Y` 50%, `O/Y` 50% |
| **3** | **Eumelanin Series** (`B` Locus) | *TYRP1* (Chr D4) | `B` (Black) > `b` (Chocolate) > `bl` (Cinnamon) (`B/B`, `B/b`, `B/bl`, `b/b`, `b/bl`, `bl/bl`) | $f(B)=\tfrac{3}{6},\; f(b)=\tfrac{2}{6},\; f(bl)=\tfrac{1}{6}$<br>`B/B`: 25.00%, `B/b`: 33.33%, `B/bl`: 16.67%<br>`b/b`: 11.11%, `b/bl`: 11.11%, `bl/bl`: 2.78% |
| **4** | **Melanosome Dilution** (`D` Locus) | *MLPH* (Chr C1) | `D` (Dense) > `d` (Dilute: Blue, Lilac, Fawn, Cream) (`D/D`, `D/d`, `d/d`) | $f(D)=\tfrac{2}{3},\; f(d)=\tfrac{1}{3}$<br>`D/D`: 44.44%, `D/d`: 44.44%, `d/d`: 11.11% |
| **5** | **Agouti Signaling** (`A` Locus) | *ASIP* (Chr A3) | `A` (Agouti banded) > `a` (Non-agouti solid; `X^O` is epistatic to `a/a`) (`A/A`, `A/a`, `a/a`) | $f(A)=0.50,\; f(a)=0.50$<br>`A/A`: 25.00%, `A/a`: 50.00%, `a/a`: 25.00% |
| **6** | **Tabby Stripe Pattern** (`Mc` Locus) | *Taqpep* / *LVRN* (Chr A1) | `Mc` (`Ta^M`, Mackerel stripes) > `mc` (`Ta^b`, Classic whorls) (`Mc/Mc`, `Mc/mc`, `mc/mc`) | $f(Mc)=0.60,\; f(mc)=0.40$<br>`Mc/Mc`: 36.00%, `Mc/mc`: 48.00%, `mc/mc`: 16.00% |
| **7** | **Ticked Agouti Epistasis** (`Ti` Locus) | *DKK4* (Chr B1) | `Ti^A` (Ticked, epistatic to *Taqpep*) > `Ti^+` (Non-ticked) (`Ti+/Ti+`, `TiA/Ti+`, `TiA/TiA`) | $f(Ti^+)=0.90,\; f(Ti^A)=0.10$<br>`Ti+/Ti+`: 81.00%, `TiA/Ti+`: 18.00%, `TiA/TiA`: 1.00% |
| **8** | **Colorpoint & Albinism Series** (`C` Locus) | *TYR* (Chr D1) | `C` > `cb` $\ge$ `cs` > `c` (`C/C`, `C/cb`, `C/cs`, `C/c`, `cb/cb`, `cb/cs`, `cs/cs`, `cs/c`, `c/c` Pink-Eyed Albino; note `c^a` Blue-Eyed Albino) | $f(C)=0.70,\; f(cs)=0.16,\; f(cb)=0.10,\; f(c)=0.04$<br>`C/C`: 49.00%, `C/cb`: 14.00%, `C/cs`: 22.40%, `C/c`: 5.60%<br>`cb/cb`: 1.80%, `cb/cs`: 3.20%, `cs/cs`: 2.56%, `cs/c`: 1.28%, `c/c`: 0.16% |
| **9** | **Melanin Inhibitor** (`I` Locus) | Unidentified Gene | `I` (Silver / Smoke / Cameo undercoat) > `i` (Normal) (`i/i`, `I/i`, `I/I` Homozygous Silver/Smoke) | $f(i)=0.90,\; f(I)=0.10$<br>`i/i`: 81.00%, `I/i`: 18.00%, `I/I`: 1.00% |
| **10** | **White Spotting & Dominant White** (`W` Locus) | *KIT* Intron 1 FERV1 (Chr B1) | Tri-allelic: `W` (~620 bp FERV1 LTR, Dom. White) > `Ws` (7,125 bp FERV1, White Spotting) > `w+` (Wild-type) (`w+/w+`, `Ws/w+`, `Ws/Ws`, `W/w+`, `W/Ws`, `W/W`) | $f(w^+)=0.65,\; f(W^s)=0.30,\; f(W)=0.05$<br>`w+/w+`: 42.25%, `Ws/w+`: 39.00%, `Ws/Ws`: 9.00%<br>`W/w+`: 6.50%, `W/Ws`: 3.00%, `W/W`: 0.25% |

---

## 3. Epistasis & Masking Hierarchy

```
1. Dominant White (KIT W/w+, W/Ws, W/W)
   └── Masks ALL other coat loci → Pure White coat (incomplete penetrance for blue or odd eyes & deafness; highest in W/W)
       │
2. Recessive Tyrosinase Pink-Eyed Albino (TYR c/c)
   └── Halts all melanin synthesis → Pink-Eyed Albino White coat (distinct from TYR c^a/c^a blue-eyed albino)
       │
3. White Spotting (KIT Ws/w+, Ws/Ws)
   ├── Ws/Ws → High-White Piebald / Van coat + increased penetrance for Odd or Blue Eyes
   └── Ws/w+ → Low-White Tuxedo bib, mittens & muzzle
       │
4. X-Linked Orange Switch (ARHGAP36 X^O vs. X^o)
   ├── X^O active: Masks autosomal B locus (Black/Choc/Cinnamon) AND masks solid (a/a)
   │               → Always expresses Orange/Cream with visible Tabby striping
   └── X^o active: Permits autosomal B locus (Black/Choc/Cinnamon) + Agouti/Solid (A vs. a)
                   │
5. Ticked Agouti Epistasis (DKK4 Ti^A vs. Ti^+)
   ├── TiA/TiA or TiA/Ti+ → Epistatic to Taqpep: suppresses torso stripes into Abyssinian Ticked coat
   └── Ti+/Ti+            → Reveals underlying Taqpep (Mc/– Mackerel or mc/mc Classic) pattern
```

---

## 4. Version Log (Changelog)

### Version 1.1 (Current)
- **Merged Tri-Allelic *KIT* Locus (`W > Ws > w+` on Chr B1):** Combined `Ws` and `W` into a single 6-genotype *KIT* locus (`w+/w+`, `Ws/w+`, `Ws/Ws`, `W/w+`, `W/Ws`, `W/W`), eliminating biologically impossible 4-allele combinations.
- **Corrected FERV1 Insertion Sizes & Citation:** Documented that `Ws` (White Spotting) is the **full-length 7,125 bp FERV1** insertion in *KIT* intron 1 and `W` (Dominant White) is the **~620 bp solo LTR** insertion, citing **David et al. (2014), *G3: Genes|Genomes|Genetics* 4(10):1881–1891**.
- **Corrected Mouse/Horse *KIT* Comparison:** Removed equine Frame Overo (*EDNRB*) from the *KIT* comparison and contrasted feline `W/W` viability with homozygous-lethal/sterile mouse *Kit* (`W`) and equine *KIT* alleles.
- **Split Tabby into *Taqpep* (Chr A1) and Epistatic *DKK4* Ticked (Chr B1):** Separated Mackerel/Classic (`Mc/Mc`, `Mc/mc`, `mc/mc`) from Ticked (`Ti+/Ti+`, `TiA/Ti+`, `TiA/TiA`), and moved Spotted tabby to an interactive **Polygenic Spotted Modifier** toggle.
- **Exact Hardy–Weinberg Equilibrium (HWE) Teaching-Pool Probabilities:** Derived all autosomal genotype probabilities analytically ($p_i^2, 2p_ip_j$) from explicit allele frequencies and adjusted `XXY` Klinefelter frequency to a realistic `0.04%` (~1 in 2,500 births), resulting in **629,856** numbered diploid combinations across **1,335** coat phenotypes.
- **Eye Genetics & Nomenclature Refinements:** Fixed the `W/w+` + `cs/cs` eye badge label, added **Golden-Yellow Eyes** for Burmese Sepia (`cb/cb`), labeled `c/c` as **Pink-Eyed Albino** (noting `c^a` blue-eyed albino), added interactive **Incomplete Penetrance / Polygenic Eye Controls**, corrected `I/I` from "Wide-Band" to **Homozygous Silver/Smoke (`I/I`)**, formatted `XXY` as `X^O X^o Y`, and adopted standard dilute tortie names (**Blue-Cream**, **Lilac-Cream**, **Fawn-Cream**).

### Version 1.0
- Initial stable release with bijective genotype combination numbering, layered procedural SVG cat illustration, sticky mobile mini-cat bar, and interactive 10-locus switcher.

---

## 5. Key Scientific References

- David VA *et al.* (2014). Endogenous retrovirus insertion in the *KIT* oncogene determines White and White spotting in domestic cats. ***G3: Genes|Genomes|Genetics*** 4(10):1881–1891.
- Kaelin CB *et al.* (2012). Specifying and sustaining pigmentation patterns in domestic and wild cats (*Taqpep* / *LVRN*). ***Science*** 337:1536–1541.
- Kaelin CB, McGowan KA, Barsh GS (2021). Developmental genetics of color pattern establishment in cats (*DKK4*, Ticked). ***Nature Communications*** 12:5127.
- Kaelin CB *et al.* (2025) & Sasaki H *et al.* (2025). *ARHGAP36* deletion underlies X-linked Orange in domestic cats. ***Current Biology***.
- Lyons LA *et al.* (2005). *Tyrosinase* mutations associated with Siamese and Burmese patterns in the domestic cat. ***Animal Genetics*** 36:119–126.
- Imes DL *et al.* (2006). Albinism in the domestic cat is associated with a *tyrosinase* (*TYR*) mutation. ***Animal Genetics*** 37:175–178.
- Ishida Y *et al.* (2006). A homozygous single-base deletion in *MLPH* causes the dilute coat color phenotype in the domestic cat. ***Genomics*** 88:698–705.
- Eizirik E *et al.* (2003). Molecular genetics and evolution of melanism in the cat family (*ASIP*). ***Current Biology*** 13:448–453.

---

## 6. Running Locally

No build tools or servers are required. Open `cat-coat-dice.html` directly in any modern browser:

```bash
open random_feline_coat/cat-coat-dice.html
```
