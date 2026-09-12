# Learning, Questionnaires and Recognition — Glossary

Every term of the domain, defined. Where a term has one name in the interface and another in the
stored identifiers, both are given, so a reader can move between the screens, the tables and the
integration payloads without guessing.

---

## A

**Accepted answer.** The one answer of a question that the asker, or a member with enough reputation
points, has marked as the answer that resolved the question. Only one answer per question may hold
the marker; accepting moves reputation points to the answer's author and to the acting user, unless
they are the same person.

**Access mode.** The setting of a questionnaire that decides who may start it: `public`, meaning
anybody holding the link, or `token`, meaning only somebody holding a personal participation token.

**Access token.** The public identifier that appears in every address of a questionnaire, and the
separate identifier that serves the same purpose on a course when a content file is fetched through
a share link. Unique per record and never copied on duplication.

**Answer option.** One proposed value of a choice question, one column of a matrix question, or one
row of a matrix question. Called Survey Answer Option in this folder and Survey Label in the
generated catalogues; stored as `survey.question.answer`.

**Answer row.** One stored answer, or one recorded skip, of one participation to one question.
Stored as `survey.user_input.line`. A choice question with three chosen options produces three rows;
a comment produces a fourth.

**Answer type.** The field of an answer row that says which value field carries the answer:
`text_box`, `char_box`, `numerical_box`, `scale`, `date`, `datetime` or `suggestion`. It is empty
exactly when the row is a skip.

**Archiving.** Hiding a record from the default queries without deleting it. On a forum post the
interface calls the operation deleting; on a course enrolment it is what leaving a course does.

**Assessment.** One of the five questionnaire purposes. Choosing it sets the access mode to `token`
and the scoring mode to `scoring_with_answers`.

**Attempt.** One participation of one person at one questionnaire. Attempts are counted per pool.

**Attempt pool.** The set of attempts that share one allowance: the same questionnaire, the same
contact or address, and the same invite token when one is set. A fresh invite token opens a fresh
pool, which is how a re-enrolled learner gets a new set of certification attempts.

**Attendee.** A contact enrolled in a course. The link between the two is a Course Enrolment.

**Attendee status.** The four-valued state of a Course Enrolment: `invited`, `joined`, `ongoing`,
`completed`.

## B

**Badge.** A distinction that can be granted by a person or by a challenge. Stored as
`gamification.badge`; a grant is stored as `gamification.badge.user`.

**Balance.** See *reputation points*.

**Batch mode.** A setting of a goal definition that lets one grouped query measure many users at
once, using a distinctive field to tell them apart and an expression that produces each user's own
value of that field.

## C

**Certification.** A scored questionnaire whose certification marker is true. Passing it produces a
printable certificate, an automatic message and, optionally, a badge. A certification may never be
run as a live session.

**Certification content.** A course content item of the category `certification` that points at a
certification questionnaire.

**Challenge.** A set of goals assigned to a population, with a periodicity, rewards and a report
schedule. Stored as `gamification.challenge`.

**Challenge line.** One goal definition of a challenge together with the value to reach. Stored as
`gamification.challenge.line`.

**Comment.** On a forum, a message on the discussion thread of a post; it is not a post. On a
course content item, a public message gated by the course's comment threshold.

**Completeness.** The percentage of a goal that has been achieved. Under the condition "the higher
the better" it is the current value over the target; under "the lower the better" it is only zero or
one hundred.

**Completion.** The percentage of a course an attendee has finished, computed as the completed
content items over the published, active, non-section content items of the course.

**Conditional question.** A question displayed only when at least one of its triggering answer
options has been chosen.

**Consolidation.** The monthly replacement of a past month's reputation movements by one movement
per user carrying the net change of that month. The sources and reasons of the replaced movements
are lost.

**Content item.** One entry of the ordered list of a course that is not a section. Called Course
Content in this folder and Slides in the generated catalogues; stored as `slide.slide`.

**Content progress.** The link between a contact and one content item, recording that it was opened,
whether it was completed, the vote cast and how many quiz attempts were made. Stored as
`slide.slide.partner`.

**Countdown.** One of the two time limits of a questionnaire: the whole-questionnaire limit in
minutes and the per-question limit in seconds used in a live session. Each has its own grace margin.

**Course.** A container of ordered content items with a visibility, an enrolment policy and reward
settings. Stored as `slide.channel`.

**Course type.** Either `training`, a guided path on which only the responsible may publish, or
`documentation`, a reference library on which the upload groups may publish too.

## D

**Deadline.** The instant after which a participation may no longer be opened or submitted. Written
by the invitation composer and carried over to a retry.

**Documentation course.** See *course type*.

**Downvote.** A vote of minus one on a forum post. It moves the forum's downvote grant, which is
negative by default, to the post's author.

## E

**Embed counter.** A record counting the loads of one content item embedded on one third-party
address. Stored as `slide.embed`. A load with no referring address is counted on the record whose
address is empty.

**Enrolment.** See *attendee* and *Course Enrolment*.

