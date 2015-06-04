# AlephBet

AlephBet is a pure-javascript A/B (multivariate) testing framework for developers.

Key Features:

* Pluggable event tracking backend (defaults to Google Universal Analytics)
* Supports multiple variants and goals
* Tracks unique visitors and goal completions
* Flexible triggers
* Ideal for use with page and fragment caching
* Developer-friendly for both usage and contirbution (using npm / browserify)
* less than 5kb when minified and gzipped with no external dependencies

## What does AlephBet mean?

Aleph (אלף) Bet (בית) are the first two letters in the Hebrew alphabet. Similar to A and B.

## Inspiration

AlephBet was heavily inspired by Optimizely (sans WYSIWYG and reporting) and
[Cohorts.js](https://github.com/jamesyu/cohorts).
The code structure and some code elements were taken from cohorts.js, with some notable changes to terminology and
built-in support for unique goals and visitor tracking.

## Quick Start

* Make sure your Google Universal analytics is set up.
* Include `alephbet.min.js` in the head section of your HTML.
* Create an experiment:

```javacript
var button_color_experiment = new AlephBet.Experiment({
  name: 'button color',  // the name of this experiment; required.
  variants: {  // variants for this experiment; required.
    blue: {
      activate: function() {  // activate function to execute if variant is selected
        $('#my-btn').attr('style', 'color: blue;');
      }
    },
    red: {
      activate: function() {
        $('#my-btn').attr('style', 'color: red;');
      }
    }
  },
});
```

* Track goals for your experiment:

```javascript
// tracking unique goal actions
$('#my-btn').on('click', function() {
  // The chosen variant will be tied to the goal automatically
  button_color_experiment.goal('button clicked');
});

// tracking non-unique goals, e.g. page views
button_color_experiment.goal('viewed page', false);  // setting unique to false
```

* view results on your Google Analytics Event Tracking Section. Visitors + Goals will be assigned to `actions`. e.g.
  - `red | Visitors` : unique count of visitors assigned to the `red` variant.
  - `red | button clicked` : unique visitors clicking on the button.
  - `red | viewed page` : count of pages viewed by all visitors (not-unique) *after* the experiment started.

## Advanced Usage

### Recommended Usage Pattern

AlephBet was meant to be used across different pages, tracking multiple goals over simultaneous experiments. It is
therefore recommended to keep all experiments in one javascript file, shared across all pages. This allows sharing goals
across different experiments. Experiments can be triggered based on a set of conditions, allowing to fine-tune the
audience for the experiments (e.g. mobile users, logged-in etc).

### Triggers

Experiments automatically start by default. However, a trigger function can be provided, to limit the audience or the
page(s) where the experiment takes place.

```javascript
var button_color_experiment = new AlephBet.Experiment({
  name: 'button color',
  trigger: function() {
    return window.location.href.match(/pricing/);
  },
  variants: { // ...
  },
});

// triggers can be assigned to a variable and shared / re-used
var logged_in_user = function() { return document.cookie.match(/__session/); };
var mobile_browser = function() { // test if mobile browser };

var big_header_experiment = new AlephBet.Experiment({
  name: 'big header',
  trigger: function() { return logged_in_user() && mobile_browser(); },
  // ...
});
```

### Sample size

You can specify a `sample` float (between 0.0 and 1.0) to limit the number of visitors participating in an experiment.

### Visitors

Visitors will be tracked once they participate in an experiment (and only once). Once a visitor participates in an
experiment, the same variant will always be shown to them. If visitors are excluded from the sample, they will be
permanently excluded from seeing the experiment. Triggers however will be checked more than once, to allow launching
experiments under specific conditions for the same visitor.

### Goals

Goals are uniquely tracked by default. i.e. if a goal is set to measure how many visitors clicked on a button, multiple
clicks won't generate another goal completion. Only one per visitor. Non-unique goals can be set by passing a second
parameter (`false`) to the goal method.

Goals will only be tracked if the experiment was launched and a variant selected before. Tracking goals is therefore
safe and idempotent (unless unique is false).

Here's a short sample of tracking multiple goals over multiple experiments:

```javascript

var button_color_experiment = new AlephBet.Experiment({ /* ... */ });
var buy_button_cta_experiment = new AlephBet.Experiment({ /* ... */ });
var experiments = [button_color_experiment, buy_button_cta_experiment];

// main goal - button click
$('#my-btn').on('click', function() {
  _(experiments).each(function (ex) { ex.goal('button clicked'); });
});

// engagement - any click on the page
$('html').on('click', function() {
  _(experiments).each(function (ex) { ex.goal('engagement'); });
});
```

### Custom Tracking Adapter

AlephBet comes with a built-in Google Analytics adapter. Creating custom adapters is however very easy.

Here's an example for integrating an adapter for [keen.io](https://keen.io)

```html
<script src="https://d26b395fwzu5fz.cloudfront.net/3.2.4/keen.min.js" type="text/javascript"></script>
<script src="alephbet.min.js"></script>
<script type="text/javascript">
    window.keen_client = new Keen({
        projectId: "ENTER YOUR PROJECT ID",
        writeKey: "ENTER YOUR WRITE KEY"
    });
    var tracking_adapter = {
        onInitialize: function(experiment_name, variant) {
            keen_client.addEvent(experiment_name, {variant: variant, event: 'participate'});
        },
        onEvent: function(experiment_name, variant, event_name) {
            keen_client.addEvent(experiment_name, {variant: variant, event: event_name});
        }
    };
    var my_experiment = new AlephBet.Experiment({
        name: 'my experiment',
        variants: { // ...
        },
        tracking_adapter: tracking_adapter,
        // ...
    });
</script>

```

### Debug mode

To set more verbose logging to the browser console, use `AlephBet.options.debug = true`.

## Development

AlephBet uses npm / browserify with the following 3rd party libraries:

* [lodash](https://lodash.com/) (using a custom build, to save space)
* [store.js](https://github.com/marcuswestin/store.js) - for localStorage

### Commands

* `npm run build` - to build both development and minified version + prerequisites
* `npm run watch` - will watch files and re-build using `watchify`

## License

AlephBet is distributed under the MIT license. All 3rd party libraries and components are distributed under their
respective license terms.

```
Copyright (C) 2015 Yoav Aner

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated
documentation files (the "Software"), to deal in the Software without restriction, including without limitation the
rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit
persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the
Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE
WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR
OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
```