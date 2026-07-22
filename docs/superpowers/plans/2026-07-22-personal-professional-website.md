# Personal Professional Website Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build and publish Xu Jianshuo's content-rich bilingual professional website so `https://xu-jianshuo.github.io/` loads the Chinese home page and `https://xu-jianshuo.github.io/#expertise` opens the expertise section.

**Architecture:** Use semantic static HTML pages in Chinese at the repository root and corresponding English pages under `/en/`. Share framework-free CSS and progressive-enhancement JavaScript across all pages, then deploy the repository root to GitHub Pages with an Actions workflow triggered by pushes to `main`.

**Tech Stack:** HTML5, CSS3, vanilla JavaScript, Node.js built-in test runner, GitHub Pages, GitHub Actions

---

## File map

Create or modify only the paths below. Preserve the existing `gefei/` directory unchanged.

```text
index.html                         Chinese home; owns #expertise
experience/index.html              Chinese career timeline
projects/index.html                Chinese project index and filters
projects/accident-reform/index.html
projects/non-auto-quality/index.html
projects/renewal-operations/index.html
projects/platform-integration/index.html
projects/cost-management/index.html
insights/index.html                Chinese article index
insights/insurance-management/index.html
about/index.html                   Chinese biography and contact
en/index.html                      English home; owns #expertise
en/experience/index.html
en/projects/index.html
en/projects/accident-reform/index.html
en/projects/non-auto-quality/index.html
en/projects/renewal-operations/index.html
en/projects/platform-integration/index.html
en/projects/cost-management/index.html
en/insights/index.html
en/insights/insurance-management/index.html
en/about/index.html
assets/css/tokens.css              Design tokens
assets/css/base.css                Reset, typography, layout, accessibility
assets/css/components.css          Header, cards, timeline, tags, footer
assets/css/pages.css               Home, listing and article page composition
assets/js/navigation.js            Mobile navigation and active state
assets/js/filters.js               Progressive project filtering
assets/js/motion.js                Reduced-motion-aware reveals
assets/images/portrait-mark.svg        Abstract decorative portrait mark
404.html                           Recovery navigation
robots.txt                         Crawler policy
sitemap.xml                        Public bilingual URLs
.nojekyll                          Prevent Jekyll processing
.github/workflows/pages.yml        Test and Pages deployment
tests/site.test.mjs                Structural, privacy and bilingual checks
```

## Task 1: Add the automated site contract

**Files:**
- Create: `tests/site.test.mjs`
- Reference: `docs/superpowers/specs/2026-07-22-personal-professional-website-design.md`

- [ ] **Step 1: Write the failing structural and privacy tests**

Create `tests/site.test.mjs` with Node built-ins only. The test must enumerate every public HTML file in the file map and assert:

```js
import test from "node:test";
import assert from "node:assert/strict";
import { readFile } from "node:fs/promises";
import { existsSync } from "node:fs";

const pairs = [
  ["index.html", "en/index.html"],
  ["experience/index.html", "en/experience/index.html"],
  ["projects/index.html", "en/projects/index.html"],
  ["projects/accident-reform/index.html", "en/projects/accident-reform/index.html"],
  ["projects/non-auto-quality/index.html", "en/projects/non-auto-quality/index.html"],
  ["projects/renewal-operations/index.html", "en/projects/renewal-operations/index.html"],
  ["projects/platform-integration/index.html", "en/projects/platform-integration/index.html"],
  ["projects/cost-management/index.html", "en/projects/cost-management/index.html"],
  ["insights/index.html", "en/insights/index.html"],
  ["insights/insurance-management/index.html", "en/insights/insurance-management/index.html"],
  ["about/index.html", "en/about/index.html"],
];

const pages = pairs.flat();

test("all planned pages exist", () => {
  for (const page of [...pages, "404.html"]) {
    assert.equal(existsSync(page), true, `${page} is missing`);
  }
});

test("pages are semantic, titled and linked to shared assets", async () => {
  for (const page of pages) {
    const html = await readFile(page, "utf8");
    assert.match(html, /<!doctype html>/i, `${page}: doctype`);
    assert.match(html, /<title>[^<]+<\/title>/i, `${page}: title`);
    assert.match(html, /<meta name="description" content="[^"]+"/i, `${page}: description`);
    assert.match(html, /<main[\s>]/i, `${page}: main landmark`);
    assert.match(html, /assets\/css|\.\.\/assets\/css/, `${page}: shared CSS`);
  }
});

test("language pairs point to each other", async () => {
  for (const [zhPath, enPath] of pairs) {
    const [zh, en] = await Promise.all([
      readFile(zhPath, "utf8"),
      readFile(enPath, "utf8"),
    ]);
    assert.match(zh, /hreflang="en"/i, `${zhPath}: English hreflang`);
    assert.match(en, /hreflang="zh-CN"/i, `${enPath}: Chinese hreflang`);
    assert.match(zh, /lang="zh-CN"/i, `${zhPath}: Chinese lang`);
    assert.match(en, /lang="en"/i, `${enPath}: English lang`);
  }
});

test("home pages expose the expertise anchor", async () => {
  for (const page of ["index.html", "en/index.html"]) {
    const html = await readFile(page, "utf8");
    assert.match(html, /id="expertise"/i, `${page}: #expertise`);
  }
});

