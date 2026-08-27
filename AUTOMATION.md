# Automation: GitLab mirror sync + dev build

This repo is a scaffold, not a copy of `revanced-patches` itself. On first run,
the workflow below pulls in the actual source from the temporary GitLab
mirror and keeps it updated from then on.

Background: GitHub took down `ReVanced/revanced-patches` after a DMCA claim.
ReVanced filed a counter-notice and is developing from
https://gitlab.com/ReVanced/revanced-patches in the meantime — see
https://github.com/ReVanced/where-is-revanced-patches for their own status
page. This repo just automates (1) pulling that mirror's latest commits and
(2) running its existing Gradle build, on a schedule.

## What the workflow does, every 24h (or on manual trigger)

1. **`sync-mirror` job** — shallow-clones `gitlab.com/ReVanced/revanced-patches`,
   rsyncs its files into this repo (excluding `.github/`, so the automation
   never deletes itself), and pushes a commit if anything changed.
2. **`build` job** — checks out the freshly synced code, sets up JDK 17, and
   runs `./gradlew build`. It finds the resulting `.rvp`/`.jar` in `build/libs/`,
   uploads it as a workflow artifact, and publishes it as a dated **pre-release**
   (tag `dev-<run_number>-<sha>`) so you get a history instead of one file
   getting overwritten each day.

## One-time setup after you push this

1. **Enable write permissions for Actions**: repo → *Settings* → *Actions* →
   *General* → *Workflow permissions* → select **Read and write permissions**.
   Without this, the sync job can't push and the build job can't create releases.
2. Optionally trigger it once by hand: *Actions* tab → *Sync GitLab mirror & build
   dev patches* → *Run workflow*, rather than waiting for the next 3 AM UTC run.
3. **Keep it warm**: GitHub disables `schedule` triggers on a repo after ~60
   days with no activity. If you go quiet for two months, re-enable it (or
   push any commit) to restart the cron.

## If the build step fails

Upstream's Gradle setup can change over time (JDK version, module layout,
output filename). If `./gradlew build` or the artifact-locating step fails:

- Check `build.gradle.kts` in the synced source for the required JDK version
  and update `java-version` in the workflow.
- Check `build/libs/` in the failed run's logs to see what was actually
  produced, and adjust the `find` glob in the "Locate build output" step.

## Licensing note

`revanced-patches` is GPL-3.0-or-later. This scaffold just re-publishes and
rebuilds that same source under the same license — it doesn't relicense or
obscure provenance. The synced `LICENSE`/`README`/`CHANGELOG` from upstream
travel with every sync. If you redistribute the built artifact further,
GPLv3's attribution/source-availability terms still apply, same as they
would for any other GPL project.
