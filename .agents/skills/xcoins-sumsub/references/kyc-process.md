# Recorded KYC process

## Evidence and limits
User screenshots supplied 2026-11-02. These are snapshots, not a live export. Some labels and connectors are truncated. No production changes have been made. Alleged false rejections after marriage are a user-reported hypothesis; actual rejection events and provider responses have not been inspected.

## Visible workflow
- Phone OTP level: Phone_verification, followed by First Phone Check.
- Virtual/disposable phone labels route to a second-chance phone level and Second Phone Check. A branch combining virtual/disposable labels with review-attempt count greater than 2 leads to Mark Virtual and Final Reject for suspected fraud. Another branch sends back for manual review.
- Phone country United States routes to NonDoc_US; other phone countries route to Standard_ApplicantData. Phone country is a routing input, not proof of residence or nationality.
- Duplicate/regulations-violation paths include Mark Duplicate and Duplicate Phone Redirect; US and non-US routes remain distinct.
- NonDoc_US leads to Mark US (NonDoc_US tag), then US Data Check & Conditions.
- Visible US branches include duplicate/regulations violations; Not found in external database (resubmission); extracted information empty OR extracted TIN begins with 9; extracted age <=25 OR >=65. Full KYC and CS Team Check are downstream destinations. Verify exact connectors in a readable export before editing. Young or Elderly is a visible tag.
- Standard_ApplicantData leads to Mark SA and SA Age & Country Check, with age <=25 or >=65, phone country Brazil, and duplicate/regulations branches.
- Full KYC uses basic-kyc-level. Last Checks includes duplicate/regulations labels, device IP/phone country mismatch, a truncated document-country/IP-country cross-check label, device risk labels, a truncated country list, and extracted age >=75. CS Team Check is manual review. Do not reconstruct truncated lists or terminal approval behavior.
- Separate start node NonDoc_US_ExistingUsers is visible; downstream behavior is not established.

## NonDoc_US level
Database Validation: USA SSN Verification.
Questionnaire: ConsentZH; auto-accept previously completed questionnaire appears unchecked.
Required applicant fields: Country, First name, Last name, Date of birth.
Required custom field: partialTin, displayed as SSN (last 4 digits).

## Comparison presets shown
Basic screenshot title: Name: weakContainment, Address: fuzzy. Flexible name/address options selected; middle-name-ignore and ignore-manual-vs-document-data boxes appear unchecked.
A separate New preset advanced screen lists 24 custom cross-check rules and 174 system rules. Do not assume this draft is active or identical to the basic preset.
Visible rules cover full name, DOB and gender across info/e-kyc and info/fixedInfo; address and TIN info/fixedInfo; DOB and gender across info/poi, poi2, poi3.
Expanded Full Name [info <-> e-kyc]: namesCompatibleOrNull, extracted name consistency data versus name consistency data from eKYC, comparisonMode AI-assisted, ignoreMiddleNameMismatch false. Otherwise rejects Applicant data > Reject applicant > E-KYC > Data mismatch with violation Name doesn't match.
Expanded Date of Birth [info <-> e-kyc]: dobCompatibleOrNull on extracted DOB and eKYC DOB (YYYYMMDD). Otherwise rejects through the same data-mismatch path with violation Date of birth doesn't match.
'OrNull' suggests missing-value handling but exact behavior is unverified. Do not treat this rule alone as proof both dates are present or exactly equal.
Rule builder exposes If/If not, And, Or group, and Otherwise reject step. Top-level data-source choices include action, aiInquiries, applicant, checks, poi, poi2 and poi3. First-name fields within the eKYC result have not been shown.

## Proposed exception (not implemented or approved)
Name: US NonDoc - surname discrepancy review.
Description: Route isolated surname discrepancies for review while retaining first-name, DOB, SSN/database and applicable fraud checks.
Eligibility: first name and DOB are present and match under an explicitly defined policy; SSN/database verification otherwise succeeds; surname is the isolated discrepancy. No eligibility based on gender. Missing data, first-name/DOB/SSN mismatches or independent fraud flags follow their existing escalation paths.
Surname-only discrepancy should reach manual review or an appropriate Full KYC/evidence route, not automatic acceptance based solely on first name and DOB. Verify how step rejection interacts with workflow routing before wiring it.
Do not globally disable full-name comparisons or use Ignore data mismatch as a surname exception. Flexible/AI name comparison does not establish support for entirely different surnames. Ignoring middle names does not solve surname changes.
Confirm preset scope, editor field support, all other applicable full-name rejection rules, provider-side rejection behavior, and existing system-rule interactions before implementation.

