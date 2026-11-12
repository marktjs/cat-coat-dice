# 🎲 Cat Coat Dice — Realistic Feline Coat Genetics Simulator

🏷️ **Version:** 1.0  
🌐 **Live Web App:** https://marktjs.github.io/cat-coat-dice/  
📦 **GitHub Repository:** https://github.com/marktjs/cat-coat-dice  
📄 **Source File:** `cat-coat-dice.html`

**Cat Coat Dice** is a self-contained, interactive feline coat genetics simulator and exact analytical probability calculator. Players can roll the genetic dice to generate random domestic cats (*Felis catus*), jump directly to any of the **734,832 numbered diploid genotype combinations (`#1` – `#734,832`)**, or manually toggle realistic diploid alleles across **10 canonical coat color and pattern loci** directly beneath the cat illustration.

---

## 1. Webpage Design & Architecture

The simulator is built as a **zero-dependency, single-file web application** (`cat-coat-dice.html`) combining semantic HTML5, responsive CSS Grid/Flexbox, procedural SVG illustration, and an exact analytical probability engine.

### A. Bijective Genotype Combination Numbering (`#1` to `#734,832`)
Every possible diploid state across all 10 feline loci is assigned a unique, deterministic **Genotype Combination Number** via `encodeGenotypeNumber(state)` and `decodeGenotypeNumber(num)`:
$$\text{Total Diploid Combinations} = 8_{\text{Sex + X}^{O/o}} \times 6_{B} \times 3_{D} \times 3_{A} \times 7_{\text{Tabby}} \times 9_{C} \times 3_{I} \times 3_{Ws} \times 3_{W} = 734,832$$
- **Interactive Genotype `#` Jump Box (`#comboJumpInput`):** Users can type any number from `1` to `734832` in the summary header to decode and inspect that exact 10-locus genotype.
- **Per-Phenotype Genotype Counts (`COAT_COMBO_COUNT_MAP`):** Each of the **915 distinct visual coat phenotypes** displays both its exact roll probability and how many of the 734,832 diploid genotypes produce that phenotype (summing to all `734,832 / 734,832` genotypes and `100.00%` total probability).

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
   - **Layer 5 — Tabby Pattern Geometry (`renderTabbyOverlay`):** Draws the classic forehead `"M"` mark, cheek swirls, tail rings, and one of four distinct torso patterns (**Mackerel** vertical stripes, **Classic** blotched bullseye whorls, **Spotted** oval rosettes, or **Ticked** Abyssinian agouti flecking—distinguishing clean homozygous `Ta/Ta` from residual leg-bar heterozygous `Ta/mc`).
   - **Layer 6 — White Spotting (`Ws`):** Adds either a tuxedo chest bib, white muzzle, and white mittens (`Ws/w`, low white) or a high-white piebald/Van coat with a facial blaze (`Ws/Ws`, high white).
   - **Layer 7 — Kawaii Facial Features & Realistic Eye Genetics:** Crisp dark outlines, pink inner ears, rosy blush cheeks, whiskers, a pink triangle nose with a `:3` smile, and dual-highlight eyes whose left and right iris colors respond directly to *KIT* and *TYR* alleles:
     - **Odd-Eyed Heterochromia (1 Sky-Blue `#38bdf8`, 1 Golden-Yellow `#f59e0b`):** Expressed in high-white piebald (`Ws/Ws`) and heterozygous Dominant White (`W/w`) cats.
     - **Two Vivid Sky-Blue Eyes (`#38bdf8`):** Expressed in homozygous Dominant White (`W/W`) and Siamese pointed (`cs/cs`, `cs/c`) cats.
     - **Aquamarine Eyes (`#06b6d4`):** Expressed in Tonkinese Mink (`cb/cs`) cats.
     - **Pink-Blue Albino Eyes (`#f472b6`):** Expressed in recessive Tyrosinase-negative Albino (`c/c`) cats.
     - **Emerald-Green Eyes (`#10b981`):** Expressed in standard pigmented coats.

