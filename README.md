# [Sponsorname] Audit

Unless otherwise discussed, this repo will be made public after audit completion, sponsor review, judging, and issue mitigation window.

**Contributors to this repo:** prior to report publication, please review the [Agreements & Disclosures](/issues/1) issue.

---

## Audit findings are submitted to this repo

Sponsors have three critical tasks in the audit process:

1. Respond to issues.
2. Weigh in on severity.
3. Share your mitigation of findings.

Let's walk through each of these.

## High and Medium Risk Issues

Wardens submit issues without seeing each other's submissions, so keep in mind that there will always be findings that are duplicates. For all issues labeled `3 (High Risk)` or `2 (Medium Risk)`, these have been pre-sorted for you so that there is only one primary issue open per unique finding. All duplicates have been labeled `duplicate`, linked to a primary issue, and closed.

Judges have the ultimate discretion in determining validity and severity of issues, as well as whether/how issues are considered duplicates. However, sponsor input is a significant criterion.

### Respond to issues

For each High or Medium risk finding that appears in the dropdown at the top of the chrome extension, please label as one of these:

- `sponsor confirmed`, meaning: "Yes, this is a problem and we intend to fix it."
- `sponsor disputed`, meaning either: "We cannot duplicate this issue" or "We disagree that this is an issue at all."
- `sponsor acknowledged`, meaning: "Yes, technically the issue is correct, but we are not going to resolve it for xyz reasons."

Add any necessary comments explaining your rationale for your evaluation of the issue. 

**Note that when the repo is public, after all issues are mitigated, wardens will read these comments; they may also be included in your C4 audit report.**

### Weigh in on severity 

If you believe a finding is technically correct but disagree with the listed severity, select the `disagree with severity` option, along with a comment indicating your reasoning for the judge to review. You may also add questions for the judge in the comments. (Note: even if you `disagree with severity`, please still choose one of the `sponsor confirmed` or `sponsor acknowledged` options as well.)

For a detailed breakdown of severity criteria and how to estimate risk, please refer to [the judging criteria in our documentation](https://docs.code4rena.com/roles/wardens/judging-criteria#estimating-risk-tl-dr).

## QA reports, Gas reports, and Analyses

All warden submissions in these three categories are submitted as bulk listings of issues and recommendations:

- **[QA reports](https://docs.code4rena.com/roles/wardens/judging-criteria#qa-reports-low-non-critical)** include *all* low severity and non-critical findings from an individual warden.
- **[Gas reports](https://docs.code4rena.com/roles/wardens/judging-criteria#gas-reports)** include *all* gas optimization recommendations from an individual warden.
- **[Analyses](https://docs.code4rena.com/roles/sponsors#analysis-pool)** contain high-level advice and review of the code: the "forest" to individual findings' "trees.”

**For QA reports, Gas reports, and Analyses, sponsors are not required to weigh in on severity or risk level.** We ask that sponsors: 

- Leave a comment for the judge on any reports you consider to be particularly high quality. (These reports will be awarded on a curve.)
- **For QA and Gas reports only:** add the `sponsor disputed` label to any reports that you think should be *completely* disregarded by the judge, i.e. the report contains no valid findings at all.

## Once labelling is complete

When you have finished labelling findings, drop the C4 team a note in your private Discord backroom channel and let us know you've completed the sponsor review process. At this point, we will pass the repo over to the judge to review your feedback while you work on mitigations.  

## Share your mitigation of findings

*Note: this section does not need to be completed in order to finalize judging. You can continue work on mitigations while the judge finalizes their decisions and even beyond that. Ultimately we won't publish the final audit report until you give us the OK.*

For each finding you have confirmed, you will want to mitigate the issue before the contest report is made public.

### If you are planning a Code4rena mitigation review:
1. In your own Github repo, create a branch based off of the commit you used for your Code4rena audit, then
2. Create a separate Pull Request for each **High or Medium risk** C4 audit finding (e.g. one PR for finding H-01, another for H-02, etc.)
3. Link the PR to the issue that it resolves within your contest findings repo.

Most C4 mitigation reviews focus exclusively on reviewing mitigations of High and Medium risk findings. Therefore, QA and Gas mitigations should be done in a separate branch. If you want your mitigation review to include QA or Gas-related PRs, please reach out to C4 staff and let’s chat!

If several findings are inextricably related (e.g. two potential exploits of the same underlying issue, etc.), you may create a single PR for the related findings.

### If you aren’t planning a mitigation review

1. Within a repo in your own GitHub organization, create a pull request for each finding.
2. Link the PR to the issue that it resolves within your contest findings repo.

This will allow for complete transparency in showing the work of mitigating the issues found in the contest. If the issue in question has duplicates, please link to your PR from the open/primary issue.