## Synthetic validation matrix
- Same first name, DOB, surname and successful database verification: normal route.
- Same first name and DOB, successful database verification, different surname only: review route.
- Different first name or DOB: retain mismatch handling.
- Missing first name or DOB: no exception acceptance.
- Failed SSN/database check or record not found: retain existing handling.
- Duplicate/fraud/system-rule flag plus surname discrepancy: retain independent escalation.
- Middle-name-only difference and hyphenated surname: test separately from surname replacement.
- Multiple conflicting records: review; no assumed unique identity match.
- Unrelated non-US and existing-user levels: verify no unintended preset changes.

## Next evidence needed
An expanded applicant/checks source picker showing first-name and DOB fields for the eKYC result; one redacted rejection trace identifying the actual rejecting rule/provider; preset assignment/scope. Never request real SSNs or unredacted applicant records.

## Documentation investigation — 2026-11-02

Method: read README and this skill on main, then queried the Sumsub plugin's how_to_sumsub tool about exact info/eKYC selectors, provider outcomes, level assignment and rule interaction. A follow-up corrected two overstatements in the initial answers: exact DOB equality is not explicitly guaranteed, and surname-only mismatch is not universally a provider RED. Findings below are documentation-backed answers returned by the plugin, not a live configuration inspection or successful sandbox test.

### Findings and sources

1. First-name-only info ↔ eKYC is not confirmed. The documentation describes applicant.info.firstName and applicant.fixedInfo.firstName, but these are not a verified pair for the recorded info ↔ e-kyc rule. Do not substitute info ↔ fixedInfo. No confirmed cross-check selector for the US SSN result's first name or DOB was returned. Raw API extractedDoc fields are not proof they are selectable in the cross-check editor. namesCompatibleOrNull has no documented ignore-surname option. See [variables](https://docs.sumsub.com/docs/cross-check-rules-variables), [SSN products](https://docs.sumsub.com/reference/usa-tin-check), and [functions](https://docs.sumsub.com/docs/cross-check-rules#how-cross-check-rules-work).

