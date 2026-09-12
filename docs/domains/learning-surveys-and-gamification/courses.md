# The learning platform

Everything a replacement needs in order to reproduce the behaviour of a course: its two types, its
visibility and enrolment matrices, the content categories and their sources, the ordering and
sectioning rules, the publication rules, the progress and completion machinery, quizzes and their
rewards, certifications, resources, embedding, selling, and the attendee-facing pages.

## 1. Course types

| Type | Intent | Who may publish content | Reviews and likes by default |
|---|---|---|---|
| `training` | A guided path taken from beginning to end. | Only the responsible, and the Learning Managers. | Allowed. |
| `documentation` | A reference library consulted in any order. | The responsible, the Learning Managers, and any user belonging to one of the course's upload groups. | Not allowed. |

The difference is expressed through two derived permissions:

```formula
may upload  = the reader is the responsible
              OR ( upload groups are set AND the reader belongs to one of them )
              OR the reader is a Learning Manager

may publish = may upload
              AND ( the reader is the responsible OR the reader is a Learning Manager )
```

On a training course the upload groups are normally empty, so only the responsible and the managers
can upload and the two permissions coincide. On a documentation course the upload groups let other
people upload; those people can upload but not publish, which means their content is created
unpublished with no publication instant. An invited attendee previewing a course may never publish,
even when the preview runs with elevated rights.

