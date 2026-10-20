Ran command: Find all cppipe pipelines (`find . -name "*.cppipe"`)
Ran command: List modules in all cppipe pipelines (`python3 -c "
import re

pipelines = [
    '1.cellprofiler_ic/pipelines/whole_image_qc.cppipe',
    '1.cellprofiler_ic/pipelines/NF1_illum_3channel.cppipe',
    '1.cellprofiler_ic/pipelines/NF1_illum_4channel.cppipe',
    '2.cellprofiler_analysis/NF1_analysis_3channel.cppipe',
    '2.cellprofiler_analysis/NF1_analysis_4channel.cppipe'
]

for p in pipelines:
    print(f'=== {p} ===')
    with open(p) as f:
        content = f.read()
    modules = re.findall(r'(\w+):\[module_num:(\d+).*?\]', content)
    for m, num in modules:
        print(f'  {num}. {m}')
"`)
Viewed NF1_analysis_4channel.cppipe:90-160

Hier is een compleet technisch naslagwerk over alle CellProfiler-modules en instellingen die in dit project worden gebruikt, inclusief uitleg van de resulterende output-metingen. Je kunt dit direct kopiëren en opslaan als bijvoorbeeld `CELLPROFILER_PIPELINE_REFERENCE.md` in je GitHub repository.

---

# CellProfiler Pipeline & Module Reference: NF1 Schwann Cell Painting

