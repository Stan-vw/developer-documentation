---
category: DevelopInDepth
previous: archiving
next: reports
---
# Archive data

**Archive data** is created during the [archiving process](/guides/archiving) by aggregating **log data**.

Piwik aggregates and persists two types of archive data:

- **metrics**, which are single numeric values
- **reports**, which are two-dimensional array of values

Reports will normally contain metric values, but they can also contain other data (either additionally or in lieu of metric values).

Reports and metrics are defined by plugins, letting any plugin extend the data analyzed by Piwik. However, there are several metrics, called **core metrics**, that are defined by Piwik Core.

## Subset parameters

Reports and metrics provide analytics data about a set of things. This set is defined by three constraints:

- a website ID
- a period
- a segment

The **website ID** selects visits that were tracked for a specific website. This ID is specified in all HTTP requests with the `idSite` query parameter.

The **period** selects visits that were tracked within a specific date range. The period is specified in all HTTP requests with the `date` and `period` query parameters.

The **segment** selects visits based on a boolean expression that uses visit properties. It is specified in all HTTP requests by the `segment` query parameter and can be used to select almost any conceivable subset of visit.

Analytics parameters are stored in reports as metadata, that means they are stored as [DataTable](/api-reference/Piwik/DataTable) metadata.

## Metrics

### Core metrics

**Core metrics** are metrics that are not defined by plugins but by **Piwik Core**.

New reports that analyze visits, action types or conversions should contain these metrics.

#### Visit metrics

Core metrics for a set of visits:

Name             | Metric ID             | Description
-----------------|-----------------------|------------
Visits           | `nb_visits`           | Number of tracked visits. <br> A visit is series of events each of which happened no more than 30 minutes apart.
Unique visitors  | `nb_uniq_visitors`    | The number of unique sources of visits. <br> A visit source is an entity that causes a visit to be tracked.
Actions          | `nb_actions`          | The number of tracked actions. <br> An action is an event tracked by Piwik.
Max Actions      | `max_actions`         | The maximum number of actions that occurred in one visit.
Sum Visit Length | `sum_visit_length`    | The sum of each visit's elapsed time.
Bounce Count     | `bounce_count`        | The number of visits that consisted of only one action.
Converted Visits | `nb_visits_converted` | The number of visits that caused at least one conversion. <br> Includes conversions for every goal of a site.
Conversions      | `nb_conversions`      | The number of conversions tracked for this set of visits. <br> Includes conversions for every goal of a site.
Revenue          | `revenue`             | The total revenue generated by these visits. <br> Includes revenue for every goal of a site plus its ecommerce revenue.

#### Action metrics

Core metrics for a single action type:

Name                      | Metric ID                      | Description
--------------------------|--------------------------------|------------
Hits                      | `nb_hits`                      | The number times this action was ever done.
Sum Time Spent            | `sum_time_spent`               | The total amount of time the user spent doing this action.
Sum Page Generation Time  | `sum_time_generation`          | The total amount of time a server spent serving this action.
Hits With Generation Time | `nb_hits_with_time_generation` | The number of hits that included generation time information.
Min Page Generation Time  | `min_time_generation`          | The minimum amount of time a server spent serving this action.
Max Page Generation Time  | `max_time_generation`          | The maximum amount of time a server spent serving this action.
Unique Exit Visitors      | `exit_nb_uniq_visitors`        | The number of unique visitors that ever exited a site after this action.
Exit Visits               | `exit_nb_visits`               | The total number of visits that ended with this action.
Unique Entry Visitors     | `entry_nb_uniq_visitors`       | The total number of unique visitors that started a visit with this action.
Entry Visits              | `entry_nb_visits`              | The total number of visits that started with this action.
Entry Actions             | `entry_nb_actions`             | <!-- TODO: isn't this the same as entry visits? -->
Entry Sum Visit Length    | `entry_sum_visit_length`       | The sum of each entry visit's elapsed time.
Entry Bounce Count        | `entry_bounce_count`           | The number of visits that consisted of this action and no other.
Hits From Search          | `nb_hits_following_search`     | The number of times this action was done after a site search.

#### E-commerce metrics

Core metrics for the set of ecommerce conversions (either all orders or all abandoned carts) recorded for a set of visits:

Name                 | Metric ID          | Description
---------------------|--------------------|------------
Revenue Subtotal     | `revenue_subtotal` | The total cost of every item that was a part of these orders or abandoned carts.
Revenue Tax          | `revenue_tax`      | The total tax amount applied to these orders/abandoned carts.
Revenue Shipping     | `revenue_shipping` | The total amount of shipping applied to these orders/abandoned carts.
Revenue Discount     | `revenue_discount` | The total amount of discounts applied to these orders/abandoned carts.
Ecommerce Item Count | `items`            | The total number of items in these orders/abandoned carts.

#### Goal metrics

Core metrics for a set of visits and one goal of a site:

