# Tiny Tiny RSS Documentation Site

This repository contains the source for <https://tt-rss.github.io>.

## Contributing

Contributions to improve the documentation are welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

By contributing, you agree that your contributions will be licensed under CC BY-SA 4.0 (see [License](#license) below).

## Structure

```text
tt-rss.github.io/
├── _config.yml          # Jekyll configuration
├── index.md             # Homepage
├── docs/                # Documentation pages
├── images/              # Screenshots and images
├── LICENSE              # CC BY-SA 4.0 license
├── CONTRIBUTING.md      # Contribution guidelines
└── README.md            # This file
```

## Building Locally

This is a Jekyll-based GitHub Pages site. To build locally:

```bash
# Install dependencies
bundle install

# Run local server
bundle exec jekyll serve

# View at http://localhost:4000
```

## Useful Links

- **Documentation Repository:** [https://github.com/tt-rss/tt-rss.github.io](https://github.com/tt-rss/tt-rss.github.io)
- **Documentation Site:** [https://tt-rss.github.io](https://tt-rss.github.io)
- **Main Repository:** [https://github.com/tt-rss/tt-rss](https://github.com/tt-rss/tt-rss)
- **Discussions:** [https://github.com/tt-rss/tt-rss/discussions](https://github.com/tt-rss/tt-rss/discussions)
- **Issues:** [https://github.com/tt-rss/tt-rss/issues](https://github.com/tt-rss/tt-rss/issues)

## License

### Documentation Content

The documentation and site content in this repository is licensed under the [Creative Commons Attribution-ShareAlike 4.0 International License (CC BY-SA 4.0)](LICENSE).

**You are free to:**
- Share and redistribute the documentation
- Adapt and build upon the documentation

**Under these terms:**
- **Attribution** — Give appropriate credit and link to the license
- **ShareAlike** — Distribute adaptations under the same CC BY-SA 4.0 license

### Tiny Tiny RSS

The tt-rss software itself is licensed under the [GNU General Public License v3.0 (GPLv3)](https://github.com/tt-rss/tt-rss/blob/master/COPYING).

Any code snippets or configuration examples derived from tt-rss retain their original GPLv3 license.
