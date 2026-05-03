# Ersilia Book

Documentation for the [Ersilia Open Source Initiative](https://ersilia.io) — equipping laboratories in the Global South with AI tools for infectious and neglected disease research.

## Read the book

The book is published at **https://ersilia.gitbook.io/ersilia-book**. That is the recommended place to browse the docs; this repository is the source the published site is built from.

## What's inside

The book is organised into six top-level sections (see [`book/SUMMARY.md`](book/SUMMARY.md) for the full table of contents):

- **🤗 Welcome to Ersilia** — the organisation, our ten principles, and the Ersilia ecosystem.
- **🚀 Ersilia Model Hub** — getting started, online and local inference, the model contribution workflow, and developer docs (CLI, Python API, CI/CD, model and CLI testing, results caching, version control).
- **💊 Chemistry tools** — automated activity prediction (LazyQSAR, ZairaChem), model distillation with Olinda, sampling the chemical space, model encryption, and AMR chemical collections.
- **🙌 Contributors** — Outreachy internship cohorts and contribution opportunities.
- **🎨 Styles** — brand guidelines, slide and document templates, and scientific figures with Stylia.
- **🌍 About Us** — governance, code of conduct, diversity and inclusion, strategic plans, and the Ersilia privacy notice.

## Repository layout

```
.
├── .gitbook.yaml      # GitBook config (root: ./book/)
├── LICENSE            # GNU GPLv3
├── README.md          # this file
└── book/              # documentation source
    ├── README.md      # the published book's landing page
    ├── SUMMARY.md     # table of contents
    └── <section>/     # markdown pages, grouped by section
```

The book is built and deployed automatically by GitBook from this repository — there is no local build step to run.

## Contributing

We welcome corrections, clarifications, and new content from the community.

- For typos, broken links, or small content suggestions, please [open an issue](https://github.com/ersilia-os/ersilia-book/issues).
- For larger edits:
  1. Fork the repository.
  2. Edit the relevant page under `book/<section>/<page>.md` (or add a new one).
  3. If you add a new page, register it in [`book/SUMMARY.md`](book/SUMMARY.md) so it appears in the book's navigation.
  4. Open a pull request describing your change.

All participants are expected to follow our [Code of Conduct](book/about-us/code-of-conduct.md) (Contributor Covenant), which lists the contacts for private reports or questions.

## About Ersilia

The Ersilia Open Source Initiative is a tech non-profit foundation based in Barcelona. We bridge the gap between data scientists and experimental researchers, with a focus on infectious and neglected diseases in low- and middle-income countries. Browse our other open source tools at [github.com/ersilia-os](https://github.com/ersilia-os) and learn more at [ersilia.io](https://ersilia.io).

## License

The contents of this book are released under the GNU General Public License v3 — see [`LICENSE`](LICENSE).

## Contact

- Email: [hello@ersilia.io](mailto:hello@ersilia.io)
- Newsletter: [subscribe](http://eepurl.com/hkX1sH)
- Social: [LinkedIn](https://linkedin.com/company/ersiliaio), [Bluesky](https://bsky.app/profile/ersilia.io), [Mastodon](https://fosstodon.org/@ersiliaio), [Medium](https://medium.com/ersiliaio)