The refusal for any publishing attempt is [`LSG-079`](business-rules.md#lsg-079).

The type also drives the default of the comment marker — false for `documentation`, true otherwise.
When comments are not allowed, attendees may neither like, dislike, comment nor review. The type
finally drives the placeholder illustration used when the course has no picture, and the extra
panels of the content page: on a documentation course the content page shows the most viewed
contents of the course and the other contents of the same section; on a training course it shows
neither.

## 2. Visibility and enrolment

Two independent settings decide who sees a course and who may join it.

### 2.1 Visibility

| Value | Who may see the course page |
|---|---|
| `public` | Everybody, including anonymous visitors. |
| `connected` | Signed-in visitors, and enrolled attendees. |
| `members` | Only enrolled attendees and invited contacts. |
| `link` | Anybody holding the direct address; the course is not listed. |

The derived visible marker is true when the visibility is `public`, or the reader is enrolled, or
the reader is signed in and the visibility is `connected`. It is what the public course list filters
on, which is why a `link` course never appears in the list even though its page is reachable.

### 2.2 Enrolment policy

| Value | How an attendee joins |
|---|---|
| `public` | Any signed-in visitor presses join and is enrolled at once. |
| `invite` | An officer adds or invites the attendee, or the visitor requests access and the responsible grants it. |
| `payment` | The visitor buys the linked product; confirming the order enrols them. Added by the course-selling package. |

### 2.3 Coupling between the two

- A course whose visibility is `members` is forced to the enrolment policy `invite`;
  [`LSG-061`](business-rules.md#lsg-061) refuses any other combination.
- A paid course must carry a product; [`LSG-062`](business-rules.md#lsg-062).
- When the course-selling package is removed, every course whose policy was `payment` falls back to
  `invite`.
- Duplicating a course whose visibility is `members` forces the copy's policy to `invite` unless the
  caller supplies one.

### 2.4 Automatic enrolment by access group

A course may list access groups; every member of those groups is enrolled automatically. The
enrolment is refreshed at five moments: when the course is created, when the group list is written,
when a user is created, when a user's group list is written — looking at the newly linked groups and
every group they imply — and when a group's user list is written.

### 2.5 Requesting access

The five outcomes are [`LSG-067`](business-rules.md#lsg-067) to
[`LSG-072`](business-rules.md#lsg-072); the procedure is
[`workflows.md`](workflows.md#workflow-9), path D.

### 2.6 Who may add attendees

The add-members operation first filters the courses: open-enrolment courses always pass; for the
others the acting user must have write access. When the caller asked for the strict check and a
course was filtered out, the operation is refused by
[`LSG-066`](business-rules.md#lsg-066).

## 3. Attendee statuses

| Status | Meaning | Course page | Contents |
|---|---|---|---|
| `invited` | An invitation was sent and not accepted. | Reachable, even for a course whose visibility is `members`. | Not reachable, except the free previews. |
| `joined` | Enrolled, nothing completed yet. | Reachable. | Reachable. |
| `ongoing` | Enrolled, completion strictly between zero and one hundred. | Reachable. | Reachable. |
| `completed` | Completion reached one hundred. | Reachable. | Reachable. |

The rules that govern the status are in
[`state-machines.md`](state-machines.md#machine-4). The three counters shown to the officer are:
invited, engaged — `joined` plus `ongoing` — and completed. The enrolled count is engaged plus
completed; the total count is invited plus engaged plus completed.

## 4. Content: categories, subtypes and sources

### 4.1 Categories

| Category | Payload | How it is completed |
|---|---|---|
| `article` | An authored rich-text body stored on the content item. | Marked completed by the attendee, or automatically when opened with no quiz attached. |
| `infographic` | A picture, uploaded or retrieved from the external document storage service. | The same. |
| `document` | A document, uploaded or retrieved from the external document storage service. | The same. |
| `video` | A video hosted on the video-sharing service, on the external document storage service, or on the video-hosting platform. | The same. |
| `quiz` | No payload; only the attached questions. | By answering every question correctly. |
| `certification` | A questionnaire from the questionnaire engine. | By passing the questionnaire. |

### 4.2 Subtypes

The precise subtype is derived from the category, the source and, for a video, the recognised
service:

| Category | Source | Subtype |
|---|---|---|
| `document` | local file | `pdf` |
| `document` | external | kept when it is already `pdf`, `sheet`, `doc` or `slides`; otherwise cleared and filled from the retrieved metadata |
| `infographic` | any | `image` |
| `article` | any | `article` |
| `quiz` | any | `quiz` |
| `certification` | any | `certification` |
| `video` | an address recognised as the video-sharing service | `youtube_video` |
| `video` | an address recognised as the external document storage service | `google_drive_video` |
| `video` | an address recognised as the video-hosting platform | `vimeo_video` |
| anything else | any | cleared |

The icon beside the content follows the subtype, as listed in section 8.2 of
[`entities.md`](entities.md).

### 4.3 Sources and address recognition

- **Local file.** The payload is uploaded and kept as an attachment. Two field views expose the same
  payload through a picture-only picker and through a document-only picker, so the form can restrict
  what may be dropped.
- **External.** The payload stays on an external service and only the address is stored. Three
  address patterns are recognised: a video-sharing address in any of its forms — short link, embed
  link, watch link with the key as a parameter, short-form link, live link — from which an
  eleven-character key is extracted; an external document storage address of the form `.../d/<key>`,
  from which the key is extracted; and a video-hosting platform address, from which a six to eleven
  digit key is extracted, optionally followed by a private hash.

A content item may never hold both an authored body and an external address;
[`LSG-076`](business-rules.md#lsg-076) refuses it, and the write operation enforces it by clearing
the other field whenever the category is switched to or away from `article`.

### 4.4 Metadata retrieval

When an address is written, the matching external service is called and the fields still empty are
filled from what it returns: the title, the description, the illustration, the duration in hours
and, for the external document storage service, the precise subtype derived from the media type.
Retrieval is skipped while a package is being installed, and skipped when the caller explicitly asks
for it. On the form the same retrieval runs as an on-change so the author sees the values before
saving; it never overwrites a field the author already filled. The key used to reach the external
document storage service is a per-site setting.

### 4.5 Duration

Durations are decimal hours. A content duration carries four decimal places, a course duration two.
The three sources of a duration are: the five-minutes-per-page estimate of
[`LSG-CALC-014`](calculations.md#lsg-calc-014) for an uploaded portable document; the declared
running time for an external video or document; and the recursive sum of
[`LSG-CALC-013`](calculations.md#lsg-calc-013) for a section and for a course.

## 5. Ordering and sections

The contents of a course form one flat, ordered list, sorted by sequence ascending, then by the
section marker ascending, then by identifier ascending; at an equal sequence, content items
therefore come before sections.

A content item belongs to the last section that appears before it in that order. Items appearing
before the first section belong to no section and are shown under the heading `Uncategorized`,
always first.

Two helper operations keep the order consistent:

- **Moving a section.** Removing a section from the list first removes its content items from the
  ordered list of identifiers, then re-inserts them at the position of the target section — or at
  the very front when there is no target — and finally rewrites every sequence as its new index plus
  one.
- **Placing a newly added content item.** When the course has sections, the new item is inserted
  just before the section that follows its own section, or before the first section when it has
  none, and every sequence is rewritten as its index plus one, starting at one. When the course has
  no section, the new item simply receives the highest sequence plus one.

Deleting a section moves its content items to the front of the course before the section record is
removed. Duplicating a single content item forces its sequence to zero, which makes the copy land
first and uncategorised; duplicating a whole course keeps the sequences.

## 6. Publication and preview

- A section is always created preview-enabled and published; writing the section marker forces both.
- A content item created by somebody who may not publish has its publication instant cleared, and
  the publication marker is left to the website behaviour, which leaves it unpublished.
- A content item created already published receives the current instant as its publication instant,
  and each later setting of the publication marker rewrites that instant.
- Publishing a non-section content item posts the new-content message on the course, addressed to
  the followers of the new-content subtype, using the course's new-content template. When the
  template defines a reply address, that address is used.
- Archiving a published non-section content item unpublishes it.
- A content item is new when it is published and its publication instant is within the last seven
  days.
- A **free preview** content item is readable without enrolling. A certification may never be a
  preview; [`LSG-078`](business-rules.md#lsg-078).

The new-content marker on a course is true when at least one non-section content item published in
the last seven days has not yet been completed by the reader; it is what shows the new-content
badge on the attendee's course card.

## 7. Completion

### 7.1 Per content item

A content item is completed for an attendee when the progress record of that pair carries the
completion marker. The five ways it becomes true and the two ways it becomes false are machine 7 of
[`state-machines.md`](state-machines.md#machine-7).

### 7.2 Per course

The completion of an enrolment is recomputed whenever a progress record changes, whenever a content
item is created published, published, unpublished, archived, un-archived or deleted, and whenever an
invited or archived attendee is enrolled. The arithmetic is
[`LSG-CALC-011`](calculations.md#lsg-calc-011).

Two transitions carry side effects, and both are evaluated **before** the new status is written, by
comparing the previous completion:

| Transition | Condition | Side effect |
|---|---|---|
| Reaching the end | The previous completion was not one hundred, the course is active, the course holds at least one content item, and the new completed count is at least the total. | The course-completion reward is granted with the reason `Course Finished`, the completion message is sent, and a résumé line of the Training type is written when the skills package is installed. |
| Falling back | The previous completion was one hundred, the course is active, and the new completed count is below the total. | The course-completion reward is taken back with the reason `Course Set Uncompleted`. |

Because archived courses are excluded from both, archiving a course never sends a completion message
and never moves reputation points.

### 7.3 The next content item

The next content of an enrolment is the first published, active, non-section content item of the
course, ordered by sequence then identifier, for which the attendee has no completed progress
record. It is empty when everything is done, and it is what the continue button points at.

A related helper decides which section to open next after a content item is completed: when the
completed item was uncategorised and every uncategorised item is now completed, the first section is
returned; when the completed item's section is now entirely completed and it is not the last
section, the following section is returned; otherwise nothing.

The course page collapses a section when it holds no section at all, when it holds the current
content item, when it is the section just designated as the next one to open, or when it is ongoing,
that is, when some but not all of its content items are completed.

## 8. Quizzes and their rewards

A quiz is a set of questions attached to one content item; each question has a set of answers, some
marked correct.

**Integrity.** Every quiz question must have at least one correct answer **and** at least one
incorrect answer; [`LSG-085`](business-rules.md#lsg-085).

**Taking a quiz.** The procedure is [`workflows.md`](workflows.md#workflow-11); the reward ladder is
[`LSG-CALC-012`](calculations.md#lsg-calc-012).

**Rewards for finishing a course.** Completing a course grants the course-completion reward, ten by
default, with the reason `Course Finished`; falling back below one hundred percent takes the same
amount back with the reason `Course Set Uncompleted`. A configured value of zero or less grants
nothing.

**Rewards for a review.** Posting a message that carries a rating on the course grants the review
reward, five by default, with the reason `Course Ranked`, but only when the author is the reader
themselves. Posting a review requires the review threshold, ten by default; the two refusals are
[`LSG-075`](business-rules.md#lsg-075).

**Rewards earned after the fact.** An officer may ask how many points a set of attendees earned on a
set of courses; the arithmetic and its caveat are
[`LSG-CALC-016`](calculations.md#lsg-calc-016).

## 9. Likes, dislikes and comments

- Liking and disliking are stored as the vote of the progress record: plus one, minus one or zero.
- Pressing like sets the vote to one, or back to zero when it was already one. Pressing dislike sets
  it to minus one, or back to zero when it was already minus one. A progress record is created when
  none exists.
- The like endpoint applies the four refusals of
  [`LSG-090b`](business-rules.md#lsg-090b).
- The like and dislike counters of a content item are the counts of progress records whose vote is
  plus one and minus one; the vote total of a course is the sum of the likes minus the sum of the
  dislikes of its published, active, non-section content items.
- Commenting a content item requires the course's comment threshold;
  [`LSG-080`](business-rules.md#lsg-080). A message of any other kind — a notification, a system
  note — is not gated.
- The comment count of a content item is the number of public messages on it.

## 10. Certifications inside a course

A content item of the `certification` category points at a questionnaire.
[`LSG-077`](business-rules.md#lsg-077) guarantees the link and
[`LSG-078`](business-rules.md#lsg-078) guarantees it is never a free preview. When a questionnaire is
linked at creation, the category is forced to `certification`; when the content title is still empty
it is filled with the questionnaire title.

The whole cycle — starting an attempt, resuming one, passing, failing the last attempt, and the
badge category — is [`workflows.md`](workflows.md#workflow-12). A questionnaire used as a course
certification may not be deleted; [`LSG-090`](business-rules.md#lsg-090).

## 11. Resources

A content item may carry additional resources, each either an uploaded file or an external link. The
two are mutually exclusive; [`LSG-089`](business-rules.md#lsg-089).

Switching the type clears the other field. The name defaults to the literal `Resource` and is
recomputed only while it still holds that default or is empty: for a file it becomes the stored file
name; for a link it becomes the link. Once the author types a name of their own, the recomputation
stops touching it.

The download address appends the extension of the original file name to the chosen name when the
chosen name does not already end with it, and carries a download marker.

Resources are readable only by the attendees of the course, by the Learning Officers and by the
Learning Managers; they are hidden entirely from anonymous visitors, and the quiz panel lists them
only when the reader is enrolled. The separate download marker on the content item decides whether
the payload of the content item itself may be downloaded.

## 12. Embedding on third-party sites

Every content item exposes two embed markups: one for a page of this platform and one for an
external site. Loading the external markup increments a counter:

1. The host part of the referring address is examined. When the request carries no referring
   address, or the address has no host part, the counter of the record whose address is empty is
   used, so anonymous embeds are still counted.
2. When a counter record already exists for that content item and that address, its view count is
   incremented by one.
3. Otherwise a counter record is created with a view count of one.

The embed count of a content item is the sum of the view counts of its counter records.

## 13. View counters

| Counter | Incremented when |
|---|---|
| anonymous views | An anonymous or non-enrolled visitor opens a content item they have not opened before in this browser session. The identifiers already counted are held in the browser session, so a refresh does not inflate the figure. Both the anonymous counter and the total counter are incremented at once, without taking a lock. |
| signed-in views | Derived: the number of progress records on the content item, that is, the number of signed-in people who opened it. |
| total views | Derived: signed-in views plus anonymous views. |

An enrolled attendee opening a content item does not touch the anonymous counter; the mark-viewed
operation creates or refreshes their progress record instead, optionally incrementing the quiz
attempt count.

## 14. Invitations and the invitation link

Two actions open the same composer with a different template and a different mode:

| Action | Mode | Template | Resulting status |
|---|---|---|---|
| Add attendees | enrolment | `Elearning: Add Attendees to Course` | `joined` |
| Invite | invitation | `Elearning: Promotional Course Invitation` | `invited` |

The composer's send marker defaults to true unless the course is public and the mode is invitation.
When the marker is false the operation does nothing at all. The sending procedure is
[`workflows.md`](workflows.md#workflow-9), paths A and B.

<a id="invitation-link"></a>

### 14.1 The personal invitation link

Each enrolment exposes a personal address of the shape: the site address, then
`/slides/<course identifier>/invite`, then two query parameters carrying the contact identifier and
a digest. The digest is a keyed message digest over the pair of contact identifier and course
identifier, using a stored platform secret and the label `website_slides-channel-invite`. It is
compared in constant time.

The six outcomes of opening that address are listed in
[`workflows.md`](workflows.md#workflow-9). The invitation of an attendee still in the `invited`
status expires three months after the last invitation instant; a missing instant counts as expired.

### 14.2 Expiry of invitations

The periodic cleanup deletes every enrolment whose status is `invited`, whose completion is zero,
and whose last invitation instant is either empty or older than three months.

## 15. Sharing

Two sharing operations exist, one for a whole course and one for a single content item. Both require
a template on the course; the two refusals are
[`LSG-074`](business-rules.md#lsg-074) and [`LSG-090c`](business-rules.md#lsg-090c). When the sender
is an external portal user, the message is sent with elevated rights and from the company's
catch-all address, so an external user cannot forge a sender.

## 16. Selling course access

When the course-selling package is installed:

- A course may carry the enrolment policy `payment` and must then point at a product whose service
  tracking is `course`. When exactly one such product exists in the catalogue it becomes the
  default.
- The sales description of that product becomes `Access to: ` followed by the course names, on one
  line for a single course and on separate lines for several.
- Such a product may always be added to the cart when at least one published course uses it;
  [`LSG-131`](business-rules.md#lsg-131).
- Only one unit of a course line may be bought; [`LSG-130`](business-rules.md#lsg-130).
- Confirming an order enrols the customer into every paid course whose product appears on a line.
- Publication is kept in step in both directions; [`LSG-132`](business-rules.md#lsg-132).
- The revenue of a course is the total of the confirmed sales-analysis amounts of its product, shown
  in the product currency and visible only to the salesperson group.

The ledger consequences of the sale belong entirely to the selling and receivable domains; see
[`accounting-effects.md`](accounting-effects.md).

## 17. Prerequisites

A course may list prerequisite courses. The picker offers only other courses with the same
visibility and the same publication state. The completed-prerequisites marker is true when the
reader holds a `completed` enrolment on **every** prerequisite, so a course with no prerequisite
always reports true.

## 18. Tags and tag families

- A content tag is a free, globally unique label; [`LSG-090a`](business-rules.md#lsg-090a).
- A course tag belongs to a family and carries a colour. A tag whose colour is empty or zero is an
  internal tag and is never displayed on the public pages.
- Filtering courses by tags combines the tags of the same family with a logical or, and the families
  with a logical and. Choosing two tags of the family `Level` shows the courses carrying either of
  them; choosing one tag of `Level` and one of `Tags` shows only the courses carrying both.
- Only published families appear in the public filter panel; new families are published by default.
- The public search on courses also filters on the course visibility, on membership when the visitor
  asked for their own courses, and on a chosen content category, keeping only courses that hold at
  least one content item of that category.

## 19. The attendee-facing pages

### 19.1 Course list

The list shows the courses visible to the reader. Each card shows the picture, the name, the short
description, the tags that carry a colour, the duration, the attendee count, the rating average,
and, for an enrolled attendee, the completion and the new-content marker.

### 19.2 Course page

The page shows the cover block, the title, the description, the rating summary, the attendee actions
— join, request access, buy, leave — the featured content when one is designated, the ordered list
of contents grouped by section with the completion state of each, the reviews, and, for a publisher,
the upload and configuration actions.

The featured content follows the promotion strategy: the most recently published, the most liked,
the most viewed, a manually chosen content item, or none.

The base filter used to fetch the contents of a course page is: the contents of that course that are
not sections, restricted to the current site, plus one of three refinements — a publisher sees
everything; a signed-in visitor sees the published contents and the contents they uploaded
themselves; an anonymous visitor sees only the published contents.

### 19.3 Content page

The content page shows the payload — the player for a video, the viewer for a document, the body for
an article, the quiz panel for a quiz, the certification button for a certification — the
description, the resources, the previous and next content items, the completion actions, the like
and dislike buttons, the comment thread, and the ordered list of contents of the course as a side
panel, with the sections collapsed according to the rule of section 7.3.

The quiz panel is served as data rather than as markup, and it hides what the attendee must not see:
the correctness of an answer is disclosed only when the attendee has already completed the content
or is a site designer, and the per-answer comment only to a site designer.

## 20. Leaving a course and merging contacts

Leaving is [`workflows.md`](workflows.md#workflow-14). Merging two contacts that are both enrolled
in the same course is refused by [`LSG-129`](business-rules.md#lsg-129).
