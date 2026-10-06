<div align="center">

# Furqan al-Hadith
### فرقان الحديث

**Verify. Compare. Follow the evidence.**

A hadith reference app that presents primary-source comparisons across Sunni, Shia, and Ahmadiyya traditions, so readers can verify claims against the Quran and authentic hadith, and learn to distinguish truth (Haqq - حق) from falsehood (Batil - باطل).

![License](https://img.shields.io/badge/license-MIT-green)
![Status](https://img.shields.io/badge/status-in%20development-orange)
![PRs](https://img.shields.io/badge/PRs-welcome-brightgreen)

[Features](#-features) · [Data](#-data-sources) · [Getting Started](#-getting-started) · [Roadmap](#-roadmap) · [Contributing](#-contributing)

</div>

---

## 📖 About

> وَقُلْ جَاءَ الْحَقُّ وَزَهَقَ الْبَاطِلُ
> *"Truth has come, and falsehood has vanished."* (Quran 17:81)

**Furqan** means the criterion that separates truth from falsehood. This project lets readers weigh claims against the Quran and hadith by going to the sources themselves, with every reference cited and traceable.

### Our approach

- **Primary sources first.** Every claim links to its original text, book, volume, and number.
- **Each tradition in its own words.** Shia and Ahmadiyya material comes directly from their own published libraries, quoted accurately and in context, never paraphrased to fit a conclusion.
- **Transparent perspective.** This project is written from an **Ahlus Sunnah** perspective, and says so openly. Readers are encouraged to check every source and judge for themselves (*tabayyun*, Quran 49:6).
- **Authentication matters.** Hadith are shown with their grading and chain of narration (isnad) wherever available.

---

## ✨ Features

**Current focus (Sunni Islam)**

- [x] Browse the Forty Hadith Qudsi
- [ ] Major hadith collections with Arabic text and English translation
- [ ] Isnad (chain of narration) and grading display
- [ ] Topic-based search (belief, worship, companions, etc.)
- [ ] Haqq / Batil study guides with cited evidence
- [ ] Bookmarks, personal notes, and offline mode

**Planned (future)**

- [ ] Side-by-side comparison with Shia and Ahmadiyya sources, pulled from their own published libraries (API or scraping, with links back to the original)
- [ ] Multi-language support

---

## 🗂 Data Sources

Arabic text, gradings, and translations come from the sources below. Each text is traced back to its original publisher or translator, and licenses are recorded in [`data/SOURCES.md`](data/SOURCES.md).

### Sources by collection

| Collection | Arabic | English | Tamil | Gradings |
|---|---|---|---|---|
| **Sahih al-Bukhari** | [fawazahmed0](https://github.com/fawazahmed0/hadith-api) | [Sunnah.com](https://sunnah.com/bukhari) once approved; until then fawazahmed0's English | Check fawazahmed0 editions; else [HadeethEnc](https://hadeethenc.com) | Collection-level note (sahih by consensus) |
| **Sahih Muslim** | [fawazahmed0](https://github.com/fawazahmed0/hadith-api) | [Sunnah.com](https://sunnah.com/muslim); stand-in: fawazahmed0 | Same as above | Collection-level note |
| **Sunan Abi Dawud** | [fawazahmed0](https://github.com/fawazahmed0/hadith-api) | [Sunnah.com](https://sunnah.com/abudawud); stand-in: fawazahmed0 | Same as above | fawazahmed0 (multi-grader) |
| **Jami' at-Tirmidhi** | [fawazahmed0](https://github.com/fawazahmed0/hadith-api) | [Sunnah.com](https://sunnah.com/tirmidhi); stand-in: fawazahmed0 | Same as above | fawazahmed0, plus the author's own grading |
| **Sunan an-Nasa'i** | [fawazahmed0](https://github.com/fawazahmed0/hadith-api) | [Sunnah.com](https://sunnah.com/nasai); stand-in: fawazahmed0 | Same as above | fawazahmed0 (multi-grader) |
| **Sunan Ibn Majah** | [fawazahmed0](https://github.com/fawazahmed0/hadith-api) | [Sunnah.com](https://sunnah.com/ibnmajah); stand-in: fawazahmed0 | Same as above | fawazahmed0 (multi-grader) |
| **Muwatta Malik** | [fawazahmed0](https://github.com/fawazahmed0/hadith-api) | [Sunnah.com](https://sunnah.com/malik); stand-in: fawazahmed0 | Same as above | fawazahmed0 |
| **Musnad Ahmad** | [mhashim6 Open-Hadith-Data](https://github.com/mhashim6/Open-Hadith-Data) | [Sunnah.com](https://sunnah.com/ahmad) request; [hadith-json](https://github.com/AhmedBaset/hadith-json) for development only | None found | None ("not yet graded") |
| **Sunan ad-Darimi** | [mhashim6 Open-Hadith-Data](https://github.com/mhashim6/Open-Hadith-Data) | [Sunnah.com](https://sunnah.com/darimi) request; hadith-json for development only | None found | None ("not yet graded") |
| **Forty Hadith an-Nawawi** | [fawazahmed0](https://github.com/fawazahmed0/hadith-api) | fawazahmed0 or [HadeethEnc](https://hadeethenc.com) | HadeethEnc | fawazahmed0 (verify in the data) |
| **Forty Hadith Qudsi** | [fawazahmed0](https://github.com/fawazahmed0/hadith-api) (also [hapiam/hadith-json](https://github.com/hapiam/hadith-json/blob/main/db/by_book/forties/qudsi40.json)) | fawazahmed0 or HadeethEnc | HadeethEnc | fawazahmed0 (verify in the data) |
| **Riyad as-Salihin** | [hadith-json](https://github.com/AhmedBaset/hadith-json) (development only) | [Sunnah.com](https://sunnah.com/riyadussalihin) request | HadeethEnc (selected hadith only) | None found; ask Sunnah.com |

### Source types

| Source | Type | Role |
|---|---|---|
| [fawazahmed0/hadith-api](https://github.com/fawazahmed0/hadith-api) | Compiler | Arabic text and gradings |
| [mhashim6/Open-Hadith-Data](https://github.com/mhashim6/Open-Hadith-Data) | Compiler | Arabic for Musnad Ahmad and ad-Darimi |
| [Sunnah.com](https://sunnah.com) / [API](https://github.com/sunnah-com/api) | Original publisher | English, with permission and attribution |
| [HadeethEnc](https://hadeethenc.com) | Original publisher | Graded selection in many languages |
| [QuranLab on Hugging Face](https://huggingface.co/datasets/quranlab/hadith) | Aggregator | Cross-check copy of the above |
| [AhmedBaset/hadith-json](https://github.com/AhmedBaset/hadith-json) | Scraped from Sunnah.com | Development and prototyping only |

### Licensing notes

- **Arabic text** is classical and in the public domain. The repo licenses of the compilers cover their code and formatting.
- **Translations** belong to their translators and publishers. Each hadith shows the translator's name, and removal requests are honored. See the [disclaimer](#️-disclaimer).
- **Bukhari and Muslim** carry a collection-level note ("accepted as authentic by scholarly consensus"), not a grade on every hadith.
- **Gradings** always show the grader's name and source.
- Check each repo's `LICENSE` file before use and record it in `data/SOURCES.md`.

For comparison, Shia and Ahmadiyya material is **not stored in this repo as our own database**. It is fetched from those communities' own published sources, cached, and linked back to the original.

### Data format (example)

```json
{
  "id": 1,
  "collection": "Forty Hadith Qudsi",
  "arabic": "...",
  "english": "...",
  "narrator": "...",
  "reference": "Book, Number",
  "grading": "Sahih",
  "source_url": "..."
}
```

---

## 🛠 Tech Stack

> Fill in once decided.

| Layer | Choice |
|-------|--------|
| Frontend | Next.js |
| Backend | FastApi |
| Data | JSON files → database |
| Hosting | Railway & Vercel |

---

## 🚀 Getting Started

```bash
# Clone the repository
git clone https://github.com/<your-username>/furqan-al-hadith.git
cd furqan-al-hadith

# Install dependencies
npm install

# Run locally
npm run dev
```

---

## 📁 Project Structure

```
furqan-al-hadith/
├── data/
│   ├── sunni/
│   │   ├── hadith/      # Hadith collections (JSON)
│   │   ├── tafsir/      # Quran commentary
│   │   └── creed/       # Creed and history sources
│   ├── quran/           # Quran text and translations
│   └── topics/          # Topic tags that map Sunni entries to external sources
├── src/
│   ├── connectors/      # (future) API / scraper modules for external sources
│   └── ...              # Application code
├── docs/                # Methodology and guidelines
├── CONTRIBUTING.md
├── LICENSE
└── README.md
```

---

## 🗺 Roadmap

The project is built in stages. **Sunni Islam is completed first**, so the foundation (data model, grading system, citations) is solid before any other tradition is added.

### Part A: Sunni Islam (current focus)

- [ ] **Phase 1: Foundation**
  - [x] Forty Hadith Qudsi
  - [ ] Forty Hadith of an-Nawawi
  - [ ] Sahih al-Bukhari
  - [ ] Sahih Muslim
  - [ ] Data model: text, translation, numbering system, narrator, source URL
  - [ ] Grading system: grade, grader name, and source of the verdict stored separately from the text
- [ ] **Phase 2: Complete the Six Books**
  - [ ] Sunan Abi Dawud
  - [ ] Jami' at-Tirmidhi
  - [ ] Sunan an-Nasa'i
  - [ ] Sunan Ibn Majah
  - Note: these contain sahih, hasan, and da'if narrations, so grading must be live before release.
- [ ] **Phase 3: Wider collections**
  - [ ] Muwatta Malik
  - [ ] Musnad Ahmad
  - [ ] Sunan ad-Darimi
  - [ ] Riyad as-Salihin
- [ ] **Phase 4: Verification tools**
  - [ ] Isnad (chain of narration) display
  - [ ] Narrator biographies (rijal)
  - [ ] Multiple scholars' gradings side by side
  - [ ] Search by topic, narrator, keyword, and hadith number
- [ ] **Phase 5: Quran and commentary**
  - [ ] Quran text (Uthmani) with translation
  - [ ] Tafsir (at-Tabari, Ibn Kathir, al-Qurtubi, as-Sa'di)
  - [ ] Links between verses and related hadith
- [ ] **Phase 6: Creed, history, and study guides**
  - [ ] Core creed texts (Tahawiyyah, Wasitiyyah)
  - [ ] Sirah and history sources
  - [ ] Haqq / Batil study guides with cited evidence
- [ ] **Phase 7: App polish**
  - [ ] Bookmarks and notes
  - [ ] Offline mode
  - [ ] Mobile apps (iOS / Android)

### Part B: Comparison layer (after Sunni Islam is complete)

> We do **not** build or host our own Shia or Ahmadiyya database. Those communities already maintain their own digital libraries, so the app pulls from **their** published sources and shows them next to the Sunni evidence for the same topic.

- [ ] **Source connectors**
  - [ ] Identify each community's own official or widely used digital libraries
  - [ ] Use official APIs where they exist; use scraping only where permitted
  - [ ] Cache results and always link back to the original page
- [ ] **Topic mapping**
  - [ ] Tag Sunni entries by topic, so a topic can be matched against external sources
  - [ ] Manual review of every match before it appears in the app
- [ ] **Comparison view**
  - [ ] Side-by-side display: Sunni sources vs. the other tradition's own sources
  - [ ] Every external quote shows book, volume, number or page, edition, and a link to the original
  - [ ] Context shown with each quote, never an isolated line
- [ ] **Quality and fairness checks**
  - [ ] Broken-link and changed-content monitoring
  - [ ] Report button for incorrect or out-of-context matches

---

## 📏 Source & Citation Standards

Every comparison entry must include:

1. The original text (Arabic) and a faithful translation
2. Book, volume, chapter, and hadith number
3. The grading and who graded it
4. Enough surrounding context that the quote is not misleading
5. A link to a scan or trusted digital edition where possible

Entries that cannot be sourced will not be accepted.

---

## 🤝 Contributing

Contributions are welcome, especially from people with knowledge of Arabic and hadith sciences. Please read [CONTRIBUTING.md](CONTRIBUTING.md) first.

- Report data errors or mistranslations through Issues
- Submit new sourced entries through Pull Requests
- Help review translations and references

---

## ⚠️ Disclaimer

This app is an educational reference tool and not a replacement for qualified scholars. Rulings and understanding should be taken from trusted, knowledgeable people. Any errors are unintentional, so please report them so they can be corrected.

---

## 📄 License

Released under the [MIT License](LICENSE). Data sources keep their original licenses.

---

<div align="center">

*وَمَا تَوْفِيقِي إِلَّا بِاللَّهِ*

Made with the intention of seeking truth.

</div>
