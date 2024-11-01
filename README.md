# Dynamic Report Status Dashboard Using Arcade Expressions



## Overview

This script is an Arcade Expression designed for use in ArcGIS Dashboards, specifically within the advanced formatting options of List Elements. It generates an HTML representation of a report's validation process status, showcasing key information such as the report ID, the last status change, and the next steps in the review process. The script combines SVG graphics for visual representation and animations, enhancing the user experience with dynamic feedback.

In this script, SVG is used to visually represent data elements like the report ID and last status change. SMIL animations are integrated to create engaging visual effects, making status changes more noticeable. The advanced formatting capabilities of ArcGIS Dashboards allow for the seamless integration of this HTML and SVG content, providing a rich user experience. The Arcade Expression serves as the backbone of this functionality, dynamically pulling in data and rendering it in an easily digestible format.

- **SVG (Scalable Vector Graphics)** is an XML-based format for vector graphics that allows for high-quality images that can scale to any size without losing clarity. SVG is widely used on the web because it can be manipulated via inline styling and JavaScript, making it highly versatile for graphical presentations in web applications.

- **SMIL (Synchronized Multimedia Integration Language)** animations allow for the creation of animations directly within SVG files. This means you can animate attributes of SVG elements, such as position, size, and colour, using simple XML syntax. SMIL animations enable developers to enhance user interfaces with engaging, dynamic graphics without relying on external libraries.

- In ArcGIS Dashboards, **advanced formatting** allows users to customise the appearance of dashboard elements significantly. This includes the use of HTML to style data presentations, making it possible to create visually compelling and informative layouts tailored to specific user needs.

- **Arcade Expressions** are a scripting language used in the ArcGIS platform to perform calculations and data manipulations on the fly. They enable users to create custom expressions that can drive the presentation of data within maps, dashboards, and other applications, providing a flexible way to display information based on user-defined logic.

- **HTML (Hypertext Markup Language)** is the standard markup language for creating web pages. It structures content on the web and can include text, images, links, and other media. **Inline HTML design** refers to incorporating HTML directly within other programming or scripting languages, allowing for dynamic content generation based on the underlying logic.


## Fields Overview

Before delving into the code, it is essential to understand the fields employed in this script:

- **`sys_workflow_status`**: This field indicates the current status of the report within the workflow. Possible values include:
  - `submit_to_team`: The report was rejected, requiring the team to address the feedback and resubmit.
  - `submit_to_org`: The report is currently awaiting review by the organisation's operations team.
  - `submit_to_org_im`: The report is under review by the organisation's Information Manager.
  - `submit_to_libmac`: The report is with the LibMAC operations officer for review.
  - `submit_to_im`: The report is in the final validation stage.

- **`last_review_step`**: This field stores the last review action taken on the report, helping to determine the necessary next steps.

- **`last_status_change`**: This field records the date of the last status change, enabling the script to calculate the duration since the last update.

- **`record_id`**: A unique identifier for the report, prominently displayed in the output.

## Code Overview

```javascript
var html = "<table border='0' cellspacing='0' style='border-collapse:collapse; color:#bdbdbd; font-family:\"Consolas\"; font-size:10px; width:90%'>" +
    "<tbody>" +
    "<tr>";
```
- This initialises an HTML string for a table with specific styles, including font, colour, and size.

### Status Variables

```javascript
var status = $datapoint["sys_workflow_status"];
var lastReviewStep = $datapoint["last_review_step"];
var lastStatusChange = $datapoint["last_status_change"];
var nextStep;

var nextStepColor;
```
- Retrieves the current workflow status, the last review step, and the last status change date from a data point object. `nextStepColor` will be used to define the colour based on the current status.

### Define Next Step Colour

```javascript
if (status == "submit_to_team") {
    nextStepColor = "#BA494A"; // Red for rejection
} else if (status == "submit_to_org" || status == "submit_to_org_im" || status == "submit_to_libmac") {
    nextStepColor = "#8f8f8f"; // Grey for review stages
} else if (status == "submit_to_im") {
    nextStepColor = "#6f7d6e"; // Green for final validation
}
```
- This segment assigns a colour to `nextStepColor` based on the current status, which will be applied in the SVG elements.

### Create SVG for Report ID

