<h1 style="font-size:2.6rem;line-height:1.1;">U.S. tax return App</h1>

**Snap your tax documents. Get your federal return, explained in English, Spanish or Chinese. Read every line of the code behind it.**

<p class="try-cta"><a href="https://hub.smartonlabs.com">Try Libertas →</a></p>

<ul class="stat-strip">
<li class="stat-total"><strong>345,700</strong> <span>lines of code, written by AI</span></li>
<li><strong>~100,200</strong> <span>lines of tax calculation</span></li>
<li><strong>~36,800</strong> <span>lines of input validation</span></li>
<li><strong>~49,400</strong> <span>lines of explanations, scope and record handling</span></li>
</ul>

<ul class="stat-strip">
<li><strong>190</strong> <span>different full screens, including each document editor</span></li>
<li><strong>7,911</strong> <span>fields a filer can see</span></li>
<li><strong>12,099</strong> <span>fields in the public schema</span></li>
</ul>

<ul class="stat-strip">
<li><strong>59,317</strong> <span>localized strings</span></li>
<li><strong>11,324</strong> <span>runtime messages</span></li>
<li><strong>1,107</strong> <span>lines of display rules</span></li>
</ul>

[Read the metrics](https://github.com/LibertasIoT/libertas-tax/blob/main/CODE-METRICS.md)

<p class="section-claim"><strong>AI writes as much as it can.<br>People maintain as little as they&nbsp;must.</strong></p>

## Where the code goes

Most of the App is tax law. Screens, storage and the connection to Libertas
together take 0.7% of the code.

| What the App needs | Lines | Share |
| --- | ---: | ---: |
| Tax calculation, validation and explanations | 186,455 | 53.9% |
| Schema and data declarations | 61,131 | 17.7% |
| Interview, navigation and page mapping | 58,631 | 17.0% |
| Localized runtime messages | 30,698 | 8.9% |
| Return persistence | 1,153 | 0.33% |
| Display rules for 190 screens | 1,107 | 0.32% |
| Libertas platform integration | 154 | 0.04% |

Reference tables, wiring and shared helpers make up the remaining 1.9%.

Libertas packs the App's information into very few lines:

- **Platform integration: 154 lines.** Eleven functions form the App's boundary
  with Libertas: they take each request from the Hub and read and write the
  return through the Data API. Screens, AI Import, translation and access
  control come from the platform.
- **Calculation and validation.** Of the 186,455 tax lines, about 100,200
  calculate the return and about 36,800 check the facts entered. The other
  49,400 or so explain each figure with its form line and IRS source, handle
  what is out of scope, and select and convert records. Calculation helpers
  in the interview code add about 7,000 lines, for about 107,300 lines of
  calculation in all.
- **Schema fields: 20,346 declared.** They take 61,131 lines, internal records
  included: about three lines per field with its type, limits and checks.
  The public schema has 12,099 fields, 7,911 of them visible to a filer.
  Labels and help are doc comments, outside the line count.
- **Localized strings and runtime messages: 59,317 strings.** Every label, help
  text and message can be translated. 11,324 of them are runtime messages,
  form line labels included, that the code selects as it runs, many with
  placeholders such as a note's amounts. They take 30,698 lines.
- **Screens: 190, arranged by 1,107 lines.** Every screen comes from the
  declarations; display rules arrange them, under six lines a screen. The
  interview code decides which questions apply and converts answers to stored
  records; it draws nothing.

Counts are from the
[metrics report](https://github.com/LibertasIoT/libertas-tax/blob/main/CODE-METRICS.md),
excluding tests. Calculation and validation lines are
estimates from reviewed random samples.

## Every figure explains itself

Every line the App calculates comes with its explanation: how the amount was
worked out, the IRS form, instructions or publication it follows, and the
entries and earlier lines it came from.

When the App can't calculate something, it doesn't guess. Each gap is its own
note on the Review page. The note says what is out of reach and links to the
page where you enter the missing fact. Nothing silently counts as zero.

## Document center: your paperwork is the interview

Most tax software makes you retype your forms. Here you start with the
documents themselves. Photograph or upload your W-2s, 1099s, K-1s and other
statements, and AI reads each one into the matching entries. You check the
result and save it; nothing is accepted without you. Each document stays
listed under whose it is, and every figure on the return traces back to it.

The interview then asks only about what your documents cannot say: your
household, your dependents and your situation.

## Written by AI. Open line by line.

All of it is [public on GitHub](https://github.com/LibertasIoT/libertas-tax).
Every rule, limit and calculation is source code you can read, with the IRS
form or publication it comes from.

## Coverage: TurboTax-level, and closing fast

**The 80/20 rule:** 80% of the code serves only 20% of what filers need. Wages,
interest and the standard deduction take a small part of it; farms, rentals,
K-1s and foreign income take the rest. AI writes that long tail, so one App
can aim for the coverage of TurboTax. Changes land every day.

- **Federal, today.** Form 1040 with its schedules, worksheets and credit
  forms: wages, business, farm, rental, investments, retirement, foreign
  income, education, health coverage, and much more. The
  [coverage audit](https://github.com/LibertasIoT/libertas-tax/blob/main/COVERAGE.md)
  lists every form and its limits.
- **State, next.** State and local tax is not calculated yet. The state boxes
  on your W-2s and 1099s are already kept for it.

It is a 2026 estimate on the IRS draft forms and tables, with synthetic data.
It does not print or file a return.

## Try it with sample papers

**No retyping. Hand the App the paperwork and watch AI read it.**

Two synthetic households are ready for you:

1. **[Casey Sample](https://github.com/LibertasIoT/libertas-tax/tree/main/samples/single-w2-2026):
   simple, 2 papers.** Single, no dependents: one W-2 and a bank's 1099-INT.
   The quickest way to a whole return.
2. **[The Smith family](https://github.com/LibertasIoT/libertas-tax/tree/main/samples/smith-family-2026):
   a 65-paper extravaganza.** Two incomes, a retirement, a college student, a
   small farm, a rental, stock sales and K-1s, on 37 kinds of tax papers. The
   latest automatic QA walk took all 65 to an estimate, with nothing left on
   Review.

To try one:

1. **Download** its papers.
2. **Open the App** on the Libertas Cloud Hub, on your phone or in your browser.
3. **Tap Import with AI** in the Document center and pick up to eight papers.
   AI works out what each paper is, reads its boxes and files it in the right
   place.
4. **Check and save.** Nothing goes in until you approve it.
5. **Answer what no paper says,** and get an estimate where every line explains
   itself.

AI Import and AI Search are on in the Libertas Cloud Hub, within a monthly
usage cap.

<p class="try-cta"><a href="https://hub.smartonlabs.com">Try Libertas →</a></p>
