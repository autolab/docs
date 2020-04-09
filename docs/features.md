# Autolab Features Documentation

This guide details the usage of features in Autolab.

Features Documented (Work in Progress):

-   [Scoreboards](#scoreboards)
-   [Embedded Forms](#embedded-forms)
-   [MOSS Plagiarism Detection](#moss)

## Scoreboards

Scoreboards are a central concept of Autolab and are created by the output of [Autograders](/docs/lab/#writing-autograders). They anonomously rank students submitted assignments inspiring health competition and desire to improve. They are simple and highly customizable. Scoreboard's can be added/edited on the edit assessment screen (`/courses/<course>/assessments/<assessment>/edit`).

![Scoreboard Edit](/docs/images/scoreboard_edit.png)

In general, scoreboards are configured using a JSON string.

### Default Scoreboard

The default scoreboard displays the total problem scores, followed by each individual problem score, sorted in descending order by the total score.

### Custom Scoreboards

Autograded assignments have the option of creating custom scoreboards. You can specify your own custom scoreboard using a JSON column specification.

The column spec consists of a "scoreboard" object, which is an array of JSON objects, where each object describes a column.

**Example:** a scoreboard with one column, called `Score`.

```json
{
    "scoreboard": [{ "hdr": "Score" }]
}
```

A custom scoreboard sorts the first three columns, from left to right, in descending order. You can change the default sort order for a particular column by adding an optional "asc:1" element to its hash.

**Example:** Scoreboard with two columns, "Score" and "Ops", with "Score" sorted descending, and then "Ops" ascending:

```json
{
    "scoreboard": [{ "hdr": "Score" }, { "hdr": "Ops", "asc": 1 }]
}
```

### Scoreboard Entries

The values for each row in a custom scoreboard come directly from a `scoreboard` array object in the autoresult string produced by the Tango, the autograder.

**Example:** Autoresult returning the score (97) for a single autograded problem called `autograded`, and a scoreboard entry with two columns: the autograded score (`Score`) and the number of operations (`Ops`):

```json
{
    "scores": {
        "autograded": 97
    },
    "scoreboard": [97, 128]
}
```

For more information on how to use Autograders and Scoreboards together, visit the [Guide for Lab Authors](/docs/lab/).

## Embedded Forms

This feature allows an instructor to create an assessment which does not require a file submission on the part of the student. Instead, when an assessment is created, the hand-in page for that assessment will display an HTML form of the instructor’s design. When the student submits the form, the information is sent directly in JSON format to the Tango grading server for evaluation.

!!! attention "Tango Required"
	Tango is needed to use this feature. Please install [Tango](/docs/tango/) and connect it to Autolab before proceeding.

![Embedded Form](/docs/images/embedded_quiz.png)

### Creating an Embedded Form

Create an HTML file with a combination of the following elements. The HTML file need only include form elements, because it will automatically be wrapped in a `<form></form>` block when it is rendered on the page.

In order for the JSON string (the information passed to the grader) to be constructed properly, your form elements must follow the following conventions:

-   A unique name attribute
-   A value attribute which corresponds to the correct answer to the question (unless it is a text field or text area)

HTML Form Reference:

**Text Field (For short responses)**

```html
<input type="“text”" name="“question-1”" />
```

**Text Area (For coding questions)**

```html
<textarea name="“question-2”" style="“width:100%”" />
```

**Radio Button (For multiple choice)**

```html
<div class="“row">
    <input name="“question-3”" type="“radio”" value="“object”" id="“q3-1”" />
    <label for="q3-1">Object</label>
    <input name="“question-3”" type="“radio”" value="“boolean”" id="“q3-2”" />
    <label for="“q3-2”">Boolean</label>
</div>
```

**Dropdown (For multiple choice or select all that apply)**

```html
<select multiple name="question-4">
    <option value="1">Option 1</option>
    <option value="2">Option 2</option>
    <option value="3">Option 3</option>
</select>
```

**Example Form (shown in screenshot above)**

```html
<input type="text" name="question-1" id="q1" placeholder="What's your name?" />

<div class="row">
    <input name="question-2" type="radio" value="freshman" id="q3-1" />
    <label for="q3-1">Freshman</label>

    <input name="question-2" type="radio" value="sophomore" id="q3-2" />
    <label for="q3-2">Sophomore</label>

    <input name="question-2" type="radio" value="junior" id="q3-3" />
    <label for="q3-3">Junior</label>

    <input name="question-2" type="radio" value="senior" id="q3-4" />
    <label for="q3-4">Senior</label>
</div>
<label for="q4">What's your favorite language?</label>
<select name="question-3" id="q4">
    <option value="C">C</option>
    <option value="Python">Python</option>
    <option value="Java">Java</option>
</select>
```

Navigate to the Basic section of editing an assessment (`/courses/<course>/assessments/<assessment>/edit`), check the check box, and upload the HTML file. Ensure you submit the form by clicking `Save` at the bottom of the page.

![Embedded Form Edit](/docs/images/embedded_quiz_edit.png)

### Grading an Embedded Form

When a student submits a form, the form data is sent to [Tango](/docs/tango/) in the form of a JSON string in the file `out.txt.` In your grading script, parse the contents of `out.txt` as a JSON object. The JSON object will be a key-value pair data structure, so you can access the students response string (`value`) by its unique key (the `name` attribute).

For the example form shown above, the JSON object will be as follows:

```json
{
    "utf8": "✓",
    "authenticity_token": "LONGAUTHTOKEN",
    "submission[embedded_quiz_form_answer]": "",
    "question-1": "John Smith",
    "question-2": "junior",
    "question-3": "Python",
    "integrity_checkbox": "1"
}
```

Use this information to do any processing you need in Tango.If you find any problems, please file an issue on the [Autolab Github](https://github.com/autolab/Autolab).


## Annotations

Annotations is a feature introduced as part of the Speedgrader update to Autolab. It allows instructors and TAs to quickly leave comments and grade code at the same time. 

![Annotation Form](/docs/images/annotations.png)

Hover over any line of the code and click on the green arrow, and the annotation form will appear. Add the comment, adjust the score, and select the targetted problem.

!!! attention "Non-Autograded Problems Only"
    Note that annotations can only be added to non-autograded problems. Specifically, a problem is non-autograded if there is no assigned score for that problem in the json outputted by the autograder

### Scoring Behavior

There are two intended ways for course instructors to use the add annotation features. Deductions from maximum, or additions from zero.

**Deductions from maximum**

Set a `max_score` either programmatically, or under `Edit Assessment > Problems` for the particular non-autograded question. Then when the grader is viewing the code, add negative score, such as `-5` into the score field, to deduct from the maximum. This use case is preferred when grading based on a rubric, and the score is deducted for each mistake.

The maximum score can be `0` if the deductions are meant to be penalties, such as for poor code style or violation of library interfaces.

**Additions from zero**

Set a `max_score` either programmatically, or under `Edit Assessment > Problems` for the particular non-autograded question to `0`. When the grader is viewing the code, add positive scores, such as `5` to the score field, to add to the score. This use case is preferred when giving out bonus points.

### Interaction with Gradesheet

We have kept the ability the edit the scores in the gradesheet, as we understand that there are instances in which editing the gradesheet directly is much more efficient and/or needed. However, this leads to an unintended interaction with the annotations.

In particular, modifications on the gradesheet itself will override all changes made to a problem by annotations, but the annotations made will still remain. 

A example would be, if the `max_score` of a problem is `10`. A TA adds an annotation with `-5` score to that problem (so the score is now `10-5=5`). Then if the same TA or another TA changes the score to `8` on the gradesheet, the final score would be `8`.

**Recommendation**

It is much preferred to grade using annotations whenever possible,
as it provides a better experience for the students who will be able to identify the exact line at which the mistake is made. Gradesheet should be used in situations where the modification is non-code related.


## MOSS Plagiarism Detection Installation

[MOSS (Measure Of Software Similarity)](https://theory.stanford.edu/~aiken/moss/) is a system for checking for plagiarism. MOSS can be setup on Autolab as follows:

1. Obtain the script for MOSS based on the instructions given in [https://theory.stanford.edu/~aiken/moss/](https://theory.stanford.edu/~aiken/moss/).

2. Create a directory called `vendor` at the root of your Autolab installation, i.e

	```bash
	cd <autolab_root>
	mkdir -p vendor
	```

3. Copy the moss script into the `vendor` directory and name it `mossnet`

	```bash
	mv <path_to_moss_script> vendor/mossnet
	```
