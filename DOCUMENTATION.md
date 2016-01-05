Documentation
=============
There are a total of three places you need to configure, in that order:

1. `_ data/goals/`
    - and `_data/info.yml`, which is experimental and only contains term information
2. `_config.yml`
3. `index.html`

In that order.

### `_data/goals/` ###

The project contains these 4 goal datafiles by default:

1. `achieved.yml`
2. `failed.yml`
3. `unfinished.yml`
4. `wip.yml`

*Remember these four goal( type)s, because they have to match across the aforementioned three places.* If you wish for them to go buy another name, you might as well start renaming them from this point on in the guide.

Here is what `achieved.yml` looks like:

```yaml
- short: Mandatory MP declaration of income sources
  long: bar
  topic: Government

- short: Fixed election terms
  long: bar
  topic: Government

- short: 60% election turnout
  long: bar
  topic: Government
```

`achieved.yml` is a so-called `YAML` file, the simplest and most accessible data format (compared to formats like CSV, TSV, JSON).

In each YAML file is a list of goals with the three values:

1. `short`: A short description of the goal, which is displayed on the site.
2. `long`: A longer description of the goal, which is currently not being used.
3. `topic`: The type of topic the goal belongs in. Because the word “category” is a reserved keyword in Jekyll, you have to user another term; hence “topic”.

To access and parse a datafile like `achieved.yml` in your Jekyll template, such as `index.html`, you write the following using the Liquid template:

    {{ site.data.goals.achieved }}

`{{ site.data }}` here refers to the `_data/` folder.

### `_config.yml` ###

Here is the relevant part of `_config.yml`—which is also a YAML file by the way:

```yaml
goal_types:
    - unfinished
    - wip
    - achieved
    - failed
topics:
    - Civil Liberties
    - Environment
    - Government
```

You will recognize both the goal types and topics from before.

### `index.html` ###

This is where it gets more technical, as the data finally has to be loaded, processed, and manipulated.

As you read through this, keep in mind that this is the end result we want and get:

![Screenshot of the finished design][screenshot]

#### `<head>` ####

`<head>` contains this lovely chunk of [Liquid markup][]:

```liquid
{% comment %}Shorter variable names for readability{% endcomment %}
    {% assign info = site.data.info %}
    {% assign goals = site.data.goals %}

{% comment %}Count different goals{% endcomment %}
{% for goal in goals.unfinished %}
    {% if forloop.last == true %}
        {% assign num_unfinished = forloop.length %}
    {% endif %}
{% endfor %}
{% for goal in goals.wip %}
    {% if forloop.last == true %}
        {% assign num_wip = forloop.length %}
    {% endif %}
{% endfor %}
{% for goal in goals.achieved %}
    {% if forloop.last == true %}
        {% assign num_achieved = forloop.length %}
    {% endif %}
{% endfor %}
{% for goal in goals.failed %}
    {% if forloop.last == true %}
        {% assign num_failed = forloop.length %}
    {% endif %}
{% endfor %}
{% capture num_total %}{{ num_unfinished | plus:num_wip | plus:num_achieved | plus:num_failed }}{% endcapture %}

{% capture ratio_unfinished %}{{ num_unfinished | times:100.0 | divided_by:num_total | round }}{% endcapture %}
{% capture ratio_wip %}{{ num_wip | times:100.0 | divided_by:num_total | round }}{% endcapture %}
{% capture ratio_achieved %}{{ num_achieved | times:100.0 | divided_by:num_total | round }}{% endcapture %}
{% capture ratio_failed %}{{ num_failed | times:100.0 | divided_by:num_total | round }}{% endcapture %}
```

Let’s try to break this down piece by piece.

```liquid
{% comment %}Shorter variable names for readability{% endcomment %}
    {% assign info = site.data.info %}
    {% assign goals = site.data.goals %}
```

This creates two shorthand variables:

* `info`
* `goals`

The stuff in our `_data/` folder is loaded in Jekyll using `site.data`—and outside the `{% tags %}` as `{{ site.data }}` in Liquid markup, which is the template language of Jekyll.

The `goals/` folder is accessed with `site.data.goals`; YML files like `wip.yml` are accessed with `site.data.goals.wip`. (Notice the lack of extension.) You get the concept by now.

We use the `{% assign %}` tag, because we’re lazy, to create a shorthand variable for our `info/` and `goals/` folders without `site.data`. Again, this is pure laziness, but it makes future code a lot more readable.

Moving on to the next part of the snippet:

```liquid
{% comment %}Count different goals{% endcomment %}
{% for goal in goals.unfinished %}
    {% if forloop.last == true %}
        {% assign num_unfinished = forloop.length %}
    {% endif %}
{% endfor %}
{% for goal in goals.wip %}
    {% if forloop.last == true %}
        {% assign num_wip = forloop.length %}
    {% endif %}
{% endfor %}
{% for goal in goals.achieved %}
    {% if forloop.last == true %}
        {% assign num_achieved = forloop.length %}
    {% endif %}
{% endfor %}
{% for goal in goals.failed %}
    {% if forloop.last == true %}
        {% assign num_failed = forloop.length %}
    {% endif %}
{% endfor %}
```

