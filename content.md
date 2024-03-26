# Integrating Charts and Graphs with ChartKick

## Introduction
In this lesson, we're going to explore how to enrich your application with interactive and visually appealing charts and graphs using the [ChartKick gem](https://github.com/ankane/chartkick). Additionally, we'll delve into the importance of the [groupdate gem](https://github.com/ankane/groupdate) for grouping data by time intervals and provide SQL examples to understand what happens under the hood.

## Step 1: Adding Necessary Gems
To get started, we need to add both [chartkick](https://github.com/ankane/chartkick) and [groupdate](https://github.com/ankane/groupdate) gems to our project. While ChartKick enables us to create charts effortlessly, Groupdate enhances our querying capabilities, particularly for time-based data aggregation.

In your Gemfile, add:

```ruby
gem 'chartkick'
gem 'groupdate'
```
Then, run bundle install to install these gems.

## Step 2: Creating Your First Chart
ChartKick simplifies chart creation, allowing you to generate various charts with minimal code. For instance, to create a line chart displaying sales per day, you can use:

```erb
<%= line_chart Sale.group_by_day(:created_at).count %>
```

This code snippet generates a line chart by grouping sales based on the day they were created and counts the total number of sales for each day.



### Understanding the Query
The magic behind `group_by_day` comes from the groupdate gem, which extends Active Record's grouping capabilities. Here's what the SQL query looks like for the above Ruby code:

```sql
SELECT DATE(created_at) AS created_at, COUNT(*) AS count 
FROM sales 
GROUP BY DATE(created_at);
```
This SQL query groups records by the day they were created and counts the number of sales for each group.

## Step 3: Filtering and Aggregating Data
To visualize specific data, such as sales over the last month, you can filter and aggregate data using Active Record's querying methods. A good approach to querying data for charts involves three main steps:

- **Filter**: Use the `where` method to narrow down the records you want to include in the chart.
- **Group**: Use the `group` method to define how your data should be grouped in the chart (e.g., by day, month, or category). 
- **Calculate**: Use calculation methods like `sum`, `count`, or `average` to aggregate the data within each group. 


Here's an example:

```erb
<%= line_chart Sale.where(created_at: 1.month.ago..Time.now).group_by_day(:created_at).sum(:amount) %>
```

### SQL Behind the Scenes
The corresponding SQL query for this Ruby code would be:

```sql
SELECT DATE(created_at) AS created_at, SUM(amount) AS sum_amount 
FROM sales 
WHERE created_at BETWEEN '2023-02-01' AND '2023-03-01' 
GROUP BY DATE(created_at);
```
This query filters sales records created in the last month, groups them by day, and sums up the sales amount for each day.

<aside>
You can also use [explain](https://guides.rubyonrails.org/active_record_querying.html#running-explain) on any active record query to get more details on the raw sql.
</aside>

## Step 4: Loading Charts Asynchronously with Rails Helpers
For a cleaner and more "Rails-way" approach to loading charts asynchronously, you can utilize Rails path helpers in combination with ChartKick. This method involves creating a dedicated action in your controller for chart data and then referencing this action with a helper method in your view. Here’s how you can do it:

First, define a new action in your controller that will serve the chart data as JSON. This action does not need a corresponding view since it will only be returning JSON data.

```ruby
# app/controllers/sales_controller.rb
def chart_data
  render json: Sale.group_by_day(:created_at).count
end
```
Ensure that you have a route set up for the new chart data action. This route will be used to generate the path helper.

```ruby
# config/routes.rb
get 'sales/chart_data', to: 'sales#chart_data', as: 'sales_chart_data'
```

Use the Helper Method in Your View
Now, instead of embedding the chart data directly in your view or fetching it with JavaScript, you can use the helper method provided by Rails to reference the chart data action. ChartKick will handle the asynchronous loading of the chart data from the specified endpoint.

```erb
<%= line_chart sales_chart_data_path %>
```
The `sales_chart_data_path` helper method generates the URL for the chart data action, instructing ChartKick to fetch the data from that URL.

### Advantages
This method offers several benefits:

- **Simplicity**: There’s no need to write custom JavaScript or use additional frameworks to fetch and display chart data.
- **Maintainability**: Keeping the chart data logic within a Rails controller action adheres to MVC principles and keeps your code organized.
- **Performance**: Loading chart data asynchronously can improve page load times, as the chart data is fetched in the background after the initial page load.

## Conclusion
By integrating ChartKick and Groupdate into your application, you can effortlessly create dynamic and informative charts. These visualizations not only improve the user experience but also provide valuable insights into your data. Experiment with different chart types and data sets to fully leverage the potential of these powerful gems in your projects.

## Resources
- [ChartKick](https://github.com/ankane/chartkick)
- [Groupdate](https://github.com/ankane/groupdate)
- [Example Project](https://github.com/DPI-WE/chartkick-example)
  - [Pull Request](https://github.com/DPI-WE/chartkick-example/pull/1)
- [Lecture Video](https://youtu.be/fBxbMVnJHDY)
- [Active Record Querying Guide](https://guides.rubyonrails.org/active_record_querying.html)
  - [grouping](https://guides.rubyonrails.org/active_record_querying.html#grouping)
  - [calculations](https://guides.rubyonrails.org/active_record_querying.html#calculations)
