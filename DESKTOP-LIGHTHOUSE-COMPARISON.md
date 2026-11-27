# Desktop Lighthouse comparison

I audited the same 28 indexable pages in English and Spanish at the Lighthouse desktop preset, 1080p, 1440p, and 4K. Accessibility and SEO scored **100** on every page at every size. Best Practices scored **100** on 26 pages and **96** on both booking pages in each sweep. The booking reports recorded the same third-party calendar script error in the local test browser.

| Desktop viewport             | Performance | Median LCP | Slowest LCP | Median Speed Index | Peak TBT | Peak CLS |
| ---------------------------- | ----------: | ---------: | ----------: | -----------------: | -------: | -------: |
| 1,350 × 940, original preset |         100 |     404 ms |      549 ms |             282 ms |     0 ms |    0.003 |
| 1,350 × 940, control rerun   |         100 |     404 ms |      547 ms |             283 ms |     0 ms |    0.015 |
| 1,920 × 1,080, 1080p         |         100 |     404 ms |      549 ms |             283 ms |   3.5 ms |    0.017 |
| 2,560 × 1,440, 1440p         |         100 |     403 ms |      546 ms |             290 ms |     0 ms |    0.043 |
| 3,840 × 2,160, 4K            |      99–100 |     405 ms |      550 ms |             342 ms |    12 ms |    0.078 |

The main content appeared at essentially the same speed across all four sizes. The median page-by-page change in Largest Contentful Paint (LCP) was under 1 ms in either direction for each wider viewport compared with the original preset. Speed Index, which measures how quickly the visible page fills in, was higher at 4K. At that size, 25 pages scored 100 for Performance. Both Process pages and the English booking page scored 99 in the full sweep after their FAQ sections shifted.

## What the repeat runs showed

- **Spanish homepage.** Median Speed Index across three runs was 565 ms at the preset size, 601 ms at 1080p, 672 ms at 1440p, and 880 ms at 4K. One of the three 4K runs scored 99 for Performance.
- **Spanish Process page.** The FAQ section shifted in all three 4K runs, producing 0.063 Cumulative Layout Shift (CLS) each time. That page recorded 0.043 in all three 1440p runs and 0 at both narrower sizes.
- **English Process page.** The FAQ shift produced 0.061 CLS in two of three 4K runs. Those runs scored 99 for Performance. The third scored 100 with 0 CLS.
- **Booking pages.** The English FAQ section shifted in all three 4K runs, producing 0.078 CLS and a Performance score of 99 each time. The Spanish booking page scored 100 for Performance in all three runs, with CLS ranging from 0.002 to 0.060.

All recorded layout shifts stayed below the 0.1 threshold for good CLS. Total Blocking Time (TBT) had a median of 0 ms in every sweep.

## How I tested

I used the same checksum-verified release build for each viewport. Lighthouse 13.4.1 ran in headless Chrome 158 at device pixel ratio 1 with simulated desktop network settings and a local server serving Brotli-compressed text. The build used the static hero fallback with WebGL, analytics, and error reporting disabled.

The table summarizes one audit per page and viewport. I reran the preset as a control, then used the same configuration for the later 4K session. The repeat runs above checked specific pages where timings or layout shifts merited a closer look.

[Engineering notes and per-page preset results](ENGINEERING.md#desktop-lighthouse-results)