If you look at the screenshot, you can see that we want a count for each of our four types of goals as well as a sum total. This section accounts for, well, our counting.

As you can probably tell, this contains for similar for loops for each of our four types of goals, so let’s just focus on the first one of them:

```liquid
{% comment %}Count different goals{% endcomment %}
{% for goal in goals.unfinished %}
    {% if forloop.last == true %}
        {% assign num_unfinished = forloop.length %}
    {% endif %}
{% endfor %}
```

First, we commence a loop through all the Unfinished goals.

Then, we wait until we’ve reached the last one of the Unfinished goals.

We then create a variable, `num_unfinished`, based on the “length” of the list of Unfinished goals. Length is programming-speak for the number of items in a list, so we are basically counting the number of Unfinished goals.

If this seems like an abstruse roundabout way of doing this to you, then you are absolutely correct. This is a hack for counting the number of goals in a list within the limited capabilities of the Liquid template language—unlike a regular *programming language* for which this would be trivial.

The next part reads:

```liquid
{% capture num_total %}{{ num_unfinished | plus:num_wip | plus:num_achieved | plus:num_failed }}{% endcapture %}
```

Unless you a very familiar with Jekyll and Liquid markup, you have no idea what the hell is going on here, so I will spoil it for you: we are adding up the numbers of all the four goal types to get a sum total.

This contains yet another hack in order to get what we want. Because you can’t create a variable defined by other values the regular way using `{% assign %}` as we did above for our shorthand definitions, we have to do something else.

`{% capture %}` instead just eats whatever you manage to manage to type and spit out, and stores it as a variable. The tag is mainly intended for saving large chunks of text and doing stuff with out, but it gets our job done nonetheless.

Inside the capturing tag, we see

```liquid
{{ num_unfinished | plus:num_wip | plus:num_achieved | plus:num_failed }}
```

We start off with the variable from before, `num_unfinished`.

We then use a so-called template filter, `plus`, to perform an addition with the three remaining goal-type counts. Where a sane solution would read

```python
num_total = num_unfinished + num_wip + num_achieved + num_failed
```

Liquid requires us to write this monstrosity

```liquid
{% capture num_total %}{{ num_unfinished | plus:num_wip | plus:num_achieved | plus:num_failed }}{% endcapture %}
```

With our goal counts and sum total, we can now calculate each goal type’s share of the total number of goals:

```liquid
{% capture ratio_unfinished %}{{ num_unfinished | times:100.0 | divided_by:num_total | round }}{% endcapture %}
{% capture ratio_wip %}{{ num_wip | times:100.0 | divided_by:num_total | round }}{% endcapture %}
{% capture ratio_achieved %}{{ num_achieved | times:100.0 | divided_by:num_total | round }}{% endcapture %}
{% capture ratio_failed %}{{ num_failed | times:100.0 | divided_by:num_total | round }}{% endcapture %}
```

As usual, we are doing the same thing four times, so let’s focus on the first instance:

```liquid
{% capture ratio_unfinished %}{{ num_unfinished | times:100.0 | divided_by:num_total | round }}{% endcapture %}
```

As before, we use our devilish `{% capture %}` and filter trick, so eyes on what’s inside the `capture` tag:

```liquid
{{ num_unfinished | times:100.0 | divided_by:num_total | round }}
```

We all know from high-school maths that to get the share of something out of a total, you calculate it with

    share = num / total * 100.0

In Liquid, we write the equivalent of

    share = num * 100.0 / total

We have to multiply by 100.0 first, because we run into some Liquid wonkiness if we do it the other way around. Also, the round filter doesn’t work for some reason, because Liquid. As of right now, you also get a bunch of decimals on the result value, which wasn’t the intention.

This concludes the `<head>` section!

#### `<body>` ####

First we look at the so-called Goal Overview:

##### Goal Overview #####

```liquid
<ul id="goal-overview" class="u-max-full-width">
    <!-- Liquid template doesn't have a `timesince` filter -->
    <li class="tenure"><a><em>Since {{ info.tenure.start | date: "%B %-d, %Y" }}</em> In office</a></li>
    <li class="unfinished"><a title="Unfinished goals make up {{ ratio_unfinished }}%"><em>{{ num_unfinished }} of {{ num_total }}</em> Unfinished</a><span style="width:{{ ratio_unfinished }}%">{{ ratio_unfinished }}%</span></li>
    <li class="wip"><a title="Goals in progress make up {{ ratio_wip }}%"><em>{{ num_wip }} of {{ num_total }}</em> In progress</a><span style="width:{{ ratio_wip }}%">{{ ratio_wip }}</span></li>
    <li class="achieved"><a title="Achieved goals make up {{ ratio_achieved }}%"><em>{{ num_achieved }} of {{ num_total }}</em> Achieved</a><span style="width:{{ ratio_achieved }}%">{{ ratio_achieved }}</span></li>
    <li class="failed"><a title="Failed goals make up {{ ratio_failed }}%"><em>{{ num_failed }} of {{ num_total }}</em> Failed</a><span style="width:{{ ratio_failed }}%">{{ ratio_failed }}</span></li>
</ul>
```