Name             | Metric ID                      | Description
-----------------|--------------------------------|------------
Goal Conversions | `goal_<idGoal>_nb_conversions` | The conversions tracked for a specific goal and this set of visits.
Goal Revenue     | `goal_<idGoal>_revenue`        | The total revenue generated by the conversions for a specific goal.

_Note: `<idGoal>` should be replaced with the ID of a goal._

Goal specific metrics are stored in the database in the `goals` column of serialized reports. The column contains a PHP array mapping goal IDs with arrays of goal specific metric values. These values are set as normal column values with the metric names described above by the [AddColumnsProcessedMetricsGoal](/api-reference/Piwik/DataTable/Filter/AddColumnsProcessedMetricsGoal) DataTable filter.

### Processed metrics

In the interests of [archiving](/guides/archiving) and database size efficiency, some metrics are not stored in database. They are instead calculated when needed using other metrics. These metrics are called **processed metrics**.

Below is the list of processed metrics that are calculated using *core metrics*. New reports that analyze visits, action types or conversions should have these metrics added when possible.

_Note: Some processed metrics will appear multiple times in the lists below. These metrics have different meanings based on the reports they are in._

Processed metrics for a set of visits:

Name                 | Metric ID              | Description
---------------------|------------------------|------------
Conversion Rate      | `conversion_rate`      | The percent of visits that had at least one conversion.
Actions Per Visit    | `nb_actions_per_visit` | The average number of actions for a single visit.
Average Time On Site | `avg_time_on_site`     | The average number of time spent per visit in seconds.
Bounce Rate          | `bounce_rate`          | The percent of visits that resulted in a bounce.

Processed metrics for a single action type:

Name                                         | Metric ID             | Description
---------------------------------------------|-----------------------|------------
Average Generation Time                      | `avg_time_generation` | The average amount of time it took for a server to serve this action.
Average Number of Search Result Pages Viewed | `nb_pages_per_search` | The average number of search result pages viewed after a site search. <br> Only valid for site search keywords and site search categories.
Average Time On Page                         | `avg_time_on_page`    | The average amount of time users spent doing this action.
Entry Bounce Rate                            | `bounce_rate`         | The percent of all visits that consisted of this action and no other.
Exit Rate                                    | `exit_rate`           | The percent of all visits that ended with this action.

Processed metrics for the set of ecommerce orders recorded for a set of visits:

Name                  | Metric ID           | Description
----------------------|---------------------|------------
Average Order Revenue | `avg_order_revenue` | The average revenue of each order.

Processed metrics for the set of ecommerce items in a set of orders or abandoned carts:

Name                    | Metric ID         | Description
------------------------|-------------------|------------
Average Price           | `avg_price`       | The average price of each item.
Average Quantity        | `avg_quantity`    | The average number of each item in an order/abandoned cart.
Product Conversion Rate | `conversion_rate` | The percent of orders/abandoned carts that include this item.

The following is a list of processed metrics that are also specific to one goal of one site:

Name                      | Metric ID                         | Description
--------------------------|-----------------------------------|------------
Average Revenue per Visit | `goal_<idGoal>_revenue_per_visit` | The average amount of revenue generated per visit for this goal.

### Naming convention

Metrics calculated and persisted by plugins **must** be named with the following format: `PluginName_metricName`. For example: `MyPlugin_myFancyMetric`.

Core metrics have special names and do not follow this convention.

## Reports

Reports are stored in memory using the [`DataTable`](/api-reference/Piwik/DataTable) class. A `DataTable` is a 2-dimension array composed of rows and columns.

Each row contains metrics that relate to a set of visits, actions, conversions… That set is defined and described by a special **label** column. How the column describes the set depends entirely upon the specific report. For example in the `UserSettings.getBrowser` report, a row with the label *Firefox* would hold metrics for visits that used the Firefox browser.

Some reports like `VisitsSummary.get` will not have a label column: they have only one row that refers to the entire set of entities.

### Report metadata

In addition to metrics, each row can also contain **metadata**. This metadata will usually assist the label column in describing the set of things the row represents.

Some metadata have special meaning in Piwik, for example:

- `logo`: the value can be a path to an image that will be shown alongside each row in the UI
- `url`: the value can be a URL to which the row will link in the UI

### Subtables

Reports can be hierarchical: each row can be attached to another DataTable. Tables that are attached to rows are called **subtables**.

Subtables provide further analytics for the set of visits that a row represents. For example, the `Referrers.getSearchEngines` report has one row per search engine. Each row has a subtable that describes keywords used with that search engine. Here is a schematic representation:

```
Search Engine  Keyword (subtable)  Visitors
--------------|-------------------|----------
Google                            | 207
--------------|------------------------------
              | piwik             | 11
              | libre analytics   | 6
              | ...
---------------------------------------------
Duck Duck Go                      | 121
--------------|------------------------------
              | ...
```

### Naming convention

Reports **must** be named like metrics are: `PluginName_reportName`. For example: `MyPlugin_myFancyReport`.