**Enrolment policy.** The setting of a course that decides how an attendee joins: `public`, `invite`
or `payment`.

## F

**Favourite.** A bookmark placed by a signed-in user on a forum question. Bookmarking also
subscribes the user to the question's thread; un-bookmarking leaves the subscription in place.

**Flagged.** The state of a forum post that a member has reported and that waits in the moderation
queue.

**Follow marker.** The no-follow attribute added to every link in a post written by an author below
the forum's follow threshold. The address itself is preserved.

**Forum.** A question-and-answer space with its own mode, privacy, ordering and reputation-point
table. Stored as `forum.forum`.

**Free preview.** A content item readable without enrolling in the course. A certification may never
be a free preview.

## G

**Goal.** One user's measurement against one definition over one period, with its own five-valued
state. Stored as `gamification.goal`.

**Goal definition.** What to measure, over which entity, with which filter, in which computation
mode, and whether higher or lower is better. Stored as `gamification.goal.definition`.

**Grace margin.** The tolerance beyond a countdown inside which a submit is still accepted: three
seconds for a question countdown, ten seconds for a questionnaire countdown.

## I

**Invite token.** The identifier that groups the participations of one attempt pool. Not unique; a
fresh one starts a new pool.

**Invited attendee.** A contact who received a course invitation and has not accepted it. The course
page is reachable, the contents are not, except the free previews.

## K

**Karma.** The stored name of the reputation-point balance held on a user account, and the prefix of
every field that grants or requires reputation points. This folder writes *reputation points* in
prose and uses the stored name only inside identifiers.

**Karma movement.** One recorded change of one user's balance, with its old value, its new value,
its source record and its reason. Stored as `gamification.karma.tracking`. The balance is derived
from these records, not stored independently.

**Karma rank.** A named tier unlocked at a minimum balance. Stored as `gamification.karma.rank`.

## L

**Last-activity instant.** The field of a forum question refreshed whenever it receives an answer or
a comment on itself or on one of its answers. It drives the Last Updated ordering.

**Lead-generating answer.** An answer option marked so that a participation choosing it produces a
lead. Added by the lead-generation package.

**Live session.** A questionnaire run in real time: attendees join by code, the host advances
question by question, results are revealed and a leaderboard ranks the attendees.

## M

**Matrix question.** A question whose answer is a grid: each row accepts one option in the `simple`
subtype and several in the `multiple` subtype. Each chosen cell is one answer row.

**Misplaced trigger.** A conditional question that has been moved before one of the questions whose
option triggers it. The stored trigger is kept, the question is flagged, and the question is dropped
from the screen list when every one of its triggers is invalid.

**Moderator.** A signed-in user whose balance reaches the forum's moderation threshold. Moderation
is not a group; it is a threshold.

## O

**Offensive.** The state of a forum post a moderator has judged offensive. The archiving marker is
set to false at the same time and the author loses the flagged amount.

**Own threshold.** The lower of the two thresholds of an action, applied when the acting user is the
author of the record concerned. For accepting an answer, "own" means the acting user asked the
parent question; for converting or deleting a comment, it means the acting user wrote the comment.

## P

**Participation.** One person's attempt at one questionnaire, with its tokens, its state, its timing
and its score. Called Survey Participation in this folder and Survey User Input in the generated
catalogues; stored as `survey.user_input`.

**Pending.** The state of a forum question written by an author below the publish threshold. It
waits in the validation queue and is hidden from everybody except moderators and its author.

**Prerequisite.** A course that must be completed before another. A course with no prerequisite
always reports its prerequisites as satisfied.

**Progress record.** See *content progress*.

**Purpose.** The setting of a questionnaire that names what it is for and applies a package of
defaults when it is chosen: `survey`, `live_session`, `assessment`, `custom` or `recruitment`.

## Q

**Question.** In the questionnaire engine, an entry of the flat ordered list that is not a section.
In the forum, a post with no parent.

**Quiz.** A set of questions attached to a course content item, answered on one screen, passed only
when every correct answer is chosen. Its questions are stored as `slide.question` and its answers as
`slide.answer`.

**Quiz attempt counter.** The number of times an attendee has submitted the quiz of a content item.
It decides which reward the next pass grants.

## R

**Rank.** See *karma rank*.

**Real-time reward.** A challenge setting under which the badge for every succeeding participant is
granted as soon as that participant has reached every goal, once per challenge, rather than at the
end.

**Relevance.** The stored ranking score of a forum post, combining its vote total with its age
through the two relevance parameters of the forum.

**Reputation points.** The community currency of the domain: a whole number held on a user account,
derived from the movement ledger, with no monetary meaning. Stored under the name `karma`.

**Responsible.** The user answerable for a questionnaire or for a course. On a training course the
responsible is the only person besides a Learning Manager who may publish.

**Restricted list.** The list of officers a questionnaire is reserved for. When it is non-empty,
only those officers and the administrators may read or write the questionnaire and everything under
it.

