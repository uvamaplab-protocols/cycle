# Requirements, round two

What the researcher asked for after trying release 0.0.10 (2026-10-30), with what the app and the
trails cloud already do for each, what changes where, and what is still to decide. "Protocol" is
this repo. "App" is digital-trails. "Server" is azure-iac, "portal" is web-portal2.

Settles follow-up questions 1 (the photo is wanted), 4 (the researcher sets the test days, not
the participant) and 10 (turn on `show_settings_config`). Question 5, when the first check-in
opens, is still open.

## 1. Choose the daily check-in time, not just 8 PM

**Already there.** A frequency survey's `time` takes a variable with a default, and the `Time`
flow element writes one: protocol-mindmove's config flow does exactly this with
`"variable_name": "global:random_time_evening"` and an `open`/`close` window. At enrollment the
config flow runs before the protocol loads, so the surveys are built with the chosen time. The
Settings Configuration row, when the code has `show_settings_config`, reruns the config flow and
reloads the protocol, so the time can be changed later.

**Protocol.** A config page with a `Time` element, `default_time` 20:00, writing
`global:checkin_time` (`output_name` `config_checkin_time`). In `surveys.json`,
`"time": "${global:checkin_time, 20:00}"`. The instructions flows read the time with
`has_variable` instead of saying 8:00 PM. Survey reminders, if wanted, go in a `reminders` section
and count from the slot's start, so they follow the chosen time without naming it.

**App.** Nothing. Two small things to know: a `Time` value prints as `20:00:00` when put into
text, and a reminder's own `time` field does not take a variable (the slot-relative form does not
need it).

**Study setup.** Turn on `show_settings_config` for the cycle codes.

**To decide.** The allowed window (e.g. 5 PM to 11 PM), and whether the 24 hour expiry stays.

## 2. The OPK survey: scheduled by the researcher, with 3 or 4 reminders that day

Today the participant enters the test days under My Test Days (a `Scheduler` element), which
gives one notification per day and no survey: logging a result is a home tile, not a slot in Show
Surveys. The ask is the reverse: the researcher sets the days, the app opens a survey on each and
reminds the participant several times.

**Already there.**

- Variables on an invite, server side. A direct invite (`GET /invite?code=…&email=…`) already
  takes every extra query parameter as a study variable (`Invite.GetExtra`; mtm sends
  `population` this way from Qualtrics), stores it on the invitation record and seeds it into the
  participant's enrollment document at redemption (`OnboardingEnrolledPost`). Redemption seeds
  `variables` from any invitation kind, so a portal-made invitation only lacks a way to put them
  on the record: `POST /invitations` and the Invitations form take label, roles, expiry and
  linking code, nothing generic.
- In the app those variables are the `global:` variables. Survey fields (`time`, `cron`, `span`,
  `expiry`) already take `${global:…}`.
- Reminders. A `reminders` entry (`survey_id`, `frequency`, `take`, `skip`) fires `take`
  notifications after a slot opens, so "3 or 4 on that day" is `"frequency": "03:00:00",
  "take": 3` once the OPK log is a survey.

**App.** A survey type that opens on a list of dates, say `"type": "dates"` with `dates` (the
variable, comma-separated), `time`, `expiry`, `path`, `name`, implementing `Ascending` and
`Descending` per `ISurvey`. One slot per date; Show Surveys, the open-survey notification and the
reminders then come with it. Also: the protocol's surveys are built when it loads, and the study
document re-syncs on resume but nothing reloads the protocol when its variables change (overrides
do trigger a reload). A test day set after enrollment needs that reload, or it is not seen until
the next launch.

**Server and portal.** Generic `variables` (name and value pairs) on `POST /invitations` and on
the Invitations form, carried on the record and seeded at redemption like roles. The test days
depend on the participant's cycle, so they may not be known when the invite is made: a
per-participant variables editor on the participant page (writing the enrollment document's
variables) covers that, and would also let staff set group and contraception type (6) instead
of asking. One of the two is needed; both is best.

