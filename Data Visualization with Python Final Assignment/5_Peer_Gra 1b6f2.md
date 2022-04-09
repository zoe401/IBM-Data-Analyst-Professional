# 5_Peer_Graded_Assignment_Questions

[5_Peer_Graded_instruction.pdf](5_Peer_Gra%201b6f2/5_Peer_Graded_instruction.pdf)

[5_Peer_Graded_Assignment_Questions.py](5_Peer_Gra%201b6f2/5_Peer_Graded_Assignment_Questions.py)

[5_Peer_Graded_Assignment_Questions.ipynb](5_Peer_Gra%201b6f2/5_Peer_Graded_Assignment_Questions.ipynb)

Import required libraries

```python
import pandas as pdimport dash# import dash_html_components as htmlfrom dash import html# import dash_core_components as dccfrom dash import dccfrom dash.dependencies import Input, Output, Stateimport plotly.graph_objects as goimport plotly.express as pxfrom dash import no_update
```

Create a dash application

```python
app = dash.Dash(__name__)
```

REVIEW1: Clear the layout and do not display exception till callback gets executed

```python
app.config.suppress_callback_exceptions = True
```

Read the airline data into pandas dataframe

```python
airline_data =  pd.read_csv('https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DV0101EN-SkillsNetwork/Data%20Files/airline_data.csv',                            encoding = "ISO-8859-1",                            dtype={'Div1Airport': str, 'Div1TailNum': str,                                   'Div2Airport': str, 'Div2TailNum': str})
```

List of years

```python
year_list = [i for i in range(2005, 2021, 1)]
```

Compute graph data for creating yearly airline performance report  Function that takes airline data as input and create 5 dataframes based on the grouping condition to be used for plottling charts and grphs. Argument:  df: Filtered dataframe  Returns: Dataframes to create graph.

```python
def compute_data_choice_1(df):    # Cancellation Category Count    bar_data = df.groupby(['Month','CancellationCode'])['Flights'].sum().reset_index()    # Average flight time by reporting airline    line_data = df.groupby(['Month','Reporting_Airline'])['AirTime'].mean().reset_index()    # Diverted Airport Landings    div_data = df[df['DivAirportLandings'] != 0.0]    # Source state count    map_data = df.groupby(['OriginState'])['Flights'].sum().reset_index()    # Destination state count    tree_data = df.groupby(['DestState', 'Reporting_Airline'])['Flights'].sum().reset_index()    return bar_data, line_data, div_data, map_data, tree_data
```

Compute graph data for creating yearly airline delay report This function takes in airline data and selected year as an input and performs computation for creating charts and plots. Arguments: df: Input airline data.  Returns: Computed average dataframes for carrier delay, weather delay, NAS delay, security delay, and late aircraft delay.

```python
def compute_data_choice_2(df):    # Compute delay averages    avg_car = df.groupby(['Month','Reporting_Airline'])['CarrierDelay'].mean().reset_index()    avg_weather = df.groupby(['Month','Reporting_Airline'])['WeatherDelay'].mean().reset_index()    avg_NAS = df.groupby(['Month','Reporting_Airline'])['NASDelay'].mean().reset_index()    avg_sec = df.groupby(['Month','Reporting_Airline'])['SecurityDelay'].mean().reset_index()    avg_late = df.groupby(['Month','Reporting_Airline'])['LateAircraftDelay'].mean().reset_index()    return avg_car, avg_weather, avg_NAS, avg_sec, avg_late
```

Application layout

