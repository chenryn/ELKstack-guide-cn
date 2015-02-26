A Kibana *dashboard* displays a set of saved visualizations in groups that you can arrange freely. You can save a dashboard to share or reload at a later time.

Sample dashboard. ![Example dashboard](http://www.elasticsearch.org/guide/en/kibana/current/images/NYCTA-Dashboard.jpg)

## getting started

You need at least one saved ![visualization](http://www.elasticsearch.org/guide/en/kibana/current/visualize.html) to use a dashboard.

### building a new dashboard

The first time you click the **Dashboard** tab, Kibana displays an empty dashboard.

![New Dashboard screen](http://www.elasticsearch.org/guide/en/kibana/current/images/NewDashboard.jpg)

Build your dashboard by adding visualizations.

### adding visualizations to a dashboard

To add a visualization to the dashboard, click the **Add Visualization** ![Plus](http://www.elasticsearch.org/guide/en/kibana/current/images/AddVis.png) button in the toolbar panel. Select a saved visualization from the list. You can filter the list of visualizations by typing a filter string into the **Visualization Filter** field.

The visualization you select appears in a *container* on your dashboard.

> If you see a message about the container’s height or width being too small, [resize the container](http://www.elasticsearch.org/guide/en/kibana/current/dashboard.html#resizing-containers).

### saving dashboards

To save the dashboard, click the **Save Dashboard** button in the toolbar panel, enter a name for the dashboard in the **Save As** field, and click the **Save** button.

### loading a saved dashboard

Click the **Load Saved Dashboard** button to display a list of existing dashboards. The saved dashboard selector includes a text field to filter by dashboard name and a link to the **Object Editor** for managing your saved dashboards. You can also access the **Object Editor** by clicking **Settings > Edit Saved Objects**.

### sharing dashboards

You can share dashboards with other users. You can share a direct link to the Kibana dashboard or embed the dashboard in your Web page.

> A user must have Kibana access in order to view embedded dashboards.

Click the **Share** button to display HTML code to embed the dashboard in another Web page, along with a direct link to the dashboard. Click the copy button ![Copy](http://www.elasticsearch.org/guide/en/kibana/current/images/Clipboard.png) next to either option to copy the code or the link to your clipboard.

### embedding dashboards

To embed a dashboard, copy the embed code from the *Share* display into your external web application.

## customizing dashboard elements

The visualizations in your dashboard are stored in resizable *containers* that you can arrange on the dashboard. This section discusses customizing these containers.

### moving containers

Click and hold a container’s header to move the container around the dashboard. Other containers will shift as needed to make room for the moving container. Release the mouse button to confirm the container’s new location.

### resizing containers

Move the cursor to the bottom right corner of the container until the cursor changes to point at the corner. After the cursor changes, click and drag the corner of the container to change the container’s size. Release the mouse button to confirm the new container size.

### removing containers

Click the x icon at the top right corner of a container to remove that container from the dashboard. Removing a container from a dashboard does not delete the saved visualization in that container.

### viewing detailed information

To display the raw data behind the visualization, click the bar at the bottom of the container. Tabs with detailed information about the raw data replace the visualization, as in this example:

Table. A representation of the underlying data, presented as a paginated data grid. You can sort the items in the table by clicking on the table headers at the top of each column. ![images/NYCTA-Table.jpg](http://www.elasticsearch.org/guide/en/kibana/current/images/NYCTA-Table.jpg)

Request. The raw request used to query the server, presented in JSON format. ![images/NYCTA-Request.jpg](http://www.elasticsearch.org/guide/en/kibana/current/images/NYCTA-Request.jpg)

Response. The raw response from the server, presented in JSON format. ![images/NYCTA-Response.jpg](http://www.elasticsearch.org/guide/en/kibana/current/images/NYCTA-Response.jpg)

Statistics. A summary of the statistics related to the request and the response, presented as a data grid. The data grid includes the query duration, the request duration, the total number of records found on the server, and the index pattern used to make the query. ![images/NYCTA-Statistics.jpg](http://www.elasticsearch.org/guide/en/kibana/current/images/NYCTA-Statistics.jpg)

## changing the visualization

Click the *Edit* button ![Pencil button](http://www.elasticsearch.org/guide/en/kibana/current/images/EditVis.png) at the top right of a container to open the visualization in the [Visualize](./visualize.md) page.