2. Documented operators include "equals ignore case" and "isn't empty". "is empty" covers null, empty string and empty collection; use presence checks on both first names and both DOBs once their actual selectors are verified. Whitespace normalization is not established. First-name case-insensitive equality is a proposed policy, not the existing AI-assisted name semantics. See [equality operators](https://docs.sumsub.com/docs/cross-check-rules-operators#equality-operators) and [other operators](https://docs.sumsub.com/docs/cross-check-rules-operators#other-operators).

3. dobCompatibleOrNull is documented as "Compares two dates of birth provided as integers in YYYYMMDD format. Returns true if either value is null." It therefore does not establish presence. The quoted definition does not explicitly guarantee exact equality or describe tolerance. Do not infer exact-match semantics from YYYYMMDD representation. Obtain confirmed date equality/operator behavior before implementing an exact-DOB policy. See [function definition](https://docs.sumsub.com/docs/cross-check-rules#how-cross-check-rules-work).

4. Existing full-name rejection must be addressed in the isolated draft. A new passing first-name/DOB rule is not evidence that Full Name [info ↔ e-kyc] stops rejecting. Advanced settings distinguish Custom cross-check rules (configurable) and System rules (not editable). If/If not, And, and Or group are documented; Otherwise reject step fires when no pass-condition group succeeds. No override/ordering guarantee was established. Proposed change: replace or narrow the relevant custom full-name rejection in the draft only after the supported eKYC fields and outcome logic are confirmed; inspect other independent name rejection rules and retain other checks. Do not claim a particular disable/delete control or override syntax without editor evidence. See [preset configuration](https://docs.sumsub.com/docs/cross-check-rules#configure-preset).

5. Provider mismatch is a separate possible failure. TIN API documentation exposes answer, tinCheckInfo.tinValidity, tinCheckInfo.personDataMatchFound and violations including NAME_MISMATCH, BDATE_MISMATCH and DATA_NOT_FOUND. Examples show name mismatch can yield RED for particular products. They do not guarantee surname-only mismatch always yields RED, prove evaluation order, or map the Dashboard label USA SSN Verification to the actual account product. Inspect the actual product and a redacted check/rule trace. Rejection labels alone may be reused and are not sufficient attribution. A local preset does not establish that provider matching can be changed. See [TIN response](https://docs.sumsub.com/reference/get-tin-check-results#response-explained), [SSN product examples](https://docs.sumsub.com/reference/usa-tin-check), and [rejection buttons](https://docs.sumsub.com/docs/rejection-explanatory-buttons#external-databases-rejects).

6. Database Validation / 1x1 matching documentation says data matching rules are standard and not available for customization. Do not conflate that provider methodology with configurable local cross-checks. No documented provider setting for first name + exact DOB + last-four SSN while ignoring surname was identified. Product-specific support confirmation is required. See [matching configurations](https://docs.sumsub.com/docs/database-validation-matching-configurations).

7. Documented level assignment: create a separate/duplicated preset under Global settings → Integrations → Data Comparison; then Integrations → Verification levels → NonDoc_US → Configurations → Cross-check rules → select preset → Save. Preset changes affect every level using that preset. A new isolated preset assigned only to NonDoc_US avoids changing a shared preset. Verify actual assignments; do not infer current scope from the screenshot. Leave NonDoc_US_ExistingUsers and non-US levels on their existing presets. See [create and assign preset](https://docs.sumsub.com/docs/cross-check-rules#create-new-preset).

8. Manual review is a proposed optional policy, not an approved requirement. Workflow Builder documents review steps; a Data Comparison rule's Otherwise reject step is not documented as a direct manual-review or level-change action. Confirm available workflow signals, rejection behavior and connectors before proposing exact wiring to CS Team Check or Full KYC. Do not apply generic Transaction Monitoring rule actions to this KYC workflow without evidence. See [workflow review steps](https://docs.sumsub.com/docs/get-started-with-workflow-builder#set-up-review-steps).

### Proposed policy — not a deployable expression

Eligibility requires both first names present and matching under a chosen policy, both DOBs present and exactly matching under confirmed comparison semantics, successful applicable SSN/database verification, and no independent fraud/duplicate blockers. Surname-only difference is the targeted exception. Whether it should proceed or receive additional review requires policy agreement; review is a recommendation only. Keep collecting current legal surname and last-four SSN. This prose is not Sumsub expression syntax.

A deployable rule remains blocked on supported eKYC selectors, exact-DOB behavior, provider product/outcomes and the handling of existing full-name rejection. No live settings were changed.

### Synthetic tests to run after those gaps are resolved

These are proposed cases, not executed results. Baseline synthetic profile: first name Alex, surname Morgan, DOB 1990-04-12; SSN check result simulated, no actual SSN stored.

| Case | Inputs / condition | Expected policy result |
|---|---|---|
| Baseline | Alex Morgan / Alex Morgan; DOB equal; DB succeeds | Existing normal route |
| Surname only | Alex Morgan / Alex Taylor; DOB equal; DB otherwise succeeds | Candidate surname exception; review only if adopted |
| First name | Alex / Jamie; DOB equal | Existing mismatch handling |
| DOB day/month/year | Separately vary day, month and year | No exact-DOB exception |
| Missing first name | Each side separately null or empty | No exception |
| Missing DOB | Each side separately null or empty | No exception, despite OrNull behavior |
| Case/spacing | Alex / ALEX, then Alex with whitespace | Check chosen case policy; establish whitespace behavior |
| Provider failure | SSN invalid, record not found, NAME_MISMATCH, unavailable | Preserve applicable failure/escalation; no local-rule bypass |
| Independent flags | Surname only plus duplicate/fraud/system rejection | Preserve independent handling |
| Multiple records | Conflicting matching records | No assumed unique identity; review proposed |
| Other name forms | Middle-name-only, hyphenated names, transliteration | Evaluate separately; no presumed surname exception |
| Scope regression | Non-US, existing-users and other shared-preset levels | Unchanged configuration and routing |

### Support questions ready to send — not sent

- For this account's USA SSN Verification product, what exact cross-check editor selectors expose first name and DOB on each side of info ↔ e-kyc, and are both available as scalar operands?
- Which supported operator/function enforces exact full-date equality with explicit presence, and what normalization/tolerance does dobCompatibleOrNull apply?
- Can provider verification succeed for a surname-only discrepancy with matching first name, DOB and last-four SSN? What product code and field-level results apply?
- How should the existing custom full-name rejection be replaced/scoped, what independent system rules remain, and how can a surname-only outcome be exposed to this KYC workflow for optional review without automatic rejection?

Plugin capability feedback was submitted without applicant data for the unavailable selectors and ambiguous semantics. No support message was sent.