```javascript
var recordIdHeight = 21; // Adjust height to fit the text comfortably
var recordIdSvg = 
    "<svg width='2' height='" + recordIdHeight + "'>" +
        "<rect x='0' y='0' width='4' height='" + recordIdHeight + "' rx='0' opacity='0.5' fill='" + nextStepColor + "'></rect>" +
        "<text x='10' y='" + (recordIdHeight / 2 + 4) + "' font-size='11' fill='#bdbdbd' text-anchor='left' font-family='Consolas'>Report ID: " + $datapoint["record_id"] + "</text>" +
    "</svg>";

html += "<td style='width:50%; vertical-align: top;'>" + recordIdSvg + "</td>";
```
- This creates an SVG element that displays the report ID, styled with a rectangle and text. The rectangle's fill colour is determined by `nextStepColor`.

### Last Status Change Message

```javascript
var lastStatusChangeMessage = ""; // Initialise the message

if (lastStatusChange != Null) {
    // Calculate days since last status change
    var startDate = Date(lastStatusChange);
    var endDate = Date(now());
    var daysSinceLastChange = Round(DateDiff(endDate, startDate, 'days'));
    
    // Determine the appropriate time unit
    if (daysSinceLastChange >= 365) {
        var years = Floor(daysSinceLastChange / 365);
        lastStatusChangeMessage = "The last status change was on " + Text(lastStatusChange, 'dddd, DD MMMM YYYY') + " (" + years + " year";
        if (years > 1) {
            lastStatusChangeMessage += "s";
        }
        lastStatusChangeMessage += " ago)";
    } else if (daysSinceLastChange >= 30) {
        var months = Floor(daysSinceLastChange / 30);
        lastStatusChangeMessage = "The last status change was on " + Text(lastStatusChange, 'dddd, DD MMMM YYYY') + " (" + months + " month";
        if (months > 1) {
            lastStatusChangeMessage += "s";
        }
        lastStatusChangeMessage += " ago)";
    } else if (daysSinceLastChange >= 7) {
        var weeks = Floor(daysSinceLastChange / 7);
        lastStatusChangeMessage = "The last status change was on " + Text(lastStatusChange, 'dddd, DD MMMM YYYY') + " (" + weeks + " week";
        if (weeks > 1) {
            lastStatusChangeMessage += "s";
        }
        lastStatusChangeMessage += " ago)";
    } else {
        lastStatusChangeMessage = "The last status change was on " + Text(lastStatusChange, 'dddd, DD MMMM YYYY') + " (" + daysSinceLastChange + " day";
        if (daysSinceLastChange > 1) {
            lastStatusChangeMessage += "s";
        }
        lastStatusChangeMessage += " ago)";
    }
} else {
    lastStatusChangeMessage = "The report is currently awaiting review.";
}
```
- This section checks the `lastStatusChange` variable and calculates the number of days since that change. The message is formatted based on how much time has elapsed (years, months, weeks, or days). If no status change exists, it sets a default message indicating that the report is awaiting review.

### Create SVG for Last Status Change

```javascript
var lastStatusChangeHeight = recordIdHeight; // Use the same height for consistency
var lastStatusChangeSvg;

if (lastStatusChange != Null && daysSinceLastChange > 10) {
    lastStatusChangeSvg = 
        "<svg width='2' height='" + lastStatusChangeHeight + "'>" +
            "<rect x='0' y='0' width='4' height='" + lastStatusChangeHeight + "' rx='0' opacity='0.5' fill='" + nextStepColor + "'></rect>" +
            "<text x='10' y='" + (lastStatusChangeHeight / 2 + 4) + "' font-size='11' fill='#bdbdbd' text-anchor='left' font-family='Consolas'>" + 
            "<animate attributeName='opacity' values='1;0;1' dur='2s' repeatCount='indefinite' />" + 
            lastStatusChangeMessage + 
            "</text>" +
        "</svg>";
} else {
    lastStatusChangeSvg = 
        "<svg width='2' height='" + lastStatusChangeHeight + "'>" +
            "<rect x='0' y='0' width='4' height='" + lastStatusChangeHeight + "' rx='0' opacity='0.5' fill='" + nextStepColor + "'></rect>" +
            "<text x='10' y='" + (lastStatusChangeHeight / 2 + 4) + "' font-size='11' fill='#bdbdbd' text-anchor='left' font-family='Consolas'>" + lastStatusChangeMessage + "</text>" +
        "</svg>";
}

html += "<td style='width:50%; vertical-align: top;'>" + lastStatusChangeSvg + "</td>" +
    "</tr>";
```
- Similar to the report ID, this creates an SVG for the last status change, including an opacity animation if the last change was more than ten days ago, enhancing the visual feedback.

