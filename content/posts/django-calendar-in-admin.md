---
title: Calendar-like views in Django admin
date: "2023-02-19"
description: "How to get month-by-month stepped calendars in the Django admin"
tags: ["Technology", "Django"]
---

In my coding work for [SeekHealing](https://seekhealing.org/), we make extensive use of the Django Admin for our program management database. For a one-person coding team, Django has been invaluable for maintaining a functional, accessible system that evolves with the program's needs despite the limited number of hours I have to devote to the work.

We manage our calendar of program events in Django, with a model appropriately called `CalendarEvent`, which has a start-time and an end-time - and we've got data in this database going back to 2018. Events are planned out two months in advance, but the most active event management work is done on the events in handful of days before-and-after the current date. From a user-experience standpoint, how can we provide optimal navigation for calendar event management for the staff?

Listing the events in chronological order seems like a good first place to start. Doing so in ascending order, with the earliest events nearly 5 years ago, doesn't make any sense. Doing so in descending order might seem like a great idea, but because we plan events two months into the future, finding the set of events around the current date means hunting down the list across dozens of events further in the future.

From user interviews, what came out as providing the most intuitive UX was to make a customized template for the list view that shows up in a calendar form, month-by-month, with the present month selected by default, with navigation aiding the ability to move to the next and previous month.

I won't go into the calendar based layout in this post, suffice to say that the Python stdlib `calendar` module proved most helpful. But I do want to share about the month-by-month navigation.

My first attempt was to override the `ModelAdmin` class such that if you went to the `changelist_view` without specifying any `GET` parameters, it filled them in with the current month.

```python
# events/admin.py

from urllib.parse import urlencode

from django.contrib import admin
from django.http import HttpResponseRedirect
from events import models
from django.utils.timezone import localdate

class CalendarEventAdmin(admin.ModelAdmin):
    model = models.CalendarEvent
    change_list_template = "admin/events/calendarevent/calendarevent_list.html"
    ordering = ["start_time"]
    date_hierarchy = "start_time"

    def changelist_view(self, request, extra_context={}):
        if request.GET:
            return super().changelist_view(request, extra_context)
        date = localdate()
        params = ["month", "year"]
        field_keys = ["{}__{}".format(self.date_hierarchy, i) for i in params]
        field_values = [getattr(date, i) for i in params]
        query_params = dict(zip(field_keys, field_values))
        url = "{}?{}".format(request.path, urlencode(query_params))
        return HttpResponseRedirect(url)
```

... which initially seemed to work. Except with the Django admin `date_hierarchy` navigator, it listed breakdowns to the day-level. To get to the next or previous month, you have to step back out to the `year` only view and select the proper month - that's not ideal, but it would work... mostly. In December 2023, if you want to see January 2024, you can't, because when you step back out in the `date_hierarchy` to select the year, it would automatically redirect you back to the current month, as no `GET` querystring parameters would remain.

So I knew I had to modify the way `date_hierarchy` worked. And for that, I wrote my own template tag based on the `date_hierarchy` template tag:

```python
# events/templatetags/calendar_tags.py

import datetime

from dateutil import relativedelta
from django.contrib.admin.templatetags.base import InclusionAdminNode
from django.template import Library
from django.utils import format
from django.utils.text import capfirst
from django.utils.timezone import localdate

register = Library()

def month_explorer(cl):
    if cl.date_hierarchy:
        field_name = cl.date_hierarchy
        year_field = "%s__year" % field_name
        month_field = "%s__month" % field_name
        field_generic = "%s__" % field_name
        year_lookup = cl.params.get(year_field)
        month_lookup = cl.params.get(month_field)

        def link(filters):
            return cl.get_query_string(filters, [field_generic])

        if not (year_lookup and month_lookup):
            relative_to_date = localdate()
        else:
            relative_to_date = datetime.date(int(year_lookup), int(month_lookup), 1)
        next_date = relative_to_date + relativedelta.relativedelta(months=1)
        prev_date = relative_to_date - relativedelta.relativedelta(months=1)
        return dict(
            show=True,
            back=None,
            choices=[
                dict(
                    link=link({year_field: prev_date.year, month_field: prev_date.month}),
                    title="« " + capfirst(formats.date_format(prev_date, "YEAR_MONTH_FORMAT")),
                ),
                dict(
                    link=link({year_field: next_date.year, month_field: next_date.month}),
                    title=capfirst(formats.date_format(next_date, "YEAR_MONTH_FORMAT")) + " »",
                ),
            ],
        )


@register.tag(name="month_explorer")
def month_explorer_tag(parser, token):
    return InclusionAdminNode(
        parser,
        token,
        func=month_explorer,
        template_name="date_hierarchy.html",
        takes_context=False,
    )
```

```html
<!-- admin/events/calendarevent/calendarevent_list.html -->
{% load calendar_tags %}

{% block date_hierarchy %}{% month_explorer cl %}{% endblock %}
```

With these changes, the `changelist_view` in the Django admin defaults to loading the current month's events in the calendar, and in the date hierarchy navigator, it includes links to the previous and following month, making for easy navigation.