test("public files do not disclose restricted personal data", async () => {
  for (const page of [...pages, "404.html"]) {
    const html = await readFile(page, "utf8");
    assert.doesNotMatch(html, /18926485677/, `${page}: full phone leaked`);
    assert.doesNotMatch(html, /1989\s*年?\s*10\s*月?/, `${page}: birth date leaked`);
  }
});

test("the existing gefei area is not referenced or repurposed", async () => {
  for (const page of pages) {
    const html = await readFile(page, "utf8");
    assert.doesNotMatch(html, /href=["'][^"']*gefei/i, `${page}: gefei link introduced`);
  }
});
```

- [ ] **Step 2: Run the tests and confirm the expected failure**

Run:

```powershell
node --test tests/site.test.mjs
```

Expected: FAIL on `all planned pages exist` because the site files have not been created.

- [ ] **Step 3: Commit the failing contract**

```powershell
git add tests/site.test.mjs
git commit -m "test: define personal site contract"
```

## Task 2: Build the shared visual and interaction foundation

**Files:**
- Create: `assets/css/tokens.css`
- Create: `assets/css/base.css`
- Create: `assets/css/components.css`
- Create: `assets/css/pages.css`
- Create: `assets/js/navigation.js`
- Create: `assets/js/filters.js`
- Create: `assets/js/motion.js`
- Create: `assets/images/portrait-mark.svg`

- [ ] **Step 1: Define tokens and base behavior**

Implement these stable public tokens in `assets/css/tokens.css`:

```css
:root {
  --navy-950: #0b1f33;
  --navy-800: #173a5e;
  --slate-600: #5a6875;
  --slate-300: #cbd3da;
  --paper: #f7f4ee;
  --white: #ffffff;
  --gold: #a87932;
  --max-width: 1180px;
  --reading-width: 720px;
  --radius: 18px;
  --shadow: 0 18px 50px rgb(11 31 51 / 10%);
  --space-1: 0.5rem;
  --space-2: 1rem;
  --space-3: 1.5rem;
  --space-4: 2rem;
  --space-5: 3rem;
  --space-6: 5rem;
}
```

In `base.css`, implement border-box sizing, a warm-white body, system Chinese/English font stacks, visible `:focus-visible`, a skip link, `.container`, `.reading-width`, responsive typography, images with intrinsic sizing, and a reduced-motion media query. Do not import external fonts.

- [ ] **Step 2: Implement reusable components and page layouts**

In `components.css`, define `.site-header`, `.site-nav`, `.language-switch`, `.credential-list`, `.expertise-grid`, `.card-grid`, `.project-card`, `.article-card`, `.timeline`, `.metric`, `.tag`, `.site-footer`, and `.button-link`.

In `pages.css`, define `.hero`, `.hero__content`, `.hero__aside`, `.section`, `.section-heading`, `.project-filter`, `.article-layout`, and `.contact-panel`. Use CSS Grid for wide layouts and collapse to one column below `760px`.

- [ ] **Step 3: Add progressive-enhancement JavaScript**

`navigation.js` must toggle the mobile menu with `aria-expanded`, close it on Escape, and leave desktop links as normal anchors. `filters.js` must read each button's `data-filter` and each card's `data-category`, update the hidden state, and show a `.filter-empty` message when no card matches. `motion.js` must skip observers when reduced motion is requested and otherwise add `.is-visible` to `[data-reveal]` elements.

Use this defensive initialization pattern in every script:

```js
document.addEventListener("DOMContentLoaded", () => {
  const root = document.documentElement;
  root.classList.add("js");
  // Query optional controls and return without error when they are absent.
});
```

- [ ] **Step 4: Add a neutral portrait illustration**

Create an accessible geometric SVG using navy, paper and gold shapes. The file must contain no embedded personal photo and no text claiming to be a portrait; HTML uses empty alt text while it remains decorative.

- [ ] **Step 5: Run the tests**

Run `node --test tests/site.test.mjs`.

Expected: still FAIL only because public HTML pages do not exist.

- [ ] **Step 6: Commit shared assets**

```powershell
git add assets
git commit -m "feat: add site design system"
```

## Task 3: Build the Chinese home and profile pages

**Files:**
- Create: `index.html`
- Create: `experience/index.html`
- Create: `about/index.html`

- [ ] **Step 1: Create the Chinese home page**

Use `lang="zh-CN"`, title `徐建硕｜财产保险经营、精算与风险管理`, and a description that mentions ten years of property-insurance experience. Include canonical and `hreflang` links for `/` and `/en/`, Open Graph metadata, and `Person` JSON-LD.

The visible sections, in order, must be:

```html
<main id="main-content">
  <section class="hero" aria-labelledby="hero-title">徐建硕、专业定位、十年经验简介、资质</section>
  <section class="section" id="expertise" aria-labelledby="expertise-title">六项专业能力</section>
  <section class="section" id="outcomes" aria-labelledby="outcomes-title">代表成果</section>
  <section class="section" id="featured-projects" aria-labelledby="projects-title">五个项目入口</section>
  <section class="section" id="latest-insights" aria-labelledby="insights-title">专业文章入口</section>
  <section class="section" id="career" aria-labelledby="career-title">职业经历摘要</section>
  <section class="section" id="contact" aria-labelledby="contact-title">邮箱与脱敏手机号</section>
</main>
```

The six expertise cards must be named `保险经营`、`精算定价`、`产品开发`、`核保风控`、`渠道平台`、`系统规划`. The contact block must render `xujianshuo@163.com` and `189****5677`, never the complete phone number.

- [ ] **Step 2: Create the career page**

Present the supplied work history in reverse chronology. Group duties under meaningful labels, include dates and roles, and use only confirmed outcomes from the spec. Add education and credentials. Do not include the ambiguous “保单成本90%” wording.

- [ ] **Step 3: Create the about page**

Write a concise first-person biography centered on evidence, data sensitivity, practical execution and cross-functional collaboration. Include professional themes and contact details without birth date or overt job-seeking language.

- [ ] **Step 4: Run focused tests and inspect anchors**

```powershell
node --test tests/site.test.mjs
Select-String -Path index.html -Pattern 'id="expertise"'
```

Expected: tests still fail for planned pages not yet created; `Select-String` returns the expertise section line.

- [ ] **Step 5: Commit Chinese profile pages**

```powershell
git add index.html experience about
git commit -m "feat: add Chinese profile pages"
```

## Task 4: Build Chinese projects and insights

**Files:**
- Create: `projects/index.html`
- Create: all five Chinese project detail paths from the file map
- Create: `insights/index.html`
- Create: `insights/insurance-management/index.html`

- [ ] **Step 1: Create the project index**

Add filter buttons for `all`, `operations`, `actuarial`, `product`, `risk`, `platform`, and `systems`. Each project card must remain visible without JavaScript and contain title, period, role, anonymized summary, category tags and a normal detail link.

- [ ] **Step 2: Create the five project details**

Each page must implement the approved seven-part template: background, problem, personal responsibility, analytical approach, implementation, outcome, and reusable lessons. Use only these evidence boundaries:

- Accident reform: regulatory response, phased product upgrades and compliance control.
- Non-auto quality: end-to-end risk framework; the 9 million input and 30 million loss reduction figures must carry a note that the public wording was confirmed before launch.
- Renewal operations: production scheduling, monitoring, workflow optimization and system planning; no internal rates or rules.
- Platform integration: requirements, policies, integration and production monitoring; partner names only if confirmed public.
- Cost management: pre-assessment, in-process monitoring and post-review; omit the undefined 90% metric.

- [ ] **Step 3: Create the insights index and launch article**

The first article is `从事前评估到事后复盘：保险经营管理的闭环思路`. It explains a generic management framework and contains no company-confidential details. The article page includes publication date, summary, headings and related topics. Because it is the only launch article, previous/next navigation is not rendered.

- [ ] **Step 4: Run tests**

Run `node --test tests/site.test.mjs`.

Expected: remaining failures are limited to missing English pages and global publishing files.

- [ ] **Step 5: Commit Chinese knowledge pages**

```powershell
git add projects insights
git commit -m "feat: add Chinese projects and insights"
```

## Task 5: Create the complete English mirror

**Files:**
- Create: every `en/` path in the file map

- [ ] **Step 1: Create English profile pages**

Use professional English rather than literal translation. The positioning is `Property Insurance, Actuarial & Risk Management Practitioner`. Preserve facts, dates and credential abbreviations. Every page must link back to its Chinese counterpart and use root-relative asset URLs.

- [ ] **Step 2: Create English project and insight pages**

Mirror the structure and evidence boundaries of the Chinese pages. Translate section intent consistently as `Context`, `Challenge`, `My Role`, `Approach`, `Execution`, `Outcomes`, and `Lessons`.

- [ ] **Step 3: Run the bilingual tests**

Run `node --test tests/site.test.mjs`.

Expected: the page-existence, `lang`, `hreflang`, expertise-anchor and privacy tests PASS.

- [ ] **Step 4: Manually verify paired links**

Open `/`, `/en/`, one Chinese project and its English counterpart. Switch languages in both directions and confirm the route preserves the page topic.

- [ ] **Step 5: Commit the English mirror**

```powershell
git add en
git commit -m "feat: add English site"
```

## Task 6: Add recovery, SEO and crawler files

**Files:**
- Create: `404.html`
- Create: `robots.txt`
- Create: `sitemap.xml`
- Create: `.nojekyll`
- Modify: `tests/site.test.mjs`

- [ ] **Step 1: Add a useful 404 page**

Create a bilingual recovery page with links to `/`, `/en/`, `/projects/`, and `/insights/`. It must use the shared CSS and contain no JavaScript dependency.

- [ ] **Step 2: Add crawler files**

Use this exact `robots.txt`:

```text
User-agent: *
Allow: /
Sitemap: https://xu-jianshuo.github.io/sitemap.xml
```

Create `sitemap.xml` with every public URL from the file map, using `https://xu-jianshuo.github.io` and paired `xhtml:link` language alternates. Add an empty `.nojekyll` file.

- [ ] **Step 3: Extend tests for SEO files and internal links**

Add assertions that `robots.txt`, `sitemap.xml` and `.nojekyll` exist; that the sitemap contains `https://xu-jianshuo.github.io/`; and that every local `href` resolves to an existing file or a valid in-page anchor.

- [ ] **Step 4: Run the complete test suite**

Run `node --test tests/site.test.mjs`.

Expected: all tests PASS.

- [ ] **Step 5: Commit publishing metadata**

```powershell
git add 404.html robots.txt sitemap.xml .nojekyll tests/site.test.mjs
git commit -m "feat: add site recovery and SEO"
```

## Task 7: Configure GitHub Pages deployment

**Files:**
- Create: `.github/workflows/pages.yml`

- [ ] **Step 1: Add the Pages workflow**

Use the current official Pages actions and deploy only from `main`:

```yaml
name: Deploy static site to Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - uses: actions/setup-node@v6
        with:
          node-version: 24
      - run: node --test tests/site.test.mjs

  deploy:
    needs: test
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - uses: actions/configure-pages@v5
      - uses: actions/upload-pages-artifact@v4
        with:
          path: .
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

- [ ] **Step 2: Validate the workflow syntax and tests**

Run:

```powershell
node --test tests/site.test.mjs
git diff --check
```

Expected: tests PASS and `git diff --check` prints no errors.

- [ ] **Step 3: Commit deployment configuration**

```powershell
git add .github/workflows/pages.yml
git commit -m "ci: deploy personal site to Pages"
```

## Task 8: Review, publish and verify the live site

**Files:**
- Inspect: all files listed in the file map
- Preserve: `gefei/**`

- [ ] **Step 1: Perform content and privacy review**

Search the complete worktree:

```powershell
rg -n "18926485677|1989年10月|保单成本90%" --glob '!gefei/**'
$unfinishedMarkers = ('TO' + 'DO'), ('T' + 'BD')
Get-ChildItem -Recurse -File -Exclude *.md | Select-String -Pattern $unfinishedMarkers
```

Expected: no matches in public site files or implementation documents except explanatory exclusions in approved documentation; public HTML must contain none.

- [ ] **Step 2: Perform responsive and keyboard review**

Check widths 375px, 768px and 1440px. Tab through skip link, navigation, language switch, filters, cards and contact links. Confirm visible focus, no horizontal scroll, readable long-form typography and reduced-motion behavior.

- [ ] **Step 3: Push and update the draft PR**

Push the implementation branch. Confirm the PR includes the design spec, plan, website, tests and deployment workflow, while `gefei/` has no diff.

- [ ] **Step 4: Merge after review**

Merge the PR into `main`. GitHub Pages deployment intentionally begins only after the merge because the workflow listens to pushes on `main`.

- [ ] **Step 5: Verify GitHub Pages**

Wait for the `Deploy static site to Pages` workflow to succeed, then check:

```text
https://xu-jianshuo.github.io/
https://xu-jianshuo.github.io/#expertise
https://xu-jianshuo.github.io/en/
https://xu-jianshuo.github.io/projects/
```

Expected: HTTP 200 for all four URLs; `#expertise` scrolls to the six-card expertise section; the Chinese and English language switches preserve page context.

- [ ] **Step 6: Record the release**

Add a PR comment containing the successful workflow run URL, the live site URL, the commit SHA deployed, and any content facts deliberately withheld pending confirmation.

