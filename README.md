# PenguinMails Documentation

[![Documentation Status](https://img.shields.io/badge/Documentation-Active-brightgreen.svg)](https://penguinmails.github.io)
[![Build Status](https://github.com/penguinmails/penguinmails.github.io/actions/workflows/ci.yml/badge.svg)](https://github.com/penguinmails/penguinmails.github.io/actions)
[![Contributors](https://img.shields.io/github/contributors/penguinmails/penguinmails.github.io.svg)](CONTRIBUTORS.md)
[![Issues](https://img.shields.io/github/issues/penguinmails/penguinmails.github.io.svg)](https://github.com/penguinmails/penguinmails.github.io/issues)
[![License](https://img.shields.io/github/license/penguinmails/penguinmails.github.io.svg)](LICENSE)

> **Enterprise-grade email platform documentation**  
> Complete, actionable guides for devs, ops, and business users.

---

## 🚀 Quick Start

**Clone & Launch Docs Locally**

git clone https://github.com/penguinmails/penguinmails.github.io.git
cd penguinmails.github.io

Using Docker (recommended):
docker compose up

OR using Ruby (needs Jekyll installed):
bundle install
bundle exec jekyll serve --livereload  

Docs will be live at `http://localhost:4000`

- **See:** [Getting Started](docs/implement/getting-started.md)
- **Live Demo:** [penguinmails.github.io](https://penguinmails.github.io)

---

## 🤝 Contributing

Want to improve the docs or platform resources?  
- Read our [CONTRIBUTING.md](CONTRIBUTING.md) for PR steps, content standards, and style guides.
- Fork this repo, branch off, commit, and open a pull request.
- All code and docs are reviewed before merging via CI and linting.

**Contributor Credits:** See [CONTRIBUTORS.md](CONTRIBUTORS.md)  
Questions? Open an [issue](https://github.com/penguinmails/penguinmails.github.io/issues) or start a Discussion!

---

## 🛠️ Project Structure

`docs/` — All main documentation (edit/add here!)  
`deploy/`, `.github/` — CI/CD and workflow configs  
`CONTRIBUTING.md`, `CONTRIBUTORS.md`, `LICENSE` — Repo meta and contributor help

Common Doc Tasks:
- Add a new doc? → `docs/`
- Update site structure? → Edit nav in Jekyll config
- Contributing guide tweaks? → `CONTRIBUTING.md`

---

## 📋 Freelancer & Contributor Fastlane

**Freelancers & OSS contributors:**  
- Start at [Freelancer Onboarding Hub](/docs/freelancer-support/)
- Review [Task Clarity Essentials](/docs/freelancer-support/README.md#task-completion-standards) for DoD, story points, process
- See open tasks: [Issues](https://github.com/penguinmails/penguinmails.github.io/issues)

---

## 📚 Key Sections

- [What is PenguinMails](docs/what-is-penguinmails.md)
- [Features & Capabilities](docs/features-capabilities.md)
- [Implementation Guides](docs/implement/getting-started.md)
- [Operations](docs/operate/)
- [Design System](docs/design/)
- [Security](docs/security/)
- [Analytics](docs/analytics/)
- [Development Conventions](docs/development/)

See full structure below.

---

## 📂 Documentation Structure

docs/
├── what-is-penguinmails.md
├── features-capabilities.md
├── goals-competitive-edge.md
├── plan/
│ ├── high-level-architecture.md
│ ├── key-performance-indicators.md
│ └── roadmap-development-priorities.md
├── implement/
│ ├── getting-started.md
│ ├── backup-recovery.md
│ ├── database-operations.md
│ ├── deployment.md
│ ├── performance-monitoring.md
│ └── connection-pooling.md
├── operate/
│ ├── compliance-standards.md
│ ├── team-workflow.md
│ └── resources-support.md
├── design/
│ ├── design-system.md
│ ├── ui-library.md
│ ├── component-library.md
│ └── user-personas.md
├── security/
│ ├── overview.md
│ ├── incident-response.md
│ └── procedures.md
├── analytics/
│ ├── financial.md
│ ├── user-behavior.md
│ ├── growth.md
│ └── product-performance.md
├── development/
│ ├── style-guide.md
│ ├── faq-gotchas.md
│ └── best-practices.md
├── tasks/
│ └── project-management.md
└── finance-business-model.md


---

## ❓ FAQ

- **Build fails on new machine?**  
  Confirm Ruby & Bundler installed, then run `bundle install` again. Try `docker compose up` for an isolated build.
- **Docker not working?**  
  See [Docker Troubleshooting](docs/implement/docker-troubleshooting.md)
- **Can't find a document?**  
  Use site search or view sidebar navigation for section overviews.

---

## 📈 Project Statistics

| Metric                | Value   |
|-----------------------|---------|
| Documentation Pages   | 93+     |
| Active Contributors   | 5+      |
| Last Updated          | Dec 2024|
| Open Issues           | [View](https://github.com/penguinmails/penguinmails.github.io/issues) |

---

## 🏆 Acknowledgments

Built with:
- [Just the Docs](https://pmarsceill.github.io/just-the-docs/)
- [Jekyll](https://jekyllrb.com/)
- [GitHub Pages](https://pages.github.com/)
- [Docker](https://www.docker.com/)

---

## 📄 License

Licensed under the [Apache License 2.0](LICENSE).  
PenguinMails is a trademark of PenguinMails, Inc.

---

**⭐️ Star this repo if it helps you!**  
*See latest guides at [penguinmails.github.io](https://penguinmails.github.io).*