If you look at the screenshot, you can see that we start with a display for the time in office, so let’s poke through that first:

```liquid
<!-- Liquid template doesn't have a `timesince` filter -->
<li class="tenure"><a><em>Since {{ info.tenure.start | date: "%B %-d, %Y" }}</em> In office</a></li>
```

Because Jekyll is a static site generator, you can not update the display of time automatically. This leaves us with an absolute display of the time **In Office**, which would otherwise have had a bar until the end of the term or the next election.

Instead, we just display the value `{{ info.tenure.start }}` (from `{{ site.data.info }}`, as you may remember) like so:

```liquid
<em>Since {{ info.tenure.start | date: "%B %-d, %Y" }}</em>
```

This takes the date value and formats it as a date according to the filter format defined by `date:`.

The remainder of the list items refer to our four goal types:

```liquid
<li class="unfinished"><a title="Unfinished goals make up {{ ratio_unfinished }}%"><em>{{ num_unfinished }} of {{ num_total }}</em> Unfinished</a><span style="width:{{ ratio_unfinished }}%">{{ ratio_unfinished }}%</span></li>
<li class="wip"><a title="Goals in progress make up {{ ratio_wip }}%"><em>{{ num_wip }} of {{ num_total }}</em> In progress</a><span style="width:{{ ratio_wip }}%">{{ ratio_wip }}</span></li>
<li class="achieved"><a title="Achieved goals make up {{ ratio_achieved }}%"><em>{{ num_achieved }} of {{ num_total }}</em> Achieved</a><span style="width:{{ ratio_achieved }}%">{{ ratio_achieved }}</span></li>
<li class="failed"><a title="Failed goals make up {{ ratio_failed }}%"><em>{{ num_failed }} of {{ num_total }}</em> Failed</a><span style="width:{{ ratio_failed }}%">{{ ratio_failed }}</span></li>
```

We’re just going to show the first one, in keeping with tradition:

```liquid
<li class="unfinished"><a title="Unfinished goals make up {{ ratio_unfinished }}%"><em>{{ num_unfinished }} of {{ num_total }}</em> Unfinished</a><span style="width:{{ ratio_unfinished }}%">{{ ratio_unfinished }}%</span></li>
```

Let’s reformat this for readability:

```liquid
<li class="unfinished">
    <a title="Unfinished goals make up {{ ratio_unfinished }}%">
        <em>{{ num_unfinished }} of {{ num_total }}</em> Unfinished
    </a>
    <span style="width:{{ ratio_unfinished }}%">{{ ratio_unfinished }}%</span>
</li>
```

The quirky HTML is there because of the CSS required to render the bar chart, basically.

The best way to explain this beyond what the code shows is to show an example result in regular HTML:

```html
<li class="unfinished"><a title="Unfinished goals make up 46.15384615384615%"><em>6 of 13</em> Unfinished</a><span style="width:46.15384615384615%">46.15384615384615%</span></li>
```

As aforementioned, all those decimals aren’t the intention and kind of a pain, especially for people with screenreaders. (Sorry screenreader people!)

This concludes the code for the Goal Overview; onto the Goal Table.

##### Goal Table #####

```liquid
<section role="region">
    {% for topic in site.topics %}
        <h2>{{ topic }}</h2>

        <table id="goal-table" class="u-full-width">
            <thead>
                <tr>
                    <th>Goal</th>
                    <th>Status</th>
                </tr>
            </thead>
            <tbody>
        {% for type in site.goal_types %}
            {% for goal in goals[type] %}{% if goal.topic == topic %}
                <tr class="{{ type }}">
                    <td>{{ goal.short }}</td>
                    <td>{{ type }}</td>
                </tr>
            {% endif %}{% endfor %}
        {% endfor %}
            </tbody>
        </table>
    {% endfor %}
</section>
```

1. `{% for topic in site.topics %}` loops through the topics you defined in `_config.yml`.
    - Here’s the relevant part:

        ```yaml
        goal_types:
            - unfinished
            - wip
            - achieved
            - failed
        topics:
            - Civil Liberties
            - Environment
            - Government
        ```
2. `{% for type in site.goal_types %}` loops through `goal_types`.
3. `{% for goal in goals[type] %}` loops through `site.data.goals[type]` where `type` is still the items in our `goal_types` list in `_config.yml`
    - We use `[type]`, from (2), instead of `.type`, because the latter would be interpreted as a "type" string instead of an iterating variable.
4. `{% if goal.topic == topic %}` if the topic of our goal (3) matches the our topic (1), fill the row with our goal data.

And that’s that!


[screenshot]: https://raw.githubusercontent.com/ndarville/goal-tracker/gh-pages/screenshot.png
[Liquid markup]: http://liquidmarkup.org