### C. Real-Time 10-Locus Interactive Switcher (`renderGeneSwitcher`)
Directly beneath the cat preview card, an interactive control grid displays all 10 loci with realistic diploid genotypes (`B/bl`, `Sp/Sp`, `Ta/Ta`, `C/cb`, `C/c`, `cs/c`, `I/I`, `W/W`, etc.), exact per-locus probabilities, and automatic sex-chromosome synchronization between `XY` (`X^o/Y`, `X^O/Y`) and `XX` / `XXY` (`X^o/X^o`, `X^O/X^o`, `X^O/X^O`).

### D. Cellphone & Mobile Browser Optimizations
- **Sticky Live Mini-Cat Header (`#stickyCatBar`):** Uses an `IntersectionObserver` on the primary cat preview card so that when a mobile user scrolls down into the 10-locus gene switcher or the probability directory, a blurred glassmorphic top bar pins to the screen with a live `44×44` mini SVG cat, the coat name, `Genotype #`, sex, probability badge, and a quick `🎲 Roll` button.
- **Zero Horizontal Overflow (`390×844` Audited):** All CSS Grid containers use `minmax(0, 1fr)` with wrapping genotype pills, ensuring `scrollWidth === innerWidth` across small phone viewports.

---

## 2. The Feline Coat Genetics Behind the Simulator

### A. Why Dominant White (*KIT*) Has a Viable `W/W` Genotype & What Causes Odd Eyes
1. **Why `W/W` Exists in Cats (Unlike Horses or Mice):**
   - In horses (Frame Overo / Dominant White) and mice, many dominant *KIT* loss-of-function mutations are embryonic lethal when homozygous (`W/W`).
   - In domestic cats (*Felis catus*), however, **both Dominant White (`W`) and White Spotting (`Ws`) are insertions of a feline endogenous retrovirus (FERV1) in intron 1 of the *KIT* proto-oncogene on chromosome B1** (David et al., *PLoS Genetics* 2014).
   - A **full-length (~7.1 kb) FERV1 insertion** produces the **Dominant White (`W`)** allele, whereas a **truncated (~662 bp LTR) insertion** at the exact same site produces the **White Spotting (`Ws`)** allele. Because this regulatory insertion affects neural-crest melanoblast migration without knocking out hematopoiesis or gametogenesis, **homozygous `W/W` cats are completely viable**.
2. **Why `Ws/Ws` and `W/w` Cause Odd Eyes (1 Blue, 1 Yellow) While `W/W` Gives Two Blue Eyes:**
   - A cat's iris turns **yellow, amber, or green** when neural-crest melanoblasts migrate into the iris stroma and deposit melanin. If melanoblasts fail to reach an iris, the unpigmented stroma scatters short-wavelength light via **Tyndall scattering**, producing a **vivid sky-blue eye** (even though the pigmented retinal epithelium at the back of the eye remains intact, unlike `c/c` albinos).
   - In **high-white piebald (`Ws/Ws`)** and **heterozygous Dominant White (`W/w`)** cats, melanoblast survival is only partially reduced; stochastic developmental noise often blocks melanoblasts from reaching **one eye** while enough reach the **other eye**, causing **complete heterochromia (one blue eye, one yellow/amber eye)**.
   - In **homozygous Dominant White (`W/W`)** cats, the double dose of the `W` allele causes a much more severe depletion of neural-crest melanoblasts—resulting in the highest penetrance of **bilateral sky-blue eyes** (and higher risk of unilateral or bilateral sensorineural deafness due to absent melanocytes in the cochlear *stria vascularis*).

### B. Why "Black" Is Split Across Two Genes (X-Linked vs. Autosomal)
Making black fur requires two sequential genetic steps:
1. **The X-Chromosome Switch (`O` Locus, *ARHGAP36*):** Determines *whether* a melanocyte produces dark **eumelanin** (`X^o`, the sex-linked black/dark allele) or switches to red/yellow **pheomelanin** (`X^O`, the sex-linked orange allele).
2. **The Autosomal Eumelanin Gene (`B` Locus, *TYRP1*):** Whenever a melanocyte has an active `X^o` allele, the autosomal *TYRP1* enzyme determines the chemical structure and shape of those dark eumelanin granules: **Black (`B`)**, **Chocolate (`b`)**, or **Cinnamon (`bl`)**.

---

### Complete 10-Locus Realistic Diploid Reference Table

