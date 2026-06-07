# AI301 Open Source Contribution Log
**Contributor:** Kourtney Miranda (@mirakour)
**Program:** CodePath AI301 — AI Open Source Capstone, Summer 2026
**Status:** Phase I Complete

---

## Phase I: Issue Selection

**Issue:** [apache/superset #36189 — Percentage number formatting broken for very small numbers](https://github.com/apache/superset/issues/36189)

**Repository:** [https://github.com/apache/superset](https://github.com/apache/superset)

**Problem Summary:** In Apache Superset's table visualization, applying D3 percentage formatting (e.g., `.8%`) to very small non-zero numbers (e.g., `-0.00001229`) fails silently — the raw number is returned instead of the formatted percentage. When any column contains very small values, formatting breaks for all rows in that column, including larger values that would otherwise format correctly. This affects Superset v5.0.0 and is confirmed reproducible by multiple users.

**Why I Chose This Issue:** This is a well-scoped TypeScript/React frontend bug in a major open-source data visualization project. It matches my experience with TypeScript, React, and frontend development. It's labeled "good first issue," meaning the maintainers have confirmed it's accessible to new contributors and are actively welcoming PRs. Fixing it will strengthen my skills in open source contribution workflows and frontend data formatting logic.

---

## Phase II: Reproduce & Plan

**Understanding the Issue:** <!-- Explain the issue in your own words -->

**Reproduction Steps:** <!-- How did you reproduce the bug locally? -->

**Solution Approach:** <!-- What is your plan to fix it? Which files/modules are involved? -->

---

## Phase III: Build

**Testing Strategy:** <!-- How are you testing your changes? -->

**Implementation Notes:** <!-- Key decisions made while building -->

---

## Phase IV: Submit & Iterate

**Pull Request:** <!-- Link to your PR -->

**PR Summary:** <!-- What does your PR do? -->

**Maintainer Feedback Log:**

| Date | Feedback | Response |
|------|----------|----------|
|      |          |          |
