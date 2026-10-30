# Follow-up questions

Open items after building the protocol from the researcher's three Word documents and testing
release 0.0.10 on a phone with the `cycle-t` code (2026-10-29).

## For the researcher

1. **Photo of the ovulation test strip.** The study summary asks participants to attach a photo of
   the strip, but survey pages in the app can't take photos. The newer "Additional Questions" doc
   asks only for positive or negative, which is what the app records now. Is the photo still
   needed? If so, participants could send it through study messaging, or the app would need a
   photo question added.
2. **Contraception question wording.** The app asks "Did you take/use your hormonal contraception
   today, per your regular schedule?" (Yes / No / I don't know), from the study summary. The
   "Additional Questions" doc words it "Did you take your birth control today as prescribed?".
   Which should it be? The current wording covers the shot and ring.
3. **"None of these" symptom option.** The symptom list has a "None of these" choice so participants
   with no symptoms can answer. The researcher's list didn't include one. Is that OK?
4. **Who enters the ovulation test days?** The researcher picks up to 5 days per participant. The app
   reminds the participant on each day entered under My Test Days, so either the participant or
   staff at enrollment has to enter them.
5. **When the daily check-in starts.** It opens at 8 PM and stays open until 8 PM the next day, for
   30 days. The first one opens the second evening after enrollment: someone enrolling on Oct 29
   at 11 PM first sees it on Oct 31 at 8 PM. Should it start the evening of enrollment instead?
   That's `skip: 0` in `src/surveys.json`.
6. **Team and contact information.** The app has no team page, FAQ or contact email. Does the
   researcher want any of these, and who should participants contact?
7. **App name.** The app shows "Cycle". The summary proposed "CycleTrail" and asked for other ideas.
8. **"Both apps."** The questions doc is titled "Additional Questions to add to both apps". Is there a
   second app or protocol that needs the same daily questions?

## For study setup

9. **Study display name.** `cycle-t` has no display name. The enrollment prompt reads
   "Enrollment in requires downloading…" with a blank where the name goes, and Settings shows
   nothing before "(cycle-t · test)".
10. **Changing a participant's group.** Settings has no Configuration row for `cycle-t`, so the only
    way to switch someone between groups is to unenroll and re-enroll. Turning on
    `show_settings_config` for the study would let staff change it in place.
11. **Reminder timing (app, not protocol).** Android delivers the app's reminders inexactly: daily
    check-in reminders can arrive any time from 8 to 9 PM, and a test-day reminder came about
    3 minutes late. Every Digital Trails notification also uses the same notification id, so a
    newer reminder replaces an older one in the shade.
12. **Portal check.** Test data was sent from `cycle-t` on Oct 29 between about 11:11 and 11:37 PM,
    for both groups. Confirm it shows in the portal with the expected output names (`daily_*`,
    `opk_result`, `opk_test_days`, `config_population`).