| # | Locus & Symbol | Molecular Gene | Realistic Diploid Alleles & Dominance | Biological Mechanism & Phenotypic Effect | Dice Pool Probability |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | **Sex Chromosomes** | Karyotype (`X`, `Y`) | `XX`, `XY`, `XXY` | Standard females (`XX`) have two X chromosomes; males (`XY`) have one. Rare `XXY` (Klinefelter) males have two X chromosomes and can be tortoiseshell/calico. | `XX`: 49.0%<br>`XY`: 50.2%<br>`XXY`: 0.8% |
| **2** | **Sex-Linked Orange vs. Black** (`O` Locus) | *ARHGAP36* (X Chr) | `X^O` (Orange) co-dominant with `X^o` (Black/Dark) via X-inactivation | `X^o/X^o` or `X^o/Y` expresses dark eumelanin. `X^O/X^O` or `X^O/Y` converts pigment to orange pheomelanin. In `X^O/X^o`, random embryonic X-inactivation (**Lyonization**) creates a mosaic of orange and dark patches (**Tortoiseshell / Calico**). | **XX / XXY:**<br>`X^o/X^o`: 25%<br>`X^O/X^o`: 50%<br>`X^O/X^O`: 25%<br>**XY:**<br>`X^o/Y`: 50%<br>`X^O/Y`: 50% |
| **3** | **Black / Chocolate / Cinnamon** (`B` Locus) | *TYRP1* (Chr D4) | `B` > `b` > `bl` (`B/B`, `B/b`, `B/bl`, `b/b`, `b/bl`, `bl/bl`) | Encodes Tyrosinase-Related Protein 1. `B/B`, `B/b`, `B/bl` produce spherical **Black** granules; `b/b`, `b/bl` produce ovoid **Chocolate** granules; `bl/bl` produces elongated **Cinnamon** granules. | `B/B`: 25.00%<br>`B/b`: 33.33%<br>`B/bl`: 16.67%<br>`b/b`: 11.11%<br>`b/bl`: 11.11%<br>`bl/bl`: 2.78% |
| **4** | **Pigment Dilution** (`D` Locus) | *MLPH* (Melanophilin, Chr C1) | `D` (Dense) > `d` (Dilute) (`D/D`, `D/d`, `d/d`) | Recessive `d/d` causes uneven clumping of melanosomes in the hair shaft, scattering light so **Black → Blue (Grey)**, **Chocolate → Lilac**, **Cinnamon → Fawn**, and **Orange → Cream**. | `D/D`: 44.44%<br>`D/d`: 44.44%<br>`d/d`: 11.11% |
| **5** | **Agouti vs. Solid** (`A` Locus) | *ASIP* (Agouti Signaling Protein, Chr A3) | `A` (Agouti) > `a` (Solid) (`A/A`, `A/a`, `a/a`) | Dominant `A/A` or `A/a` pulses agouti protein during hair growth, creating banded hairs that reveal tabby patterns. Recessive `a/a` yields solid eumelanin coats (`X^O` orange overrides `a/a`). | `A/A`: 25.00%<br>`A/a`: 50.00%<br>`a/a`: 25.00% |
| **6** | **Tabby Pattern** (`Mc`, `Sp`, `Ta` Loci) | *LVRN* (*Taqpep*) & *DKK4* | `Ta` > `Sp` > `Mc` > `mc` (`Mc/Mc`, `Mc/mc`, `mc/mc`, `Sp/Sp`, `Sp/mc`, `Ta/Ta`, `Ta/mc`) | Establishes follicle pre-patterning: **Mackerel** (`Mc/Mc`, `Mc/mc`), **Classic/Blotched** (`mc/mc`), **Spotted** (`Sp/Sp`, `Sp/mc`), **Homozygous Ticked** (`Ta/Ta`, clean Abyssinian flecking), or **Heterozygous Ticked** (`Ta/mc`, ticked torso with residual leg bars). | `Mc/Mc`: 20%<br>`Mc/mc`: 28%<br>`mc/mc`: 24%<br>`Sp/Sp`: 5%<br>`Sp/mc`: 11%<br>`Ta/Ta`: 4%<br>`Ta/mc`: 8% |
| **7** | **Colorpoint & Albinism Series** (`C` Locus) | *TYR* (Tyrosinase, Chr D1) | `C` > `cb` $\ge$ `cs` > `c` (`C/C`, `C/cb`, `C/cs`, `C/c`, `cb/cb`, `cb/cs`, `cs/cs`, `cs/c`, `c/c`) | Controls tyrosinase. `C/C`, `C/cb`, `C/cs`, `C/c` give full color. `cs/cs` and compound `cs/c` (**Siamese Pointed**) and `cb/cb` (**Burmese Sepia**) encode thermolabile tyrosinase active only on cool extremities; `cb/cs` gives intermediate **Tonkinese Mink**; `c/c` is **True Albino**. | `C/C`: 49%<br>`C/cb`: 10%<br>`C/cs`: 16%<br>`C/c`: 3%<br>`cb/cb`: 5%<br>`cb/cs`: 5%<br>`cs/cs`: 9%<br>`cs/c`: 2%<br>`c/c`: 1% |
| **8** | **Silver / Smoke Inhibitor** (`I` Locus) | Melanin Inhibitor (`I`) | `I` (Silver/Smoke) > `i` (Normal) (`i/i`, `I/i`, `I/I`) | Dominant `I/i` or `I/I` suppresses pigment at the hair base, creating a silvery-white undercoat (**Silver Tabby**, **Cameo Tabby**, or non-agouti **Smoke**). | `i/i`: 86%<br>`I/i`: 11%<br>`I/I`: 3% |
| **9** | **White Spotting / Piebald** (`Ws` Locus) | *KIT* Intron 1 Truncated FERV1 (Chr B1) | `Ws` incomplete dominant over `w` (`w/w`, `Ws/w`, `Ws/Ws`) | Slows embryonic migration of melanoblasts from the neural crest. `Ws/w` yields **Low White** (tuxedo bib, paws, muzzle); `Ws/Ws` yields **High White / Piebald / Van** coats and **Odd-Eyed heterochromia (1 blue, 1 yellow eye)**. | `w/w`: 44.44%<br>`Ws/w`: 44.44%<br>`Ws/Ws`: 11.11% |
| **10** | **Dominant White** (`W` Locus) | *KIT* Intron 1 Full-Length FERV1 (Chr B1) | `W` epistatic dominant over `w` (`w/w`, `W/w`, `W/W`) | Completely blocks melanoblast migration to the skin, producing a snow-white coat that masks all other coat genes. Heterozygous `W/w` produces **Odd-Eyed White** (1 blue, 1 yellow eye), whereas viable homozygous `W/W` produces **Double Sky-Blue-Eyed White**. | `w/w`: 94%<br>`W/w`: 5%<br>`W/W`: 1% |

