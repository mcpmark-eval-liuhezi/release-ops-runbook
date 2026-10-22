# Release Operations Runbook

This is the platform team's operational runbook for shipping releases. It covers the
standard release process, the rollback procedure, and who to contact when things go
wrong. It is the single source of truth for release day, so keep it updated whenever
the process changes.

## Release Process

1. **Freeze the release scope.** The release manager opens a tracking issue listing
   the stories, bug fixes, and infrastructure changes going out. Anything not on
   the list slips to the next release.
2. **Cut the release branch.** From the default branch, cut `release/v<date>` and
   apply only cherry-picks from the agreed scope. No new work lands on the release
   branch.
3. **Run the verification suite.** CI must be green on the release branch: unit
   tests, integration tests, lint, and the deployment dry-run against staging.
4. **Stage and smoke test.** Deploy the release candidate to staging and run the
   smoke checklist (sign-in, core flows, monitoring dashboards green, no new
   error-rate alerts).
5. **Review and sign-off.** A reviewer other than the author verifies the change
   list against the tracking issue and leaves an approving review. Never approve
   your own change.
6. **Deploy to production.** Roll out during the agreed low-traffic window, one
   component at a time, watching dashboards between steps.
7. **Tag and document.** Tag the release, update CHANGELOG.md, and post the release
   notes. The release manager confirms completion in the tracking issue.

## Rollback

If production is unhealthy after a deployment, do not debug forward — roll back first.

1. **Declare the rollback.** Anyone on the team can call it; the release manager
   confirms and starts the clock.
2. **Restore the previous state.** Re-deploy the last known-good release, revert the
   data migrations in reverse order, and restore any configuration changes made
   during the rollout.
3. **Verify recovery.** Re-run the smoke checklist and confirm dashboards and alerts
   return to baseline before closing the incident.
4. **Record the lesson.** Within 24 hours, write a short postmortem: what broke, how
   it was detected, what was rolled back, and what check would have caught it.

## Emergency Contacts

- **Release manager:** first escalation point during a rollout window; owns go/no-go
  and rollback decisions.
- **Platform on-call:** reachable via the team pager; first responder for
  infrastructure and deployment failures.
- **Service owners:** each component in the rollout has a named owner who can be
  pulled in for domain-specific triage.
- **Escalation manager:** if the primary contacts are unavailable, the escalation
  manager takes over coordination and communications.