**Protocol.** An `opk` survey in `surveys.json` reading `${global:opk_days}` with a time such as
`${global:opk_time, 06:00}`, pointing at the OPK log flow; a `reminders` entry for it; the
`Scheduler` page and the My Test Days tile removed (5).

**To decide.** The variable format (`opk_days=2026-11-03,2026-11-05`), the slot's open time and
expiry, and whether staff set the days at invite time or on the participant page.

## 3. A picture of the test strip when the result is positive

**New in the app.** No flow element takes a photo. Messaging already captures and uploads images
(`AttachmentService` builds an attachment from a picked file and uploads it to blob storage), so
a `Photo` element can reuse that path: take a picture or choose one, preview, remove, and record
the blob path as the element's value, which lands in the Flow datum with the rest of the answers.
The portal then needs to show or download that image on the participant's data.

**Protocol.** Give the OPK result buttons `"variable_name": "opk_result"` and add a second page
with `"condition": ["opk_result", "=", "positive"]` holding the `Photo` element
(`output_name` `opk_photo`).

**To decide.** Required or optional on a positive, and who views the photos and for how long.

## 4. Make the symptom stand out on the severity screens

**Protocol only.** `Text` takes `font_size`, `font_attributes` (`Bold`) and `text_alignment`, and
a page has `header_text`. Pages 4 to 15 of the daily survey become: the symptom name first, large
and bold (for example `"font_size": 28, "font_attributes": "Bold", "text_alignment": "Center"`),
then "How severe is this symptom today?" at normal size, then the 0 to 6 buttons. Optionally the
symptom in `header_text` too.

## 5. Clean up the home screen for the naturally menstruating group

Follows from 2. With the test days set by the researcher and the result a scheduled survey, the
Ovulation Tests tiles lose Log a Test Result and My Test Days, and `flows/opk/schedule.json` goes.
How to Use (the video) stays, moved under More information, so the NM home is the HC home plus
the video. `flows/instructions/nm.json` changes to: the study team has set your test days, the app
opens the test survey and reminds you on each.

**To decide.** Whether an off-schedule Log a Test Result should stay for a test taken on another
day. Suggest no, so every result is on a researcher-set day.

## 6. Contraception type, once, for the hormonal contraception group

**Already there.** The config flow runs once at enrollment and a page takes a `condition`. A page
has one condition, but no AND is needed: only HC participants see the question, so
`global:contraception_type` is unset for NM, and an unset variable fails every condition.

**Protocol.** The config flow becomes a folder, since one file is one page: `flows/config/1.json`
the population question (as now), `2.json` with `"condition": ["global:population", "=", "HC"]`
asking "What type of contraception are you on?" with `IUD` and `Oral` buttons,
`"variable_name": "global:contraception_type"`, `output_name` `config_contraception_type`, and
`3.json` the check-in time (1). The daily survey's page 2 condition becomes
`["global:contraception_type", "=", "Oral"]`, worded "Did you take your birth control pill today
as prescribed?".

**Study setup.** The code document's `config` changes from `flow://flows/configs.json` to
`flow://flows/config`.

**Or the researcher sets it.** With the invite variables or the participant-page editor from 2,
`contraception_type` (and `population`) can be set by staff and the config pages dropped or kept
as a fallback with a condition on the variable being unset.

**To decide.** The option list: the population page says "pill, shot, or ring", and shot, ring,
patch and implant are neither IUD nor Oral. Only Oral gets the nightly question either way.

## By owner

| Where | Work |
| --- | --- |
| Protocol | 1 (config page, `time` variable, instructions), 4, 5, 6, and the `opk` survey, reminders and photo page from 2 and 3 once the app has them |
| App | `dates` survey type and a protocol reload when study variables change (2); `Photo` element (3) |
| Server and portal | `variables` on portal invitations and/or a per-participant variables editor (2, 6); photo display (3) |
| Study setup | `show_settings_config` on (1); `config` path to the folder (6) |