Dit document beschrijft alle CellProfiler-modules die worden toegepast in de kwaliteitscontrole ([`1.cellprofiler_ic`](file:///home/livr/cellpainting_exercise/nf1_schwann_cell_painting_data/1.cellprofiler_ic)) en de morfologische analyse ([`2.cellprofiler_analysis`](file:///home/livr/cellpainting_exercise/nf1_schwann_cell_painting_data/2.cellprofiler_analysis)).

---

## Inhoudsopgave
1. [De 4 Basis Input-modules](#1-de-4-basis-input-modules)
2. [Quality Control (QC) Modules](#2-quality-control-qc-modules)
3. [Illumination Correction (IC) Modules](#3-illumination-correction-ic-modules)
4. [Celsegmentatie Modules](#4-celsegmentatie-modules)
5. [Morfologische Metingen (Feature Extraction)](#5-morfologische-metingen-feature-extraction)
6. [Export Modules](#6-export-modules)
7. [Interpretatie van de Feature Outputs (Kolomnamen)](#7-interpretatie-van-de-feature-outputs-kolomnamen)

---

## 1. De 4 Basis Input-modules

Deze modules staan standaard bovenaan elke CellProfiler pipeline en bepalen hoe beelden worden ingeladen.

### 1.1 `Images`
* **Doel:** Bepaalt welke bestanden vanuit de mapstructuur als afbeeldingen worden geladen.
* **Belangrijke Settings:**
  * `Filter images?: Images only` $\rightarrow$ Negeert niet-beeld bestanden (zoals `.txt` of verborgen OS-bestanden).

### 1.2 `Metadata`
* **Doel:** Haalt experimentele variabelen uit de bestandsnamen of mappenstructuur via Regular Expressions (RegEx).
* **Belangrijke Settings:**
  * `Regular expression to extract from file name:`  
    `^(?P<Plate>.*)_(?P<Well>[A-P][0-9]{2})_s(?P<Site>[0-9])_w(?P<ChannelNumber>[0-9])`
    * Haalt automatisch de plaatnaam, well-positie (bijv. B04), beeldveld/site (`s1`-`s9`) en kanaalnummer (`w1`-`w4`) uit de bestandsnaam.

### 1.3 `NamesAndTypes`
* **Doel:** Wijst de ruwe afbeeldingsbestanden toe aan herkenbare kanaalnamen (DAPI, GFP, RFP, CY5/Mito) en definieert het beeldtype.
* **Belangrijke Settings:**
  * `Select the image type: Grayscale image` $\rightarrow$ Fluorescentiebeelden worden kanaal voor kanaal in grijswaarden geanalyseerd.
  * `Set intensity range from: Image metadata` $\rightarrow$ Beveiliging tegen verkeerde normalisatie van 16-bit TIFF-intensiteiten (0–65535).
  * `Rule criteria:` Koppelt bestandsnamen met `DAPI` aan het DNA-kanaal, `GFP` aan het ER, `RFP` aan Actine, en `Mito/CY5` aan Mitochondriën.

### 1.4 `Groups`
* **Doel:** Groepeert beelden per plaat of per well als er plaat-specifieke bewerkingen nodig zijn.
* **Settings in deze pipeline:** `Do you want to group your images?: No` (Omdat de parallelisatie via de externe scriptwrapper `cp_parallel.py` per plaat wordt aangestuurd).

---

## 2. Quality Control (QC) Modules

### 2.1 `MeasureImageQuality`
* **Doel:** Berekent objectieve statistische kwaliteitsscores voor het volledige beeldveld (Field of View / FOV) vóór segmentatie.
* **Belangrijke Settings:**
  * `Calculate blur metrics?: Yes` $\rightarrow$ Berekent o.a. de *PowerLogLogSlope* (frequentiespectrum van scherpe randen).
  * `Calculate saturation metrics?: Yes` $\rightarrow$ Berekent het percentage verzadigde (overbelichte) pixels (*PercentMaximal*).
* **Output:**
  * `ImageQuality_PowerLogLogSlope`: Mate van onscherpte. Waarden dicht bij 0 duiden op lege beelden; sterk negatieve waarden duiden op ernstige onscherpte (out-of-focus).
  * `ImageQuality_PercentMaximal`: Percentage pixels op de maximale sensorwaarde. Hoge waarden wijzen op felle artefacten (stof, luchtbellen, fluorofoor-neerslag).

### 2.2 `FlagImage`
* **Doel:** Markeert en filtert beelden die buiten de statistisch bepaalde kwaliteitsgrenzen vallen.
* **Belangrijke Settings:**
  * `How should measurements be linked?: Flag if any fail` $\rightarrow$ Een beeld wordt afgekeurd zodra één enkel kanaal faalt.
  * `Skip image set if flagged?: Yes` $\rightarrow$ **Slaat het hele beeld over.** Dit voorkomt dat slechte beelden de belichtingscorrectie of celsegmentatie vervuilen.
  * `PowerLogLogSlope (Blur)`: Grenswaarden `[-2.50, -1.37]`.
  * `PercentMaximal (Saturation)`: Maximumgrens `1.40`.
* **Output:**
  * `Image_Quality_Control_QC_Flag`: `0` = goedgekeurd, `1` = afgekeurd/overgeslagen.

---

## 3. Illumination Correction (IC) Modules

Microscopen belichten het centrum feller dan de randen (vignettering). Deze modules herstellen een egaal belichtingsprofiel.

### 3.1 `CorrectIlluminationCalculate`
* **Doel:** Berekent een glad wiskundig achtergrondoppervlak (de illuminatiefunctie) per fluorescentiekanaal.
* **Belangrijke Settings:**
  * `Calculation method: Background` $\rightarrow$ Geschikt voor beelden met verspreide cellen waarin de achtergrond domineert.
  * `Block size: 50` $\rightarrow$ Verdeelt het beeld in vakjes van $50 \times 50$ pixels en zoekt per blokje de minimale intensiteit (lokale achtergrondwaarde). Moet groter zijn dan de typische celkern.
  * `Smoothing method: Fit Polynomial` $\rightarrow$ Fit een gladde 2D-polynoomcurve door de blokminima.
  * `Rescale the illumination function?: No` $\rightarrow$ Cruciaal bij de achtergrondmethode om absolute achtergrondwaarden correct te kunnen aftrekken.
* **Output:** Een kunstmatige afbeelding van de achtergrondverlichting (`IllumDAPI`, `IllumGFP`, etc.).

### 3.2 `CorrectIlluminationApply`
* **Doel:** Past de berekende achtergrondfunctie toe op het originele microscoopbeeld.
* **Belangrijke Settings:**
  * `How is illumination function applied?: Subtract` $\rightarrow$ Trekt de gemodelleerde achtergrondintensiteit af ($I_{\text{gecorrigeerd}} = I_{\text{origineel}} - I_{\text{illum}}$).
* **Output:** Gecorrigeerde beelden (`CorrDAPI`, `CorrGFP`, `CorrMito`, `CorrRFP`).

### 3.3 `SaveImages`
* **Doel:** Schrijft de gecorrigeerde beelden weg naar de schijf voor stap 2.
* **Belangrijke Settings:**
  * `Saved file format: tiff`
  * `Image bit depth: 16-bit integer` $\rightarrow$ Behoudt het volledige dynamische bereik van de microscoopsensor.
  * `Append suffix: _illumcorrect`

---

## 4. Celsegmentatie Modules

Deze modules herkennen en bakenen de drie celcompartimenten af: kernen, hele cellen en cytoplasma.

```
       +-------------------------------+
       |             Cells             |
       |    +---------------------+    |
       |    |       Nuclei        |    |
       |    |       (DAPI)        |    |
       |    +---------------------+    |
       |           Cytoplasm           |
       |    (Cells minus Nuclei)       |
       +-------------------------------+
```

### 4.1 `IdentifyPrimaryObjects` (Kernen / Nuclei)
* **Doel:** Detecteert individuele celkernen op basis van het DAPI-kanaal.
* **Belangrijke Settings:**
  * `Typical diameter of objects: 10 - 80 pixels` $\rightarrow$ Filtert debris (te klein) en grote artefacten (te groot) direct weg.
  * `Discard objects touching border: Yes` $\rightarrow$ Verwijdert cellen op de rand van de foto omdat hun vorm en volume onvolledig zijn.
  * `Thresholding method: Otsu (Three classes, middle class to foreground)` $\rightarrow$ Scheidt felle kernen, zwakke achtergrond, en intermediaire pixels betrouwbaar.
  * `Method to distinguish clumped objects: Shape / Intensity` $\rightarrow$ Splitst overlappende of rakende celkernen.
* **Output:** Het object **`Nuclei`**.

### 4.2 `IdentifySecondaryObjects` (Hele Cellen / Cells)
* **Doel:** Breidt vanaf elke geïdentificeerde kern uit naar buiten om de buitenste celmembraan te vinden.
* **Belangrijke Settings:**
  * `Input objects: Nuclei` (primaire zaden).
  * `Input image: RFP` (Actine toont het cytoskelet en de celranden van Schwann-cellen).
  * `Method: Propagation` $\rightarrow$ Laat de celrand vanaf de kern groeien totdat een intensiteitsdrempel of een buurcel wordt geraakt.
  * `Regularization factor: 0.05` $\rightarrow$ Bepaalt hoe strak de celomtrek de intensiteitsgradient volgt vs. een gladde ronde vorm aanhoudt.
* **Output:** Het object **`Cells`**.

### 4.3 `IdentifyTertiaryObjects` (Cytoplasma / Cytoplasm)
* **Doel:** Definieert het cytoplasma door het kerngebied af te trekken van de totale cel.
* **Belangrijke Settings:**
  * `Larger objects: Cells`
  * `Smaller objects: Nuclei`
  * `Shrink smaller object prior to subtraction?: Yes` $\rightarrow$ Voorkomt randoverlap tussen kern en cytoplasma.
* **Output:** Het object **`Cytoplasm`** ($\text{Cells} \setminus \text{Nuclei}$).

---

## 5. Morfologische Metingen (Feature Extraction)

In deze modules meet CellProfiler honderden numerieke kenmerken per cel. Deze vormen de biologische "fingerprint" van de cel.

### 5.1 `MeasureObjectSizeShape`
* **Wat het meet:** Oppervlakte, omtrek en geometrische vormkenmerken van `Nuclei`, `Cells` en `Cytoplasm`.
* **Belangrijke outputs:**
  * `Area`: Aantal pixels binnen het object (oppervlakte).
  * `Perimeter`: Omtreklengte.
  * `FormFactor`: $4 \pi \cdot \text{Area} / \text{Perimeter}^2$ (waarde $1.0$ is een perfecte cirkel; lagere waarden betekenen grillige/stervormige uitlopers).
  * `Solidity`: Verhouding van de oppervlakte tot de omhullende convexe veelhoek (*convex hull*). Laag bij ingedeukte of vertakte cellen.
  * `Eccentricity`: Mate van langwerpigheid (0 = cirkel, 1 = lijn/ellips; belangrijk voor spoelvormige Schwann-cellen).

### 5.2 `MeasureObjectIntensity` & `MeasureImageIntensity`
* **Wat het meet:** Fluorescentiesignalen van alle 4 de kanalen binnen elk celcompartiment.
* **Belangrijke outputs:**
  * `IntegratedIntensity`: Totale som van alle pixelwaarden binnen het compartiment.
  * `MeanIntensity`: Gemiddelde fluorescentiesterkte per pixel.
  * `StdIntensity`: Variatie in intensiteit binnen de cel.
  * `MinIntensity` / `MaxIntensity`: Uiterste intensiteitswaarden.

### 5.3 `MeasureObjectIntensityDistribution`
* **Wat het meet:** Hoe de kleuring ruimtelijk verdeeld is van het centrum van de cel naar de buitenrand (in concentrische schillen/ringen).
* **Belangrijke outputs:**
  * `FracAtDistance`: Fractie van de fluorescentie op een bepaalde relatieve afstand van het zwaartepunt.
  * `RadialCV`: Coëfficiënt van variatie van intensiteiten langs een ring op gelijke afstand van het centrum (meet inhomogeniteit in specifieke schillen).

### 5.4 `MeasureTexture`
* **Wat het meet:** Grijswaarden-textuurpatronen via Haralick co-occurrence matrices op verschillende ruimtelijke schalen (pixelschalen 3, 5, 10).
* **Belangrijke outputs:**
  * `AngularSecondMoment` (Uniformiteit/Energie): Hoog bij homogene, egale kleuring.
  * `Contrast`: Mate van lokale intensiteitsverschillen tussen buurpixels.
  * `Entropy`: Mate van wanorde of willekeur in het intensiteitspatroon.
  * `Correlation`: Lineaire afhankelijkheid tussen aangrenzende pixels.

### 5.5 `MeasureGranularity`
* **Wat het meet:** Aanwezigheid van korrelige structuren (granula) van verschillende groottes via wiskundige morfologie (opening filters).
* **Belangrijke outputs:**
  * `Granularity_1` t/m `Granularity_16`: Piekwaarden geven aan op welke schaal (hoeveelheid pixels) organellen zoals mitochondriën of ER-clusters geclusterd zijn.

### 5.6 `MeasureColocalization`
* **Wat het meet:** Ruimtelijke overlap en correlatie tussen twee verschillende kanalen binnen een cel (bijv. DAPI vs. GFP, of Mito vs. Actine).
* **Belangrijke outputs:**
  * `Correlation`: Pearson correlatiecoëfficiënt tussen twee kanalen binnen het object.
  * `Overlap`: Fractie van overlappende pixels boven de intensiteitsdrempel.
  * `MandersCoefficient`: Manders' colokalisatiecoëfficiënten ($M_1, M_2$).

### 5.7 `MeasureObjectNeighbors`
* **Wat het meet:** Ruimtelijke organisatie en celdichtheidseffecten (hoe dicht cellen op elkaar gepakt zitten).
* **Belangrijke outputs:**
  * `NumberOfNeighbors`: Aantal direct rakende buurcellen.
  * `PercentTouching`: Percentage van de celomtrek dat contact maakt met naburige cellen.
  * `DistanceToClosestNeighbor`: Afstand van zwaartepunt tot de dichtstbijzijnde buurcel.

---

## 6. Export Modules

### 6.1 `ExportToSpreadsheet` (Gebruikt in IC / QC)
* **Doel:** Exporteert per-image metingen naar een overzichtelijke CSV (`IC_QC_RunImage.csv` of `Image.csv`).
* **Gebruik:** Vormt de invoer voor R- en Python-kwaliteitsrapporten en platemap visualisaties.

### 6.2 `ExportToDatabase` (Gebruikt in Segmentatie/Analyse)
* **Doel:** Schrijft alle miljoenen individuele single-cell metingen weg naar een relationele **SQLite**-database (`.sqlite`).
* **Structuur van de database:**
  * Tabel `Per_Image`: Bevat metadata, beeldpaden en globale beeldscores.
  * Tabellen `Per_Nuclei`, `Per_Cells`, `Per_Cytoplasm`: Bevatten alle features per individuele cel, gekoppeld via `ImageNumber` en `ObjectNumber`.
* **Vervolgstap:** In stap [`3.processing_features`](file:///home/livr/cellpainting_exercise/nf1_schwann_cell_painting_data/3.processing_features) leest **CytoTable** deze SQLite-tabellen uit en voegt ze samen tot Parquet-bestanden.

---

## 7. Interpretatie van de Feature Outputs (Kolomnamen)

CellProfiler gebruikt een consistente naamgevingsconventie voor alle gemeten features:

$$\text{[Compartiment] \_ [FeatureCategorie] \_ [Meting] \_ [Kanaal/Parameter] \_ [Schaal]}$$

### Voorbeelden ontleed:

| Featurenaam | Compartiment | Categorie | Betekenis |
| :--- | :--- | :--- | :--- |
| `Nuclei_AreaShape_Solidity` | Nuclei | Area/Shape | Hoe convex en regelmatig de celkern is (waarde tussen 0 en 1). Laag bij fragmentatie of apoptose. |
| `Cells_Intensity_MeanIntensity_OrigGFP` | Cells | Intensiteit | Gemiddelde fluorescentie van ER/GFP over de gehele cel. |
| `Cytoplasm_Texture_Contrast_OrigMito_3_00_256` | Cytoplasm | Textuur | Mate van scherp contrast in mitochondriale kleuring op een afstand van 3 pixels. |
| `Nuclei_Granularity_1_OrigDAPI` | Nuclei | Granulariteit | Aanwezigheid van fijne puncta (korrels van 1 pixel breed) in de celkern (heterochromatine). |
| `Cells_Neighbors_NumberOfNeighbors_Adjacent` | Cells | Buren | Aantal cellen waarmee deze specifieke cel fysiek contact maakt. |