```python
app.layout = html.Div(children=[                                # TASK1: Add title to the dashboard                                # Enter your code below. Make sure you have correct formatting.                                html.H1('US Domestic Airline Flights Performance', style={'text-align':'center', 'color': '#503D36', 'font-size': 24}),                                # REVIEW2: Dropdown creation                                # Create an outer division                                html.Div([                                    # Add an division                                    html.Div([                                        # Create an division for adding dropdown helper text for report type                                        html.Div(                                            [                                            html.H2('Report Type:', style={'margin-right': '2em'}),                                            ]                                        ),                                        # TASK2: Add a dropdown                                        # Enter your code below. Make sure you have correct formatting.                                        dcc.Dropdown(id='input-type',                                                        options=[                                                                {'label': 'Yearly Airline Performance Report', 'value': 'OPT1'},                                                                {'label': 'Yearly Airline Delay Report', 'value': 'OPT2'}                                                                ],                                                        placeholder='Select a report type',                                                        style={'width': '80%', 'padding': '3px', 'font-size': '20px', 'text-align-last': 'center'})                                    # Place them next to each other using the division style                                    ], style={'display':'flex'}),                                    # Add next division                                    html.Div([                                        # Create an division for adding dropdown helper text for choosing year                                        html.Div(                                            [                                            html.H2('Choose Year:', style={'margin-right': '2em'})                                            ]                                        ),                                        dcc.Dropdown(id='input-year',                                                     # Update dropdown values using list comphrehension                                                     options=[{'label': i, 'value': i} for i in year_list],                                                     placeholder="Select a year",                                                     style={'width':'80%', 'padding':'3px', 'font-size': '20px', 'text-align-last' : 'center'}),                                            # Place them next to each other using the division style                                            ], style={'display': 'flex'}),                                ]),                                # Add Computed graphs                                # REVIEW3: Observe how we add an empty division and providing an id that will be updated during callback                                html.Div([ ], id='plot1'),                                html.Div([                                        html.Div([ ], id='plot2'),                                        html.Div([ ], id='plot3')                                ], style={'display': 'flex'}),                                # TASK3: Add a division with two empty divisions inside. See above disvision for example.                                # Enter your code below. Make sure you have correct formatting.                                html.Div([                                        html.Div([ ], id='plot4'),                                        html.Div([ ], id='plot5')                                ], style={'display': 'flex'}),                                ])
```

Callback function definition TASK4: Add 5 ouput components Enter your code below. Make sure you have correct formatting.

```python
@app.callback( [Output(component_id='plot1', component_property='children'),                Output(component_id='plot2', component_property='children'),                Output(component_id='plot3', component_property='children'),                Output(component_id='plot4', component_property='children'),                Output(component_id='plot5', component_property='children')],               [Input(component_id='input-type', component_property='value'),                Input(component_id='input-year', component_property='value')],               # REVIEW4: Holding output state till user enters all the form information. In this case, it will be chart type and year               [State("plot1", 'children'), State("plot2", "children"),                State("plot3", "children"), State("plot4", "children"),                State("plot5", "children")               ])# Add computation to callback function and return graphdef get_graph(chart, year, children1, children2, c3, c4, c5):        # Select data        df =  airline_data[airline_data['Year']==int(year)]        if chart == 'OPT1':            # Compute required information for creating graph from the data            bar_data, line_data, div_data, map_data, tree_data = compute_data_choice_1(df)            # Number of flights under different cancellation categories            bar_fig = px.bar(bar_data, x='Month', y='Flights', color='CancellationCode', title='Monthly Flight Cancellation')            # TASK5: Average flight time by reporting airline            # Enter your code below. Make sure you have correct formatting.            line_fig = px.line(line_data, x='Month', y='AirTime', color='Reporting_Airline', title='Average monthly flight time (minutes) by airline')            # Percentage of diverted airport landings per reporting airline            pie_fig = px.pie(div_data, values='Flights', names='Reporting_Airline', title='% of flights by reporting airline')            # REVIEW5: Number of flights flying from each state using choropleth            map_fig = px.choropleth(map_data,  # Input data                    locations='OriginState',                    color='Flights',                    hover_data=['OriginState', 'Flights'],                    locationmode = 'USA-states', # Set to plot as US States                    color_continuous_scale='GnBu',                    range_color=[0, map_data['Flights'].max()])            map_fig.update_layout(                    title_text = 'Number of flights from origin state',                    geo_scope='usa') # Plot only the USA instead of globe            # TASK6: Number of flights flying to each state from each reporting airline            # Enter your code below. Make sure you have correct formatting.            tree_fig = px.treemap(tree_data, path=['DestState', 'Reporting_Airline'],                                  values='Flights',                                  color='Flights',                                  color_continuous_scale='RdBu',                                  title='Flight count by airline to destination state')            # REVIEW6: Return dcc.Graph component to the empty division            return [dcc.Graph(figure=tree_fig),                    dcc.Graph(figure=pie_fig),                    dcc.Graph(figure=map_fig),                    dcc.Graph(figure=bar_fig),                    dcc.Graph(figure=line_fig)]        else:            # REVIEW7: This covers chart type 2 and we have completed this exercise under Flight Delay Time Statistics Dashboard section            # Compute required information for creating graph from the data            avg_car, avg_weather, avg_NAS, avg_sec, avg_late = compute_data_choice_2(df)            # Create graph            carrier_fig = px.line(avg_car, x='Month', y='CarrierDelay', color='Reporting_Airline', title='Average carrrier delay time (minutes) by airline')            weather_fig = px.line(avg_weather, x='Month', y='WeatherDelay', color='Reporting_Airline', title='Average weather delay time (minutes) by airline')            nas_fig = px.line(avg_NAS, x='Month', y='NASDelay', color='Reporting_Airline', title='Average NAS delay time (minutes) by airline')            sec_fig = px.line(avg_sec, x='Month', y='SecurityDelay', color='Reporting_Airline', title='Average security delay time (minutes) by airline')            late_fig = px.line(avg_late, x='Month', y='LateAircraftDelay', color='Reporting_Airline', title='Average late aircraft delay time (minutes) by airline')            return[dcc.Graph(figure=carrier_fig),                   dcc.Graph(figure=weather_fig),                   dcc.Graph(figure=nas_fig),                   dcc.Graph(figure=sec_fig),                   dcc.Graph(figure=late_fig)]
```