---

## 3. Epistasis & Masking Hierarchy

Because multiple genes act along the same developmental pathway (from neural crest cell migration to pigment synthesis and hair banding), upstream genes can mask (**be epistatic to**) downstream genes:

```
1. Dominant White (W/W or W/w)
   ├── W/W → Masks ALL other coat loci → Pure White coat with Two Vivid Sky-Blue Eyes
   └── W/w → Masks ALL other coat loci → Pure White coat with Odd Eyes (1 Blue, 1 Yellow)
       │
2. Recessive Tyrosinase Albino (c/c)
   └── Halts all melanin synthesis → True Albino White coat (pink-blue eyes)
       │
3. White Spotting (Ws/w, Ws/Ws)
   ├── Ws/Ws → High-White Piebald / Van coat + Odd Eyes (1 Blue, 1 Yellow, unless pointed cs)
   └── Ws/w  → Low-White Tuxedo bib, mittens & muzzle
       │
4. X-Linked Orange Switch (X^O vs. X^o)
   ├── X^O active: Masks autosomal B locus (Black/Choc/Cinnamon) AND masks solid (a/a)
   │               → Always expresses Orange/Cream with visible Tabby striping
   └── X^o active: Permits autosomal B locus (Black/Choc/Cinnamon) + Agouti/Solid (A vs. a)
                   → Solid dark coat (if a/a) OR Tabby-patterned dark coat (if A/A or A/a)
```

---

## 4. Running Locally

No build tools or servers are required. Open `cat-coat-dice.html` directly in any modern browser:

```bash
open random_feline_coat/cat-coat-dice.html
```
