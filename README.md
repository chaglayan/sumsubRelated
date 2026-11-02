# Xcoins Sumsub KYC context

Start with [.agents/skills/xcoins-sumsub/SKILL.md](.agents/skills/xcoins-sumsub/SKILL.md) and its [recorded workflow](.agents/skills/xcoins-sumsub/references/kyc-process.md).

These notes transcribe configuration visible in screenshots supplied in the original Codex conversation on 2026-11-02. The original image attachments were not accessible as files and are NOT included. Treat truncated workflow labels and unverified connectors as unknown.

## Current task for the Sumsub-connected agent

Use the Sumsub plugin to investigate a custom preset for **NonDoc_US / USA SSN Verification**, comparing applicant data against the **eKYC database result**. This is not primarily an ID-document comparison or AML screening question.

The user reports false rejections when surnames differ, possibly following marriage, and wants first-name and DOB comparison while retaining SSN verification. A review route is an assistant recommendation, not an approved user requirement or an implemented configuration.

Visible rejecting rule: **Full Name [info ⇄ e-kyc]**, `namesCompatibleOrNull`, extracted name consistency data against eKYC name consistency data; AI-assisted; ignoreMiddleNameMismatch=false. Failure rejects Applicant data → Reject applicant → E-KYC → Data mismatch.

Visible DOB rule: **Date of Birth [info ⇄ e-kyc]**, `dobCompatibleOrNull`, extracted DOB against eKYC DOB, formatted YYYYMMDD. Verify missing-value semantics from authoritative documentation.

Resolve with documented evidence:
1. Is first-name-only comparison supported for info ⇄ e-kyc under USA SSN Verification?
2. Give exact source fields, functions, operators, and UI selections, including presence checks for both first names and DOBs. Do not invent syntax.
3. Explain how to replace/scope the existing full-name rejection so it does not independently reject the same case.
4. Determine whether provider-side surname mismatches can fail verification upstream of custom cross-checks.
5. Explain how the preset is assigned only to the intended US level and how a surname-only discrepancy can route to review if desired.
6. Supply synthetic positive, mismatch, missing-value, and unrelated-level regression cases.

Do not modify live Sumsub settings. Clearly separate documented capability, proposed policy, and items requiring Sumsub support confirmation. Update the reusable skill with verified findings and sources.
