# Challenges, Failures & Lessons Learned

## Questions
- Tell me about your biggest mistake.
- Describe a project that did not go to plan.
- When did you find a data-quality problem late?
- How did you handle a missed expectation or deadline?
- What did you change after receiving feedback?

## Real story: regional lead definition mistake
**Situation:** During consolidated marketing reporting, different regions used different lead-grain definitions.
**Mistake:** You initially applied a single lead key (LOB + customer) across all regions, although some regions required campaign + customer.
**Detection:** The mismatch emerged during testing and reconciliation.
**Response:** Raised the issue with the stakeholder, corrected the regional logic and retested edge cases.
**Learning:** Document metric grain and region-specific rules before aggregation; use explicit reconciliation test cases.

## Interview-ready structure
1. Own the mistake plainly.
2. State how you detected it.
3. Explain immediate containment and communication.
4. Describe correction and retesting.
5. Show the prevention step added afterward (only if actually implemented).

## Other challenges to prepare
- Duplicate IDs and inconsistent dimensions across regional sources.
- Large-volume source data requiring aggregation at the correct analytical grain.
- Unclear definition of “product” versus “product category” in FMG.
- Refresh, report performance or deadline incident: supply an actual incident before drafting as fact.

## Avoid
Do not present hypothetical outages, missed deadlines or team conflicts as personal history.
