This page details all the endpoints of the Autolab REST API. 

The client's access token should be included as a parameter to all endpoints. For details on obtaining access tokens, please see the [API Overview](/api-overview)

## Format

All endpoints expect the HTTP GET method unless otherwise specified.

All parameters listed below are required unless denoted [OPTIONAL].

All responses are in JSON format.

* If the request is completed successfully, the HTTP response code will be 200. The reference below details the keys and their respective value types that the client can expect from each endpoint.
* If an error occurs, the response code will *not* be 200. The returned JSON will be an object with the key 'error'. Its value will be a string that explains the error.

**Notes on return value types**

All datetime formats are strings in the form of 'YYYY-MM-DDThh:mm:ss.sTZD'.<br>
e.g. '2017-10-23T04:17:41.000-04:00', which means 4:17:41 AM on October 23rd, 2017 US Eastern Time.

JSON spec only has a 'number' type, but the spec below distinguishes between integers and floats for ease of use in certain languages.

If a field does not exist, the value is generally null. Please be sure to check if a value is null before using it.

## Interface

---
### user

Get basic user info.

**Scope:** 'user_info'

**Endpoint:** `/user`

**Parameters:** [none]

**Responses:**

| key | type | description |
| --- | ---- | ----------- |
| first_name | string | The user's first name. |
| last_name | string | The user's last name. |
| email | string | The user's registered email. |
| school | string | The school the user belongs to. |
| major | string | The user's major of study. |
| year | string | The user's year. |

---
### courses

Get all courses currently taking or taken before.

**Scope:** 'user_courses'

**Endpoint:** `/courses`

**Parameters:**

* state<br>
  [OPTIONAL] filter the courses by the state of the course. Should be one of 'disabled', 'completed', 'current', or 'upcoming'. If no state is provided, all courses are returned.

**Responses:**

A list of courses. Each course contains:

| key | type | description |
| --- | ---- | ----------- |
| name | string | The unique url-safe name. |
| display_name | string | The full name of the course. |
| semester | string | The semester this course is being offered. |
| late_slack | integer | The number of seconds after a deadline that the server will still accept a submission and not count it as late. |
| grace_days | integer | AKA late days. The total number of days (over the entire semester) a student is allowed to submit an assessment late. |
| auth_level | string | The user's level of access for this course. One of 'student', 'course_assistant', 'instructor', of 'administrator'. |

---
### assessments

Get all the assessments of a course.

**Scope:** 'user_courses'

**Endpoint:** `/courses/{course_name}`

**Parameters:** [none]

**Responses:**

A list of assessments. If the user is only a student of the course, only released assessments are available. Otherwise, all assessments are available. Each assessment contains:

| key | type | description |
| --- | ---- | ----------- |
| name | string | The unique url-safe name. |
| display_name | string | The full name of the assessments. |
| description | string | A short description of the assessment. |
| start_at | datetime | The time this assessment is released to students. |
| due_at | datetime | Students can submit before this time without being penalized or using grace days. |
| end_at | datetime | Last possible time that students can submit (except those granted extensions.) |
| updated_at | datetime | The last time an update was made to the assessment. |
| max_grace_days | integer | Maximum number of grace days that a student can spend on this assessment. |
| max_submissions | integer | The maximum number of times a student can submit the assessment.<br>-1 means unlimited submissions. |
| disable_handins | boolean | Are handins disallowed by students? |
| category_name | string | Name of the category this assessment belongs to. |
| group_size | integer | The maximum size of groups for this assessment. |
| has_scoreboard | boolean | Does this assessment have a scoreboard? |
| has_autograder | boolean | Does this assessment use an autograder? |
| grading_deadline | string | *Not available if the user is a student.*<br>Time after which final scores are included in the gradebook. |

---
### problems

Get all problems of an assessment.

**Scope:** 'user_courses'

Endpoint `/courses/{course_name}/assessments/{assessment_name}/problems`

**Parameters:** [none]

**Responses:**

A list of problems. Each problem contains:

| key | type | description |
| --- | ---- | ----------- |
| name | string | Full name of the problem. |
| description | string | Brief description of the problem. |
| max_score | float | Maximum possible score for this problem. |
| optional | boolean | Is this problem optional? |

---
### writeup

Get the writeup of an assessment.

**Scope:** 'user_courses'

**Endpoint:** `/courses/{course_name}/assessments/{assessment_name}/writeup`

**Parameters:** [none]

**Responses:**

* If no writeup exists:<br>
404 Not Found: no writeup.

* If writeup is a url:

| key | type | description |
| --- | ---- | ----------- |
| url | string | The url of the writeup. |

* If writeup is a file:<br>
The file is returned.

---
### handout

Get the handout of an assessment.

**Scope:** 'user_courses'

**Endpoint:** `/courses/{course_name}/assessments/{assessment_name}/handout`

**Parameters:** [none]

**Responses:** [same as [writeup](/api-interface/#writeup)]

---
### submit

Make a submission to an assessment.

**Scope:** 'user_submit'

**Endpoint:** `POST /courses/{course_name}/assessments/{assessment_name}/submit`

**Parameters:**

* submission["file"]<br>
  The file to submit

**Success Response:**

| key | type | description |
| --- | ---- | ----------- |
| version | integer | The version number of the newly submitted submission. |
| filename | string | The final filename the submitted file is referred to as. |

**Failure Response:**

A valid submission request may still fail for many reasons, such as file too large, handins disabled by staff, deadline has passed, etc.

When a submission fails, the HTTP response code will not be 200. The response body will include a json with the key 'error'. Its contents will be a user-friendly string that the client may display to the user to explain why the submission has failed. The client *must not* repeat the request without any modifications. The client is *not* expected to be able to handle the error automatically.

---
### submissions

Get all submissions the user has made via this client. To protect the user's private scores and feedback, only the submissions made via your client is returned. i.e. submissions made through other clients or on the Autolab website are never returned.

**Scope:** 'user_scores'

**Endpoint:** `/courses/{course_name}/assessments/{assessment_name}/submissions`

**Parameters:** [none]

**Response:**

A list of submissions. Each submission includes:

| key | type | description |
| --- | ---- | ----------- |
| version | integer | The version number of this submission. |
| filename | string | The final filename the submitted file is referred to as. |
| created_at | datetime | The time this submission was made. |
| scores | object | A dictionary containing the scores of each problem.<br>The keys are the names of the problems, and the value is either the score (a float), or the string 'unreleased' if the score for this problem is not yet released. |

---
### feedback

Get the text feedback given to a problem of a submission.

For autograded assessments, the feedback will by default be the autograder feedback, and will be identical for all problems.

**Scope:** 'user_scores'

**Endpoint:** `/courses/{course_name}/assessments/{assessment_name}/submissions/{submission_version}/feedback`

**Parameters:**

* problem<br>
  The name of the problem that the feedback is given to.

**Response:**

| key | type | description |
| --- | ---- | ----------- |
| feedback | string | The full feedback text for this problem. |