**Roaming.** The questionnaire setting that lets a participant move back to earlier screens and
leave mandatory questions unanswered until a final pass over the skipped ones.

## S

**Scale question.** A question whose answer is a whole number between a minimum of at least zero and
a maximum of at most ten, with three captions under the low, middle and high values.

**Scoring mode.** One of `no_scoring`, `scoring_with_answers_after_page`, `scoring_with_answers` and
`scoring_without_answers`. It decides whether scores are computed and when the correct answers are
revealed.

**Section.** In the questionnaire engine, an entry of the flat ordered list marked as a page; the
questions after it belong to it. In a course, an entry of the ordered list marked as a category; the
content items after it belong to it.

**Session code.** The short numeric code an attendee types to join a live session. Unique across
questionnaires and drawn by the code generator.

**Session state.** The three-valued state of a live session: empty, `ready` and `in_progress`.

**Speed bonus.** The adjustment that gives a correct answer in a live session more points the faster
it arrives, with a floor of half the raw score and a ceiling of the whole raw score.

**Subtype of a content item.** The precise kind derived from the category, the source and the
recognised service: `image`, `article`, `quiz`, `pdf`, `sheet`, `doc`, `slides`, `youtube_video`,
`google_drive_video`, `vimeo_video` or `certification`.

## T

**Tag family.** A named group of course tags. Filters combine the tags of one family with a logical
or and the families with a logical and. Stored as `slide.channel.tag.group`.

**Test entry.** A participation created by an officer to try a questionnaire, or the temporary one
used to preview a certificate. Test entries are excluded from every statistic and from the attempt
count, and they may open an archived questionnaire.

**Training course.** See *course type*.

**Trigger.** An answer option whose choice makes a conditional question appear.

## U

**Uncategorised.** The label under which questions placed before the first section, and content
items placed before the first section, are grouped.

**Upvote.** A vote of plus one on a forum post. It moves the forum's upvote grant to the post's
author.

## V

**Validation queue.** The moderation queue holding the forum posts in the state `pending`.

**Visibility.** The setting of a course that decides who may see its page: `public`, `connected`,
`members` or `link`.

**Vote.** On a forum, one member's judgement of one post, stored as `1`, `-1` or `0`; a cancelled
vote keeps its record with the value zero. On a course content item, the like or dislike stored on
the progress record as plus one, minus one or zero.

---

## Two vocabularies for the same records

The generated catalogues carry the names the system's own labels produce; this folder uses the
business names. The table reconciles them.

| Business name used here | Name in the generated catalogues | Transport name |
|---|---|---|
| Survey Answer Option | Survey Label | `survey.question.answer` |
| Survey Participation | Survey User Input | `survey.user_input` |
| Survey Participation Answer | Survey User Input Line | `survey.user_input.line` |
| Course Content | Slides | `slide.slide` |
| Course Enrolment | Channel / Partners (Members) | `slide.channel.partner` |
| Content Progress | Slide / Partner decorated many-to-many | `slide.slide.partner` |
| Quiz Question | Content Quiz Question | `slide.question` |
| Quiz Answer | Slide Question's Answer | `slide.answer` |
| Content Resource | Additional resource for a particular slide | `slide.slide.resource` |
| Content Embed Counter | Embedded Slides View Counter | `slide.embed` |
| Content Tag | Slide Tag | `slide.tag` |
| Course Tag | Channel/Course Tag | `slide.channel.tag` |
| Course Tag Family | Channel/Course Groups | `slide.channel.tag.group` |
| Course Invitation Wizard | Channel Invitation Wizard | `slide.channel.invite` |
| Post Closing Reason | Post Closing Reason | `forum.post.reason` |
| Post Vote | Post Vote | `forum.post.vote` |
| Badge | Gamification Badge | `gamification.badge` |
| User Badge | Gamification User Badge | `gamification.badge.user` |
| Badge Granting Wizard | Gamification User Badge Wizard | `gamification.badge.user.wizard` |
| Challenge | Gamification Challenge | `gamification.challenge` |
| Challenge Line | Gamification generic goal for challenge | `gamification.challenge.line` |
| Goal | Gamification Goal | `gamification.goal` |
| Goal Definition | Gamification Goal Definition | `gamification.goal.definition` |
| Goal Update Wizard | Gamification Goal Wizard | `gamification.goal.wizard` |
| Karma Rank | Rank based on karma | `gamification.karma.rank` |
| Karma Movement | Track Karma Changes | `gamification.karma.tracking` |

## Words this folder deliberately avoids

| Avoided | Used instead | Why |
|---|---|---|
| slide | content item | The record holds articles, documents, videos, quizzes and certifications, not only presentation slides. |
| channel | course | The record is a course everywhere in the interface. |
| member of a channel | attendee | To keep membership of an access group and enrolment in a course distinct. |
| user input | participation | The record is one person's attempt, not an input event. |
| label | answer option | The record is a proposed answer. |
| karma, in prose | reputation points | An acronym-free, self-explaining term; the stored name is kept inside identifiers. |
