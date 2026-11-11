# 🎲 Cat Coat Dice — Complete Feline Coat Genetics Simulator

🌐 **Live Web App:** https://marktjs.github.io/cat-coat-dice/  
📦 **GitHub Repository:** https://github.com/marktjs/cat-coat-dice  
📄 **Source File:** `cat-coat-dice.html`

**Cat Coat Dice** is a self-contained, interactive feline coat genetics simulator and probability calculator. Players can roll the genetic dice to generate random domestic cats (*Felis catus*) or manually toggle alleles across **10 canonical coat color and pattern loci** directly beneath the cat illustration to see how X-linked and autosomal chromosomes interact in real time.

---

## 1. Webpage Design & Architecture

The simulator is built as a **zero-dependency, single-file web application** (`cat-coat-dice.html`) combining semantic HTML5, modern responsive CSS Grid/Flexbox, procedural SVG illustration, and an exact analytical probability engine.

### A. Layered Procedural SVG Cat Renderer (`renderCatSVG`)
Instead of relying on static image sprites (which could never cover thousands of genotype combinations), the cat is drawn dynamically using a multi-layered vector graphics pipeline inside a `240×240` SVG canvas:
1. **Silhouette Clipping Masks (`#cuteCatBodyClip` & `#cuteCatTailClip`):**
   - Separate vector `<clipPath>` definitions for the sitting torso/head/ears and the curled tail ensure that complex coat patterns (tortoiseshell patches, tabby stripes, tuxedo bibs, and piebald spots) hug the cat's contours cleanly without bleeding across boundaries.
   - Separating the tail clip path from the body allows temperature-sensitive colorpoint cats (Siamese, Mink, Sepia) and high-white piebald cats (`Ws/Ws`) to keep a distinctly pigmented, striped tail while the torso remains pale or white.
2. **Composited Genetic Coat Layers (Bottom-to-Top):**
   - **Layer 1 — Base Coat & Temperature Shading:** Fills the torso with the computed eumelanin/pheomelanin shade, silver undercoat (`I/-`), or temperature-sensitive pale body (`cs/cs`, `cb/cs`, `cb/cb`).
   - **Layer 2 — Smoke Undercoat Radial Gradient (`#smokeUndercoat`):** Renders a silvery-white root glow on non-agouti smoke cats (`a/a, I/-`).
   - **Layer 3 — Colorpoint Extremities:** Applies dark pigment to cooler body regions (ears, central face mask, front paws, and tail) when temperature-sensitive *TYR* alleles (`cs`, `cb`) are active.
   - **Layer 4 — Tortoiseshell / Calico Mosaic Patches:** Overlays asymmetric orange/cream X-inactivation patches (`X^O / X^o`) across the head, back, and tail.
   - **Layer 5 — Tabby Pattern Geometry (`renderTabbyOverlay`):** Draws the classic forehead `"M"` mark, cheek swirls, tail rings, and one of four distinct torso patterns (**Mackerel** vertical stripes, **Classic** blotched bullseye whorls, **Spotted** oval rosettes, or **Ticked** agouti flecking). On pointed lynx cats (`cs/cs`), torso stripes automatically drop to ghost opacity (`0.18`) while point stripes stay crisp.
   - **Layer 6 — White Spotting (`Ws`):** Adds either a tuxedo chest bib, white muzzle, and white mittens (`Ws/w`, low white) or a high-white piebald coat with a facial blaze (`Ws/Ws`, high white).
   - **Layer 7 — Kawaii Facial Features & Genetically Linked Eyes:** Crisp dark outlines, pink inner ears, rosy blush cheeks, whiskers, a pink triangle nose with a `:3` smile, and sparkly dual-highlight eyes whose iris colors respond to coat genetics (emerald-green on standard coats, sapphire-blue on Siamese `cs/cs` and Dominant White `W/w`, aquamarine on Tonkinese Mink `cb/cs`, pink-tinted on Albino `c/c`, and heterochromia odd-eyes on high-white `Ws/Ws`).

