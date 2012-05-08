---
layout: post
title: Connecting Highcharts and jqGrid with trigger and bind
tags:
- highcharts
- javascript
- jquery
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---

I have a page where I'm using [Highcharts](http://www.highcharts.com/products/highcharts) for a graph, and jqGrid for a grid; they're showing different views of similar data, but are sourced from two different calls to the backend. The graph is a summary, and the grid is detailed data, and it doesn't make much sense for that to come back in one call.

In my graph, I have several lines; Highcharts lets the user show/hide those lines manually. I want to make the data in the grid reflect what's been shown or hidden in the graph.

I first tried to bind the Highcharts series via Knockout, somehow, to what should be shown in the grid. Maybe it's possible, but I couldn't figure out how to get Knockout to bind to variables deep inside the Highcharts object structure.

But Javascript and jQuery have event listeners, right? Hopefully Highcharts is nice enough to use those, so I can get an event when the series visibility is changed.

I looked in the Highcharts source, and down in the Series' **setVisibility** method (more than 10000 lines deep), I found this:

    fireEvent(series, showOrHide);

(The "showOrHide" variable is always either "show" or "hide".)

That looks promising, but what does fireEvent do?

	fireEvent = function (el, type, eventArguments, defaultFunction) {
		var event = jQ.Event(type),
			detachedType = 'detached' + type;
		extend(event, eventArguments);

		// Prevent jQuery from triggering the object method that is named the
		// same as the event. For example, if the event is 'select', jQuery
		// attempts calling el.select and it goes into a loop.
		if (el[type]) {
			el[detachedType] = el[type];
			el[type] = null;
		}

		// trigger it
		jQ(el).trigger(event);

		// attach the method
		if (el[detachedType]) {
			el[type] = el[detachedType];
			el[detachedType] = null;
		}

		if (defaultFunction && !event.isDefaultPrevented()) {
			defaultFunction(event);
		}
	};

Most of that is necessary stuff that I don't care about, but the key line is this one:

    jQ(el).trigger(event);

That means that if I grab the Series object and wrap it with a jQuery call, I can call "bind" on it and be notified any time "trigger" is called. So I did that, for each series object:

    var listener = function() {
        MyGrid.updateGrid(MyGraph.visibleSeries());
    };
    for (var i=0; i < this.chart.series.length; i++) {
        $(this.chart.series[i]).bind("show", listener);
        $(this.chart.series[i]).bind("hide", listener);
    }

I created a function and saved it to a variable, and bound it to the "show" and "hide" events on each of my Series. I could have passed it in as an anonymous function, but I would have had to define it twice. I could have created it as an actual function on my object, but this way it's scoped right here and won't be accessible anywhere else.

On my MyGraph object, I need to be able to check which series are currently visible:

    function visibleSeries() {
        var selected = [];
        for (var i=0; i < MyGraph.chart.series.length; i++) {
            if (MyGraph.chart.series[i].visible) {
                selected.push(MyGraph.chart.series[i].name);
            }
        }
        return selected;
    }

That returns a list of series names, which I can then pass to the MyGrid object to update which lines it should display:

    var updateGrid = function(events) {
        var newData = {};
        newData.page = 1;
        newData.total = 1;
        newData.rows = [];
        for (var i=0; i < MyGrid.gridData.rows.length; i++) {
            if ($.inArray(MyGrid.gridData.rows[i].cell[1], events) != -1) {
                newData.rows.push(MyGrid.gridData.rows[i]);
            }
        }
        newData.records = newData.rows.length;
        MyGrid.updateData(JSON.stringify(newData));
    };

I've stored "gridData" in MyGrid, which is all the data for the grid that was returned from the server. It doesn't get updated here, which means I can look at the full dataset again each time the visible series changes, and update the grid accordingly.

Victory! Now, whenever the user hides a series in the graph, it disappears from the grid. When they add it back, it reappears in the grid. The grid also updates its display of total records available, and the number of pages, and always jumps back to the first page.
