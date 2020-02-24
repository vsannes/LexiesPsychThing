# LexiesPsychThing
<!DOCTYPE html>
<html>

<!-- This HTML file is part of the supplemental material of the article "PageFocus: Using Paradata to Detect and Prevent Cheating on Online Achievement Tests" by Birk Diedenhofen and Jochen Musch. The JavaScript section of the file includes version 1.3 of the PageFocus script.
Copyright 2015-2017 Birk Diedenhofen (mail@birkdiedenhofen.de)
This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version. This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details. You should have received a copy of the GNU General Public License along with this program. If not, see <http://www.gnu.org/licenses/>. -->

<head>
<title>PageFocus Demo</title>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />

<style type="text/css">
  body {
    font-family: sans-serif;
  }

  .numeric_input {
    text-align: right;
  }

  .footnote {
    font-size: 0.8em;
  }

  #popup_warning {
    display: none;
    width: 500px;
    height: 180px;
    position: absolute;
    background-color: #fff;
    z-index: 255;
    text-align: center;
    padding: 0 20px 20px;
    border: 5px solid red;
    top: 50%;
    left: 50%;
    margin-left: -250px;
    margin-top: -90px;
  }

  #popup_warning .ok_button {
    margin-top: 20px;
  }

  #popup_warning .ok_button button {
    width: 100px;
    height: 30px;
  }

  #disabled {
    display: none;
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background: #707070;
    filter:alpha(opacity=75);
    opacity: 0.75;
    z-index: 253;
  }
</style>

</head>

<body>
<h1>PageFocus Demo</h1>

<p>This page demonstrates the functionality of the PageFocus script (version 1.3), which has been added to this page in its JavaScript section.</p>

<p>If you switch from this page to another browser tab or window, you are triggering a page defocusing event (D). If you return to this page, you trigger a page refocusing event (R). Both for the defocusing event and for the refocusing event, a timestamp is set that records the number of milliseconds that have passed since 1.1.1970 (as a 13 digit number). By calculating the difference between the two timestamps, the duration of the absence of the test taker is determined.</p>

<hr>

<p>By toggling the below checkbox, you may activate a popup warning that is presented whenever a page defocusing event is detected.</p>
<p><label><input type="checkbox" name="popup_warning_checkbox" id="popup_warning_checkbox"> Show popup warning</label></p>

<hr>

<p><label>Page defocusing (D) and refocusing events (R):
<input type="text" id="events_input" name="events_input" size="80" readonly></label></p>

<p><label>Number of page defocusing events: <input class="numeric_input" type="text" id="defocus_count_input" name="defocus_count_input" size="4" value="0" readonly></label></p>

<p><label>Number of page refocusing events:
<input class="numeric_input" type="text" id="refocus_count_input" name="refocus_count_input" size="4" value="0" readonly></label></p>

<p><label>Duration between last page defocusing and last refocusing event (Duration of last absence):
<input class="numeric_input" type="text" id="last_duration_input" name="last_duration_input" size="10" value="0" readonly> seconds</label></p>

<p><label>Sum of all durations between page defocusing and refocusing events (Sum of all absence durations):

<input class="numeric_input" type="text" id="duration_sum_input" name="duration_sum_input" size="10" value="0" readonly> seconds</label></p>

<p><button onclick="reset()">Reset</button></p>

<hr>

<p class="footnote">This HTML file is part of the supplemental material for the article "PageFocus: Using Paradata to Detect and Prevent Cheating on Online Achievement Tests" by <a href="mailto:mail@birkdiedenhofen.de">Birk Diedenhofen</a> and Jochen Musch. The JavaScript section of the file includes the revised version 1.3 auf the PageFocus script that also works with mobile Safari browsers. The PageFocus script is published under the <a href="http://www.gnu.org/licenses/">GNU General Public License (version 3)</a>.
</p>

<div id="popup_warning">
  <h3>PageFocus warning</h3>
  <p>You have just left this window which is not allowed while participating on the test. Please return to the test by confirming this message, and do not leave the test page again.</p>
  <div class="ok_button"><button onclick='toggle_popup_warning("none"); regained_focus(); return false;'>OK</button></div>
</div>
<div id="disabled"></div>

<script type="text/javascript">
// PageFocus version 1.3-1 demo

var focus_data;  // all page defocusing and refocusing events as one string
var defocusing_count;  // number of page defocusing events
var refocusing_count;  // number of page refocusing events
var last_duration;  // duration between the last page defocusing and refocusing events
var duration_sum;  // sum of all durations between page defocusing and refocusing events

var defocus_timestamp;  // timestamp of last page defocusing event
var refocus_timestamp;  // timestamp of last page refocusing event

var pagefocus = true;  // does the current page have the focus?
var popup_visible = false;  // is the popup currently being shown?

// input elements
var popup_checkbox = document.getElementById("popup_warning_checkbox");
var events_input = document.getElementById("events_input");
var defocus_count_input = document.getElementById("defocus_count_input");
var refocus_count_input = document.getElementById("refocus_count_input");
var last_duration_input = document.getElementById("last_duration_input");
var duration_sum_input = document.getElementById("duration_sum_input");

reset()

record_timestamp = function(type, timestamp) {
  focus_data = focus_data + type + timestamp + ";";  // add new page focusing event to the data record
  events_input.value = focus_data;

  events_input.scrollLeft = events_input.scrollWidth;  // scroll input field to the right
}

function lost_focus() {  // page defocusing event detected
  if(!popup_visible) {
    pagefocus = false;

    defocus_timestamp = new Date().getTime();
    record_timestamp("D", defocus_timestamp);

    defocusing_count++;  // count the number of defocusing events
    defocus_count_input.value = defocusing_count;

    if(popup_checkbox.checked) toggle_popup_warning("block");
  }
}

function regained_focus() {  // page refocusing event detected
  if(!pagefocus && !popup_visible) {
    pagefocus = true;

    refocus_timestamp = new Date().getTime();
    record_timestamp("R", refocus_timestamp);

    refocusing_count++;  // count the number of refocusing events
    refocus_count_input.value = refocusing_count;

    last_duration = refocus_timestamp - defocus_timestamp; // calculate the duration between the last page defocusing and refocusing events
    duration_sum += last_duration; // sum durations between page defocusing and refocusing events

    last_duration_input.value = last_duration/1000;
    duration_sum_input.value = duration_sum/1000;
  }
}

function onfocusout() {  // workaround for Internet Explorer < version 11
  clearTimeout(timer);
  timer = setTimeout(lost_focus,100);
}

function onfocusin() {  // workaround for Internet Explorer < version 11
  clearTimeout(timer);
  regained_focus();
}

function reset() {  // reset captured data
  events_input.value = focus_data = '';
  defocus_count_input.value = defocusing_count = 0;
  refocus_count_input.value = refocusing_count = 0;
  last_duration_input.value = last_duration = 0;
  duration_sum_input.value = duration_sum = 0;
}

function toggle_popup_warning(state) {  // show/hide popup warning
  document.getElementById("popup_warning").style.display = state;
  document.getElementById("disabled").style.display = state;
  popup_visible = state == "block";
}

if("onfocusin" in document) {  // check for Internet Explorer version < 11
  var timer;
  document.onfocusin = onfocusin;
  document.onfocusout = onfocusout;
} else if("onpageshow" in window) {
  // use onpageshow and onpagehide for mobile Safari
  window.onfocus = window.onpageshow = regained_focus;
  window.onblur = window.onpagehide = lost_focus;
}

</script>

</body>
</html>
