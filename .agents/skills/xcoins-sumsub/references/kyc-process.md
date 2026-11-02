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
