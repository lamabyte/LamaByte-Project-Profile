# Engineering notes

These notes explain how I built lamabyte.systems, why I made the choices I did, and how I checked the site before and after release. The code samples were written for this profile to illustrate how the site works.

**On this page**

- **How I built it** · [Architecture](#static-pages-and-an-interactive-application) · [Workflow state](#predictable-workflow-state) · [Motion](#motion-across-the-site) · [Homepage effects](#interactive-homepage-effects)
- **How I tested it** · [Tools](#testing-performance-and-seo) · [Release verification](#release-verification) · [Search visibility](#search-visibility)
- **Performance** · [Lighthouse](#lighthouse-results) · [Responsiveness](#animation-and-responsiveness)

## Static pages and an interactive application

I used Astro to build the English and Spanish pages as HTML, so visitors can read them before any application code runs. I made the operations application a React component that becomes interactive as it approaches the viewport, so its JavaScript loads only when a visitor is close to it. The five scenarios run in the browser on illustrative records bundled with the site. The application keeps scenario progress in memory and resets it on reload.

This sketch shows how an Astro page can hydrate a React component shortly before it scrolls into view.

```astro
---
import WorkflowShowcase from '../components/WorkflowShowcase.tsx';
---

<section aria-labelledby="workflow-heading">
  <h2 id="workflow-heading">Try a workflow</h2>
  <WorkflowShowcase client:visible={{ rootMargin: '200px' }} />
</section>
```

## Predictable workflow state

I used TypeScript to model the five scenarios and the actions visitors can take in each. Explicit state transitions make it easy to test which actions are valid, and they give visitors a predictable reset when they want to try a workflow again.

```ts
type Stage = "open" | "follow-up" | "closed";
type Action = "queue" | "resolve" | "reset";

function advance(stage: Stage, action: Action): Stage {
  if (action === "reset") return "open";
  if (stage === "open" && action === "queue") return "follow-up";
  if (stage === "follow-up" && action === "resolve") return "closed";
  return stage;
}
```

## Motion across the site

I designed the English and Spanish pages to feel animated while staying easy to scan and use. Sections fade and rise into view as visitors scroll, selected headings reveal their text with a typewriter effect, and cards and buttons use short hover transitions. The same motion patterns carry across pages such as Solutions, Process and About.

Scroll reveals use `IntersectionObserver` and run once, when a section first comes into view. Opacity and transform transitions leave the layout in place. Keyboard focus also triggers a reveal, and visitors who prefer reduced motion see the text and sections right away.

## Interactive homepage effects

I created the 3D llama meadow as a separate animation built on Three.js, then brought it into the homepage hero. I made the scene load only on devices with hardware-accelerated WebGL 2, and only after the page itself has loaded, while the hero is on screen. A plain dark background serves as the fallback for unsupported graphics and the browser’s data-saving mode. The scene has a pause control and pauses itself for visitors who prefer reduced motion.

Farther down the homepage, I wanted the floating llama to follow a finger while the browser could still scroll and zoom normally. Passive touch listeners feed the same movement loop used for a desktop pointer. The effect moves with CSS transforms, so it follows the finger without shifting the layout. When the llama and its glow pass behind the three recovery cards, the cards cover both.

```ts
function trackTouch(
  surface: HTMLElement,
  setTarget: (x: number, y: number) => void,
) {
  surface.addEventListener(
    "touchmove",
    ({ touches }) => {
      const finger = touches.item(0);
      if (finger) setTarget(finger.clientX, finger.clientY);
    },
    { passive: true },
  );
}
```

## Testing, performance, and SEO

| Tool                     | What I used it for                                                                          |
| ------------------------ | ------------------------------------------------------------------------------------------- |
| Vitest                   | Unit tests for application state, content data and build behavior                           |
| Playwright               | Browser tests for navigation, language switching, booking, touch behavior and the live site |
| axe-core with Playwright | Automated accessibility scans, alongside keyboard and reduced-motion tests                  |
| Lighthouse and Chrome    | Repeatable audits of performance, accessibility, best practices and SEO                     |
| Cloudflare Web Analytics | Field monitoring of loading speed and layout stability after release                        |

### Release verification

These results come from the November 2026 launch and maintenance releases.

| Check              | Result                                                                                                                                                                                                                                                                      |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Unit tests         | **1,411 of 1,411** passed in release CI for the maintenance release, and again for the mobile touch update.                                                                                                                                                                 |
| Browser regression | **800 of 800** passed for the maintenance release. The cloud environment covered 774 cases, including five focused reruns. The remaining 26 cases passed on my Windows lab PC after the cloud environment hit graphics and rendering-timing limitations.                    |
| Live site          | **299 of 299** passed in the final production run of the public-site suite, covering navigation, HTTP responses, both 404 pages and the booking calendar. The reported count covers the selected browser and HTTP checks.                                                   |
| Accessibility      | axe-core found **0 violations** in **60 automated scans** on the release candidate before launch, covering all 30 pages at desktop (1440 × 900) and phone (384 × 854) sizes. I also checked 200% zoom and keyboard use by hand and used Windows Narrator in both languages. |
| Browsers           | I checked **five browser environments** during release verification, covering Chrome, Edge and Firefox on Windows, Chrome on Android and Safari on iPhone.                                                                                                                  |
| Spanish content    | All **914** Spanish text units are approved, and the production build stops if any English and Spanish pair changes after approval.                                                                                                                                         |
| Hosting            | **205 of 205** checks passed against the build served by Cloudflare's local Workers runtime: every page, redirect, error page and security header, and the content security policy.                                                                                         |
| Integrations       | **18 of 18** checks passed against mock analytics and error-reporting services, covering all nine analytics events and confirming that the tested request payloads excluded planted personal text.                                                                          |
| Security scans     | **0** enforced content security policy violations across the 31 HTML files served at launch, and **0** findings in release CI's secret and environment-file scans. The scans cover the tracked files and checks used in release CI.                                         |

The first live run of the maintenance release passed 298 of 299 checks. The English booking calendar timed out at the 45-second limit. I rolled back to the previous version and confirmed the calendar there, then released the same version again. Its focused booking check passed, and the full suite then passed 299 of 299. The timeout’s cause remains unknown.

### Search visibility

Each indexable page has a canonical URL and `hreflang` links to its counterpart in the other language, and the XML sitemap lists the same pairs. [Live XML sitemap](https://lamabyte.systems/sitemap.xml)

I checked how Google and Bing saw the site at launch in November 2026.

- **Google Search Console** read the sitemap and discovered all 28 pages. URL Inspection showed the Spanish homepage as indexed, and the English homepage passed the live test.
- **Bing Webmaster Tools** discovered all 28 URLs from the sitemap, and both homepages passed Bing's live indexability test.
- **Google's Rich Results Test** found one valid structured-data item on each of the nine pages tested.

## Lighthouse results

**How these audits were run.** Both sweeps audited the same 28 indexable pages of the November 25, 2026 release, served from a local test server, with Lighthouse 13.4.1 in headless Chrome and one run per page. The server sent Brotli-compressed text, and analytics and error reporting were disabled. With WebGL disabled, the lab measured the plain hero background.

Accessibility and SEO scored 100 on every page in both profiles. Each cell lists the English result first and the Spanish result second.

### Mobile Lighthouse results

Simulated mobile network and CPU throttling.

| Page pair                 | Performance | Best practices |     LCP (s) |           CLS |
| ------------------------- | ----------: | -------------: | ----------: | ------------: |
| Home                      |     98 / 96 |      100 / 100 | 2.41 / 2.41 |         0 / 0 |
| Solutions                 |   100 / 100 |      100 / 100 | 1.81 / 1.80 |         0 / 0 |
| Revenue Recovery Systems  |    99 / 100 |      100 / 100 | 1.80 / 1.80 |         0 / 0 |
| Operations Automation     |   100 / 100 |      100 / 100 | 1.80 / 1.80 |         0 / 0 |
| Custom Applications       |   100 / 100 |      100 / 100 | 1.80 / 1.81 |         0 / 0 |
| Operations Application    |     98 / 98 |      100 / 100 | 2.41 / 2.41 |         0 / 0 |
| Residential Services      |     99 / 99 |      100 / 100 | 1.80 / 1.80 |         0 / 0 |
| Light-Commercial Services |   100 / 100 |      100 / 100 | 1.80 / 1.80 |         0 / 0 |
| Process                   |   100 / 100 |      100 / 100 | 1.80 / 1.80 |         0 / 0 |
| About                     |    99 / 100 |      100 / 100 | 1.83 / 1.80 |         0 / 0 |
| Book a Discovery Call     |     99 / 99 |        96 / 96 | 1.95 / 1.95 | 0.011 / 0.052 |
| Privacy Policy            |   100 / 100 |      100 / 100 | 1.65 / 1.65 |         0 / 0 |
| Terms of Service          |   100 / 100 |      100 / 100 | 1.65 / 1.65 |         0 / 0 |
| Accessibility             |   100 / 100 |      100 / 100 | 1.65 / 1.66 |         0 / 0 |

| Audit                  | Range across all 28 pages |
| ---------------------- | ------------------------: |
| First Contentful Paint |               1.02–1.36 s |
| Speed Index            |               1.02–1.70 s |
| Total Blocking Time    |                  0–159 ms |

Largest Contentful Paint (LCP) measures when the main visible content appears. Cumulative Layout Shift (CLS) measures visual stability.

The booking pages scored **96 for Best Practices** in both local profiles. Each desktop report recorded a failed request for the third-party calendar script. The earlier live launch audit loaded the calendar and scored both pages 100 for Best Practices.

### Desktop Lighthouse results

The desktop preset used a 1,350 × 940 viewport with simulated desktop network settings.

| Page pair                 | Performance | Best practices |     LCP (s) |       CLS |
| ------------------------- | ----------: | -------------: | ----------: | --------: |
| Home                      |   100 / 100 |      100 / 100 | 0.54 / 0.55 |     0 / 0 |
| Solutions                 |   100 / 100 |      100 / 100 | 0.40 / 0.40 |     0 / 0 |
| Revenue Recovery Systems  |   100 / 100 |      100 / 100 | 0.40 / 0.40 |     0 / 0 |
| Operations Automation     |   100 / 100 |      100 / 100 | 0.40 / 0.40 |     0 / 0 |
| Custom Applications       |   100 / 100 |      100 / 100 | 0.40 / 0.41 |     0 / 0 |
| Operations Application    |   100 / 100 |      100 / 100 | 0.53 / 0.52 |     0 / 0 |
| Residential Services      |   100 / 100 |      100 / 100 | 0.44 / 0.44 |     0 / 0 |
| Light-Commercial Services |   100 / 100 |      100 / 100 | 0.40 / 0.40 |     0 / 0 |
| Process                   |   100 / 100 |      100 / 100 | 0.41 / 0.40 |     0 / 0 |
| About                     |   100 / 100 |      100 / 100 | 0.40 / 0.41 |     0 / 0 |
| Book a Discovery Call     |   100 / 100 |        96 / 96 | 0.51 / 0.51 | 0.003 / 0 |
| Privacy Policy            |   100 / 100 |      100 / 100 | 0.40 / 0.37 |     0 / 0 |
| Terms of Service          |   100 / 100 |      100 / 100 | 0.36 / 0.40 |     0 / 0 |
| Accessibility             |   100 / 100 |      100 / 100 | 0.40 / 0.40 |     0 / 0 |

| Audit                  | Range across all 28 pages |
| ---------------------- | ------------------------: |
| First Contentful Paint |               0.26–0.35 s |
| Speed Index            |               0.26–0.61 s |
| Total Blocking Time    |                      0 ms |

### Earlier audits

- **Maintenance comparison, November 2026.** Before release, the old and new builds were compared on six pages covering Home, Operations Application and Revenue Recovery Systems in both languages. Each page had three mobile runs on a local server with WebGL off. The new build's medians were 97–100 for performance and 100 for accessibility, best practices and SEO, with LCP of 1.80–2.56 seconds and CLS of 0 on each page. The comparison measured regressions between builds served locally.
- **Live audit at launch.** Three mobile runs per page on eight pages of the live site, with analytics blocked and the booking calendar loaded, gave median performance scores of 97–100 and 100 for accessibility, best practices and SEO. Median CLS was 0.0999 on the English booking page, just under the 0.1 threshold for a good score, and 0.052 on the Spanish one, both from the calendar frame. The other six pages had a median CLS of 0.

The live-site suite also checked that missing URLs return HTTP 404 with the correct error page in each language.

## Animation and responsiveness

- **Layout stability.** After launch, a small Cloudflare Web Analytics sample highlighted the homepage glow in its layout-shift data. At the time the glow moved with `left` and `top`. I switched the glow to CSS transforms. In scripted lab runs, the new build showed **0 glow-attributed layout shifts in all 10 cases**, where the previous build showed 1 to 5 in each.
- **Touch coverage.** Focused touch tests in emulated mobile Chrome passed on **both homepages**, and screenshot comparisons showed the llama and glow passing behind **all three recovery cards** in each language. I then confirmed it on my own phone on the live site.
- **Input timing.** In scripted lab sequences after the page had loaded, Chrome's Event Timing API recorded **all 700 interactions** on the maintenance build. I used the slowest interaction in each run to calculate the median for each test profile. Those medians ranged from **56 to 144 ms**, with the upper end on a phone profile with fourfold CPU slowdown.
- **JavaScript budgets.** The release build with analytics and error reporting stayed within its JavaScript budgets. The measured homepage interactive scripts used **10,354 of 10,496** Brotli bytes, and the combined public script modules used **11,808 of 12,032**. The React application has its own budget.

[Back to the project overview](README.md)