### New Row for Next Step

```javascript
html += "<tr><td colspan='3' style='vertical-align: top; padding-top: 20px; padding-bottom: 20px;'>";

var nextStep;

// SVGs for animations
var warningSvg = 
    "<svg width='24' height='24' style='vertical-align:middle; margin-right: 5px;'>" +
        "<rect x='0' y='0' width='20' height='20' rx='4' fill='#BA494A'>" +
            "<animate attributeName='fill' values='#BA494A;#474749;#BA494A' dur='1.5s' repeatCount='indefinite'/></rect>" +
        "<text x='10' y='13.5' font-size='11' fill='#bdbdbd' text-anchor='middle' font-family='Arial'>✕</text>" +
    "</svg>";

var finaliseSvg = 
    "<svg width='24' height='24' style='vertical-align: middle; margin-right: 5px;'>" +
        "<rect x='0' y='0' width='20' height='20' rx='4' fill='#6f7d6e'>" +
            "<animate attributeName='fill' values='#6f7d6e;#474749;#6f7d6e' dur='1.5s' repeatCount='indefinite'/></rect>" +
        "<text x='10' y='14' font-size='11' fill='#bdbdbd' text-anchor='middle' font-family='Arial'>✔</text>" +
    "</svg>";

var reviewSvg = 
    "<svg width='24' height='24' style='vertical-align: middle; margin-right: 5px;'>" +
        "<rect x='0' y='0' width='20' height='20' rx='4' fill='#8f8f8f'></rect>" +
        "<text x='10' y='14' font-size='11' fill='#bdbdbd' text-anchor='middle' font-family='Arial'>✔</text>" +
    "</svg>";

if (status == "submit_to_team") {
    nextStep = warningSvg + " The report was rejected. Teams are required to address the feedback and resubmit.";
} else if (status == "submit_to_org") {
    nextStep = reviewSvg + " The report is awaiting review from the organisation.";
} else if (status == "submit_to_org_im") {
    nextStep = reviewSvg + " The report is being reviewed by the Information Manager.";
} else if (status == "submit_to_libmac") {
    nextStep = reviewSvg + " The report is with the LibMAC officer for review.";
} else if (status == "submit_to_im") {
    nextStep = finaliseSvg + " The report is with the Information Management Officer for final validation.";
} else {
    nextStep = status; // Keep the original status if it does not match
}
```
- This section establishes a new row dedicated to the next steps, utilising SVG icons to visually represent the current workflow status. Depending on the `status`, a message is assigned to `nextStep`, describing what actions are required next.

### Final HTML Structure

```javascript
html += 
"<div class='callout' style='position: relative; width: 110%; border: 1px lightgray; padding: 20px; border-radius: 0px; background-color: #444444; display: flex; align-items: center; box-shadow: 2px 3px 5px rgba(0,0,0,.2);'>" +
    "<div style='flex-grow: 1; color: #bdbdbd;'>" + nextStep + 
    "</div></div></td></tr>" +
    "</tbody>" +
    "</table>";
```
- This constructs the final output by wrapping the next step message in a visually appealing callout box. It employs inline styling to enhance the layout and make it stand out.

### Closing the Table

```javascript
return {
    separatorColor: '#3f3f41',
    attributes: {
        attribute1: html,
    }
};
```
- This finalises the HTML table structure and returns it, making it ready for display within the ArcGIS Dashboard.
- In Line item template it is important to reference the html attribute from the arcade expression using {expression/attribute1}

## Conclusion

This Arcade Expression effectively utilises data retrieval and visual representation techniques within ArcGIS Dashboards.
By integrating SVG and SMIL animations, it creates a dynamic and engaging user experience. The use of HTML and inline styling ensures that the information is not only functional but also visually appealing.
This script can be easily modified to accommodate additional statuses or fields as needed, ensuring flexibility in report tracking across various organisational contexts.