### B. Real-Time 10-Locus Interactive Switcher (`renderGeneSwitcher`)
Directly beneath the cat preview card, an interactive control grid displays all 10 loci. Each locus shows:
- Its chromosome/gene name and the **exact probability ($P$)** of the currently active genotype at that locus.
- Segmented allele buttons that immediately recompute the cat's phenotype, redraw the SVG with a micro-bounce animation, and update all probability metrics.
- Smart sex-chromosome synchronization: switching between `XY` (1 X chromosome) and `XX` / `XXY` (2 X chromosomes) automatically translates the X-linked orange locus between hemizygous (`X^o/Y`, `X^O/Y`) and diploid (`X^o/X^o`, `X^O/X^o`, `X^O/X^O`) states.

### C. Exact Analytical Probability Engine (`precomputeAllProbabilities`)
Rather than estimating odds by Monte Carlo simulation, the page computes the **exact mathematical probability distribution** across all 10 loci on load:
- By grouping equivalent phenotypic allele classes at each locus, the engine evaluates all **23,040 phenotypic state combinations** in under 5 milliseconds, summing to `100.00%`.
- Three distinct probabilities are displayed for every cat:
  1. **Coat Probability:** The total probability of rolling the current visual coat phenotype across all genotypes and sexes (paired with `~1 in N rolls` odds and a 5-tier rarity badge from *Common* to *✨ Mythic / Ultra Rare*).
  2. **Coat + Sex Probability:** Captures sex-linked rarities—for example, showing why a **Female Calico** (`XX`) is common while a **Male Calico** (`XXY` Klinefelter) is roughly 60× rarer.
  3. **Exact 10-Locus Genotype Probability:** The exact product of the 10 individual locus probabilities:
     $$P(\text{Genotype}) = P(\text{Sex}) \times P(\text{X}^{O/o} \mid \text{Sex}) \times P(B) \times P(D) \times P(A) \times P(Mc) \times P(C) \times P(I) \times P(Ws) \times P(W)$$
- **Interactive Coat Probability Directory:** At the bottom of the page, every distinct coat phenotype is ranked from most common to rarest with visual probability bars and a live search filter. Clicking any coat in the directory loads its representative genotype onto the cat.

### D. Cellphone & Mobile Browser Optimizations
- **Sticky Live Mini-Cat Header (`#stickyCatBar`):** Uses an `IntersectionObserver` on the primary cat preview card so that when a mobile user scrolls down into the 10-locus gene switcher or the probability directory, a blurred glassmorphic top bar pins to the screen with a live `44×44` mini SVG cat, the coat name, sex, probability badge, and a quick `🎲 Roll` button. Players see the cat update on every tap without scrolling back to the top.
- **Zero Horizontal Overflow (`390×844` Audited):** All CSS Grid containers use `minmax(0, 1fr)` / `minmax(min(100%, ...), 1fr)` with wrapping genotype pills, ensuring `scrollWidth === innerWidth` across small phone viewports.
- **Touch Ergonomics & iOS Safari Safeguards:** Allele buttons enforce a `42px` minimum touch target height in balanced 2- and 3-column grids with `touch-action: manipulation` (eliminating 300ms tap delay) and `font-size: 16px` on the filter input (preventing involuntary iOS Safari zoom-on-focus).

---

## 2. The Feline Coat Genetics Behind the Simulator

Domestic cat coat color and pattern are governed by a hierarchy of pigment synthesis, melanosome transport, hair-shaft banding, and embryonic cell migration genes.

