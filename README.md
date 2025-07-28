# Organization-Wide GitHub Configuration

This repository provides **shared configuration** for all repositories in the organization.

## Purpose

The `.github` repository is used to:

- Define **reusable issue templates** that members can use across any organization repository.
- Centralize configuration for **design reviews**, bug reports, and other recurring issue types.
- Promote **consistency and best practices** across engineering teams.
- Avoid duplication of templates and policies in each individual repo.

## Available Templates

### 🧩 Design Review
Use this when proposing:
- New components or services
- Structural changes (e.g. databases, file formats)
- New interfaces (REST, GRPC, event protocols)
- High-risk or complex features

🔗 Select it from the issue creation screen under **"Design Review"**  
📄 Source: [`ISSUE_TEMPLATE/design_review.md`](./ISSUE_TEMPLATE/design_review.md)

## Notes

- This repo must be **public** for templates to be used across public repos.
- Private repos can use the templates if this repo is also private and within the same organization.
- This does **not automatically enforce** templates in all repos — they must opt in or use them manually.

## Setup in Other Repos

To use the template:
1. Create a new issue in your repo.
2. If this `.github` repo is public and properly configured, GitHub will offer the **Design Review** template in the issue chooser.
3. Alternatively, copy the template contents into your repo or issue.

## License

Internal use only.
