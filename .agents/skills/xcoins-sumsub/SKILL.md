---
name: xcoins-sumsub
description: Explain and maintain Xcoins Sumsub KYC flows, US NonDoc checks, and surname-change exception proposals using recorded workflow evidence.
---

# Xcoins Sumsub

Read references/kyc-process.md before advising on this integration. Distinguish observed configuration from proposed changes and unknowns. Source: user-supplied screenshots on 2026-11-02; no live configuration has been verified. A documentation investigation through the Sumsub plugin is recorded in references/kyc-process.md; keep documentation findings separate from screenshot evidence and untested implementation.

The user's goal is to reduce false rejections when a customer's surname differs, potentially after marriage. Diagnose the actual failing check before changing policy. Do not assume marriage or use sex/gender as exception eligibility. Prefer a surname-discrepancy review route available to any customer.

Preserve DOB, SSN/database verification, duplicate/fraud and other independently applicable checks. First name and DOB alone are insufficient evidence to automatically resolve identity. Keep collecting current legal surname. A surname exception should resolve the discrepancy through review or supported evidence without turning other mismatches into passes.

Do not invent Sumsub field paths, expression syntax, missing-value semantics, rule precedence, UI controls, or preset scope. Ask for the relevant expanded source fields and rule configuration when needed. The visible rule builder specifies a PASS condition followed by 'Otherwise reject step'; do not invert this accidentally. Adding a passing rule does not establish that an existing full-name rejection is overridden.

For changes, propose a narrowly scoped draft and test cases before production use. Confirm whether presets affect other levels and whether checks fail upstream at the database provider. Record approved changes and evidence in the reference file. Never store applicant PII, SSNs, credentials, or production case payloads in this skill. Use synthetic examples.