### Why "Black" Is Split Across Two Genes (X-Linked vs. Autosomal)
A common question in cat genetics is why "black" appears in sex-linked inheritance problems (`X^O` vs. `X^o`) *and* in autosomal pigment series (`B > b > b1`). In reality, making black fur requires two steps:
1. **The X-Chromosome Switch (`O` Locus, *ARHGAP36*):** Determines *whether* a melanocyte produces dark **eumelanin** (`X^o`, the sex-linked black/dark allele) or switches to red/yellow **pheomelanin** (`X^O`, the sex-linked orange allele).
2. **The Autosomal Eumelanin Gene (`B` Locus, *TYRP1*):** Whenever a melanocyte has an active `X^o` allele, the autosomal *TYRP1* enzyme determines the chemical structure and shape of those dark eumelanin granules: **Black (`B`)**, **Chocolate (`b`)**, or **Cinnamon (`b1`)**.

---

### Complete 10-Locus Reference Table

| # | Locus & Symbol | Molecular Gene | Alleles & Dominance | Biological Mechanism & Phenotypic Effect | Dice Pool Probability |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | **Sex Chromosomes** | Karyotype (`X`, `Y`) | `XX`, `XY`, `XXY` | Standard females (`XX`) have two X chromosomes; males (`XY`) have one. Rare `XXY` (Klinefelter) males have two X chromosomes and can be tortoiseshell/calico. | `XX`: 49.0%<br>`XY`: 50.2%<br>`XXY`: 0.8% |
| **2** | **Sex-Linked Orange vs. Black** (`O` Locus) | *ARHGAP36* (X Chr) | `X^O` (Orange) co-dominant with `X^o` (Black/Dark) via X-inactivation | `X^o/X^o` or `X^o/Y` expresses dark eumelanin. `X^O/X^O` or `X^O/Y` converts pigment to orange pheomelanin. In `X^O/X^o`, random embryonic X-inactivation (**Lyonization**) creates a mosaic of orange and dark patches (**Tortoiseshell / Calico**). | **XX / XXY:**<br>`X^o/X^o`: 25%<br>`X^O/X^o`: 50%<br>`X^O/X^O`: 25%<br>**XY:**<br>`X^o/Y`: 50%<br>`X^O/Y`: 50% |
| **3** | **Black / Chocolate / Cinnamon** (`B` Locus) | *TYRP1* (Chr D4) | `B` > `b` > `b1` | Encodes Tyrosinase-Related Protein 1. `B/-` produces spherical **Black** granules (Seal in pointed cats); `b/b` or `b/b1` produces ovoid **Chocolate** granules; `b1/b1` produces elongated **Cinnamon** granules. | `B/B`: 25.00%<br>`B/b`: 33.33%<br>`B/b1`: 16.67%<br>`b/b`: 11.11%<br>`b/b1`: 11.11%<br>`b1/b1`: 2.78% |
| **4** | **Pigment Dilution** (`D` Locus) | *MLPH* (Melanophilin, Chr C1) | `D` (Dense) > `d` (Dilute) | Recessive `d/d` causes uneven clumping of melanosomes in the hair shaft, scattering light so **Black → Blue (Grey)**, **Chocolate → Lilac**, **Cinnamon → Fawn**, and **Orange → Cream**. | `D/D`: 44.44%<br>`D/d`: 44.44%<br>`d/d`: 11.11% |
| **5** | **Agouti vs. Solid** (`A` Locus) | *ASIP* (Agouti Signaling Protein, Chr A3) | `A` (Agouti) > `a` (Non-agouti solid) | Dominant `A` pulses agouti protein during hair growth, creating pale-and-dark banded hairs (`agouti` background) that reveal tabby patterns. Recessive `a/a` floods eumelanin hairs with solid color (note: `X^O` orange overrides `a/a` and still shows tabby striping). | `A/A`: 25.00%<br>`A/a`: 50.00%<br>`a/a`: 25.00% |
| **6** | **Tabby Pattern** (`Mc`, `Sp`, `Ta` Loci) | *LVRN* (Transmembrane Aminopeptidase Q) & *DKK4* | `Ta` (Ticked) > `Sp` (Spotted) > `Mc` (Mackerel) > `mc` (Classic) | Establishes pre-patterning in skin follicles before pigment is deposited: **Mackerel** (`Mc/-`, vertical tiger bars), **Classic/Blotched** (`mc/mc`, bullseye marble swirls), **Spotted** (`Sp/-`, broken rosettes/ovals), or **Ticked** (`Ta/-`, Abyssinian flecking). | `Mc/Mc`: 22%<br>`Mc/mc`: 28%<br>`mc/mc`: 25%<br>`Sp/mc`: 15%<br>`Ta/mc`: 10% |
| **7** | **Colorpoint & Albinism Series** (`C` Locus) | *TYR* (Tyrosinase, Chr D1) | `C` > `cb` $\ge$ `cs` > `c` | Controls the rate-limiting melanin enzyme tyrosinase. `C/-` gives full body color. `cs/cs` (**Siamese**) and `cb/cb` (**Burmese Sepia**) encode thermolabile tyrosinase that unfolds at core body warmth and only functions on cooler extremities (face, ears, paws, tail); `cb/cs` produces intermediate **Tonkinese Mink**; `c/c` is **True Albino**. | `C/C`: 52%<br>`C/cs`: 26%<br>`cs/cs`: 10%<br>`cb/cb`: 6%<br>`cb/cs`: 5%<br>`c/c`: 1% |
| **8** | **Silver / Smoke Inhibitor** (`I` Locus) | Melanin Inhibitor (`I`) | `I` (Silver/Smoke) > `i` (Normal) | Dominant `I/-` suppresses pigment production at the base of each hair shaft, leaving a bright silvery-white undercoat (**Silver Tabby**, **Cameo / Red Silver Tabby**, or **Smoke** in non-agouti cats). | `i/i`: 88%<br>`I/i`: 12% |
| **9** | **White Spotting / Piebald** (`Ws` / `S` Locus) | *KIT* Retroviral Insertion (Chr B1) | `Ws` (Incomplete dominant over `w`) | Slows the embryonic migration of melanoblasts from the dorsal neural crest toward the belly and paws. `Ws/w` yields **Low White** (white chest bib, paws, and muzzle); `Ws/Ws` yields **High White / Piebald / Van** coats and frequent heterochromia. | `w/w`: 44.44%<br>`Ws/w`: 44.44%<br>`Ws/Ws`: 11.11% |
| **10** | **Dominant White** (`W` Locus) | *KIT* Dominant Allele (Chr B1) | `W` (Epistatic Dominant White) > `w` | Completely blocks melanoblast survival and migration to the skin and often the iris, producing a pure snow-white coat (`W/w`) with sky-blue or odd eyes, masking all other coat genes. | `w/w`: 95%<br>`W/w`: 5% |

---

## 3. Epistasis & Masking Hierarchy

Because multiple genes act along the same developmental pathway (from neural crest cell migration to pigment synthesis and hair banding), upstream genes can mask (**be epistatic to**) downstream genes:

```
1. Dominant White (W/-)
   └── Masks ALL other coat loci → Pure White coat (sky-blue eyes)
       │
2. Recessive Tyrosinase Albino (c/c)
   └── Halts all melanin synthesis → True Albino White coat (pink-blue eyes)
       │
3. White Spotting (Ws/w, Ws/Ws)
   └── Removes melanocytes in patched regions → White bib/mittens or piebald regions
       │
4. X-Linked Orange Switch (X^O vs. X^o)
   ├── X^O active: Masks autosomal B locus (Black/Choc/Cinnamon) AND masks solid (a/a)
   │               → Always expresses Orange/Cream with visible Tabby striping
   └── X^o active: Permits autosomal B locus (Black/Choc/Cinnamon) + Agouti/Solid (A vs. a)
                   → Solid dark coat (if a/a) OR Tabby-patterned dark coat (if A/-)
```

---

## 4. Running Locally

No build tools or servers are required. Open `cat-coat-dice.html` directly in any modern browser:

```bash
open random_feline_coat/cat-coat-dice.html
```