Run the app

```python
if __name__ == '__main__':    app.run_server()
```

```
Dash is running on http://127.0.0.1:8050/

 * Serving Flask app "__main__" (lazy loading)
 * Environment: production
[31m   WARNING: This is a development server. Do not use it in a production deployment.[0m
[2m   Use a production WSGI server instead.[0m
 * Debug mode: off

 * Running on http://127.0.0.1:8050/ (Press CTRL+C to quit)
127.0.0.1 - - [09/Apr/2022 15:24:19] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [09/Apr/2022 15:24:19] "GET /_dash-component-suites/dash/deps/polyfill@7.v2_3_1m1649379812.12.1.min.js HTTP/1.1" 200 -
127.0.0.1 - - [09/Apr/2022 15:24:19] "GET /_dash-component-suites/dash/deps/react@16.v2_3_1m1649379812.14.0.min.js HTTP/1.1" 200 -
127.0.0.1 - - [09/Apr/2022 15:24:19] "GET /_dash-component-suites/dash/deps/react-dom@16.v2_3_1m1649379812.14.0.min.js HTTP/1.1" 200 -
127.0.0.1 - - [09/Apr/2022 15:24:19] "GET /_dash-component-suites/dash/deps/prop-types@15.v2_3_1m1649379812.7.2.min.js HTTP/1.1" 200 -
127.0.0.1 - - [09/Apr/2022 15:24:19] "GET /_dash-component-suites/dash/dash-renderer/build/dash_renderer.v2_3_1m1649379812.min.js HTTP/1.1" 200 -
127.0.0.1 - - [09/Apr/2022 15:24:19] "GET /_dash-component-suites/dash/dcc/dash_core_components.v2_3_0m1649379812.js HTTP/1.1" 200 -
127.0.0.1 - - [09/Apr/2022 15:24:19] "GET /_dash-component-suites/dash/dcc/dash_core_components-shared.v2_3_0m1649379812.js HTTP/1.1" 200 -
127.0.0.1 - - [09/Apr/2022 15:24:19] "GET /_dash-component-suites/dash/html/dash_html_components.v2_0_2m1649379812.min.js HTTP/1.1" 200 -
127.0.0.1 - - [09/Apr/2022 15:24:19] "GET /_dash-component-suites/dash/dash_table/bundle.v5_1_1m1649379812.js HTTP/1.1" 200 -
127.0.0.1 - - [09/Apr/2022 15:24:19] "GET /_dash-layout HTTP/1.1" 200 -
127.0.0.1 - - [09/Apr/2022 15:24:19] "GET /_dash-dependencies HTTP/1.1" 200 -
127.0.0.1 - - [09/Apr/2022 15:24:19] "GET /_dash-component-suites/dash/dcc/async-dropdown.js HTTP/1.1" 200 -
127.0.0.1 - - [09/Apr/2022 15:24:19] "GET /_favicon.ico?v=2.3.1 HTTP/1.1" 200 -

Exception on /_dash-update-component [POST]
Traceback (most recent call last):
  File "D:\Anaconda\lib\site-packages\flask\app.py", line 2447, in wsgi_app
    response = self.full_dispatch_request()
  File "D:\Anaconda\lib\site-packages\flask\app.py", line 1952, in full_dispatch_request
    rv = self.handle_user_exception(e)
  File "D:\Anaconda\lib\site-packages\flask\app.py", line 1821, in handle_user_exception
    reraise(exc_type, exc_value, tb)
  File "D:\Anaconda\lib\site-packages\flask\_compat.py", line 39, in reraise
    raise value
  File "D:\Anaconda\lib\site-packages\flask\app.py", line 1950, in full_dispatch_request
    rv = self.dispatch_request()
  File "D:\Anaconda\lib\site-packages\flask\app.py", line 1936, in dispatch_request
    return self.view_functions[rule.endpoint](**req.view_args)
  File "D:\Anaconda\lib\site-packages\dash\dash.py", line 1383, in dispatch
    response.set_data(func(*args, outputs_list=outputs_list))
  File "D:\Anaconda\lib\site-packages\dash\_callback.py", line 151, in add_context
    output_value = func(*func_args, **func_kwargs)  # %% callback invoked %%
  File "C:\Users\XIXI\AppData\Local\Temp/ipykernel_50772/1124008570.py", line 17, in get_graph
    df =  airline_data[airline_data['Year']==int(year)]
TypeError: int() argument must be a string, a bytes-like object or a number, not 'NoneType'

127.0.0.1 - - [09/Apr/2022 15:24:19] "POST /_dash-update-component HTTP/1.1" 500 -
127.0.0.1 - - [09/Apr/2022 15:24:22] "POST /_dash-update-component HTTP/1.1" 200 -
127.0.0.1 - - [09/Apr/2022 15:24:23] "POST /_dash-update-component HTTP/1.1" 200 -
127.0.0.1 - - [09/Apr/2022 15:24:23] "GET /_dash-component-suites/dash/dcc/async-graph.js HTTP/1.1" 200 -
127.0.0.1 - - [09/Apr/2022 15:24:23] "GET /_dash-component-suites/dash/dcc/async-plotlyjs.js HTTP/1.1" 200 -
```

## Screenshot of the dashboard

![Web capture_9-4-2022_1515_moshou107-8050.theiadocker-1-labs-prod-theiak8s-4-tor01.proxy.cognitivecla.jpeg](5_Peer_Gra%201b6f2/Web_capture_9-4-2022_1515_moshou107-8050.theiadocker-1-labs-prod-theiak8s-4-tor01.proxy.cognitivecla.jpeg)

![Web capture_9-4-2022_15032_moshou107-8050.theiadocker-1-labs-prod-theiak8s-4-tor01.proxy.cognitivecl.jpeg](5_Peer_Gra%201b6f2/Web_capture_9-4-2022_15032_moshou107-8050.theiadocker-1-labs-prod-theiak8s-4-tor01.proxy.cognitivecl.jpeg)