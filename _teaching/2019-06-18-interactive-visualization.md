---
layout: post
date:   2019-06-18 16:41:16
inline: false
title: Interactive Data Visualization with Dash
url: 
authors: [Durairaj, Janani and Akdel, Mehmet]
---
Authors: Janani Durairaj and Mehmet Akdel

Required python packages for this workshop:

```text
pandas
numpy
scipy==1.3.0
plotly
dash==0.40.0
dash_bio==0.1.1
statsmodels==0.10.0
```

In this workshop we will explore the use of the Python [Dash](https://dash.plot.ly/) framework to build [interactive web applications](https://dash-gallery.plotly.host/Portal/). 
For the interactive graphs we will use the [Plotly](https://plot.ly/d3-js-for-python-and-pandas-charts/) graphing library that nicely integrates with Dash. 
Two examples of Dash used in bioinformatics applications: [inDelphi](https://indelphi.giffordlab.mit.edu/single), [Drug Discovery](https://dash-gallery.plotly.host/dash-drug-discovery/).

Here are the libraries you will need to import:
```python
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output
import dash_bio
import plotly.graph_objs as go
import plotly.offline as py
import pandas as pd
import statsmodels.api as sm
import numpy as np
```

We will be using yeast cell cycle gene expression data from [Spellman et al.](https://doi.org/10.1091/mbc.9.12.3273) to 
incrementally build up a web page to visualize different aspects of the data. 
The rows of the Pandas dataframe below represent genes, and the columns represent time points (no replicates). 
The time points are minutes after cell cycle synchronisation.

```python
expr = pd.read_csv('http://www.exploredata.net/ftp/Spellman.csv', index_col=0)
```

We first look at the gene expression values for one particular gene (selected for no particular reason).

```python
gene_id = 'YAL001C'
expr_gene = expr.loc[gene_id]
```

With the series of expression values we can make a scatter-plot using plotly's `go.Scatter`:
```python
scatter_plot = go.Scatter(
        x = expr_gene.index,
        y = expr_gene,
        mode = 'markers',
        name = gene_id
)
```

This code would work standalone in a Jupyter notebook using just plotly with `py.iplot([scatter_plot])` but here we'll make a Dash app from it and run it as a script. 
The Dash app object is created with:

```python
app = dash.Dash()
app.css.append_css({
    'external_url': 'https://cdnjs.cloudflare.com/ajax/libs/skeleton/2.0.4/skeleton.css'
})
```
We'll talk about the CSS part later, but Dash also takes care of generating the actual HTML for the web page. 
It is common to use content division or div blocks to organise components on a web page. In HTML that would look like this:
```html
<div>
    some text
</div>
```
whereas in Dash you would write it as:
```python
app.layout = html.Div(["some text"])
```

To add the scatter-plot figure to the page we use the Dash core component `Graph` and provide the scatter-plot as data.

```python
app.layout = html.Div([
    dcc.Graph(
        id='expression_plot',
        figure={
            'data':[scatter_plot]
        })
])
```

Now we are ready to show the page. For that, Dash provides a built-in web server that can simply be started by calling the `run_server()` method. 
This will bring up a web server running on your local computer at port 8050. You can view the page by pointing your browser to 
http://127.0.0.1:8050/.

```python
if __name__ == "__main__":
  app.run_server()
```
Add the above code to a script (e.g. **first_dash_script.py**) and run using `python first_dash_script.py`. 
Check the page at http://127.0.0.1:8050/ to see the expression plot of our selected gene. 
You may notice that even this simple plot already has interactive capabilities for hovering over the points, zooming, selecting etc. 
(More interactive options can be found in the top right corner of the plot). 

But the real power of Dash is in the way it can alter the plots being displayed based on input from other sources. 
We can see this by adding a drop-down box to select the gene we're interested in viewing (instead of hard-coding one example) 
and having the scatter-plot change to the expression values of that gene.

Stop the Dash app and change the `app.layout` definition with the code below, then restart and see the changes in your browser.

```python
app.layout = html.Div([
    dcc.Dropdown(
                id='gene_selector',
                options=[{'label': gene, 'value': gene} for gene in expr.index],
                value=''),
    dcc.Graph(
        id='expression_plot',
        figure={
            'data':[scatter_plot],
            'layout': {
                'title': gene_id
            }
        })
])
```

That was easy right?

Of course, it can be a bit annoying to have to stop and rerun the code every time you make a change, so instead you can 
have the Dash app dynamically check for updates of the Python code if you set:
```python
if __name__ == "__main__":
  app.run_server(debug=True)
```
Then you don't have to restart the server each time you change the code, you just have to refresh the browser page.

But... nothing happens to the graph if you select a different gene?!

This is because we're still using the same scatter-plot, the one calculated for that single gene we hard-coded in the beginning. 
What we need instead is for Dash to recalculate the scatter-plot based on the gene selected. For that we need to add update functionality in the form of a callback function that triggers when the drop-down menu is used.

[![Callback](https://miro.medium.com/max/260/1*c8yU2Jjm3QjjNWqAW15WkQ.jpeg)](https://medium.com/@alroth/understanding-callbacks-for-people-who-want-to-punch-javascript-in-the-face-76cb856e8469)

The callback function starts with a decorator `@app.callback` that defines the inputs and the outputs - this function is triggered when one of the inputs is changed and updates the corresponding outputs. In our case, the input is the selected gene id, and the output will be the scatter-plot.

```python
@app.callback(
    Output('expression_plot', 'figure'),
    [Input('gene_selector', 'value')])
def update_expression_plot(gene_id):
    if not gene_id:
        return None
    else:
        expr_gene = expr.loc[gene_id]
        scatter_plot = go.Scatter(
            x=expr_gene.index,
            y=expr_gene,
            mode='markers',
            name=gene_id)
        return {'data': [scatter_plot]}
```

This function looks for changes in the `value` field of the component with id `gene_selector`. When a change is detected, 
it makes the scatter-plot and puts it as the `figure` field of the component with id `expression_plot`.

Now we can remove the scatter-plot from the layout definition, as it will be calculated and rendered by the function above:

```python
app.layout = html.Div([
    dcc.Dropdown(
                id='gene_selector',
                options=[{'label': gene, 'value': gene} for gene in expr.index],
                value=expr.index[0]),
    dcc.Graph(id='expression_plot')
])
```

Congratulations on creating and running an interactive web application!

You are not restricted to having a single input and output in the callback function or even a single callback function in your Dash app; the final page can be a mix of interlinked plots changing according to inputs from various [drop-down boxes, sliders, checkboxes etc.](https://dash.plot.ly/dash-core-components) For instance, let's also add a regression line to the scatter-plot and have a slider that determines the smoothing fraction of the regression line.

We can also take the time now to add some other elements to our layout, using a simple CSS framework called [Skeleton](http://getskeleton.com/) (already loaded above) to position our components. To put components on the same row you would put them all in a `Div` with the `className` as 'row'. The web-page is divided into 12 columns (which change in size accordingly when you use a phone instead of your laptop for instance). You can make a component take up 4 columns in this layout by putting it in a `Div` with the `className` as "four columns". 
The "container" `className` just puts everything into a Skeleton container with automatically calculated margins according to your screen size. 

```python
app.layout = html.Div([
    html.Div([html.H1('Yeast expression browser', style={"text-align": "center"}),
              html.H3('Parameters', style={"text-align": "center"})],
             className="row"),
    html.Div([
        html.Div([
            html.Div([html.Div('Smoothing fraction:', id='smoothing_fraction'), 
              dcc.Slider(
                  id='fraction_slider',
                  min=0,
                  max=1,
                  step=0.05,
                  marks={0: '0', 1: '1.0'},
                  value=0.5 # default smoothing fraction
                  )], className="six columns"),
            html.Div([dcc.Dropdown(
                id='gene_selector',
                options=[{'label': gene, 'value': gene} for gene in expr.index],
                value=expr.index[0],
                multi=False)], className="six columns")],
            className="container"),

        html.Div([html.Div([html.Br(), html.Br(),
                            html.H3("Expression plot", style={"text-align": "center"}),
                            dcc.Graph(id='expression_plot')])], className='row')])])
```

We add the `fraction_slider` and `smoothing_fraction` as another input and output of our previous callback function:  

```python
@app.callback(
    [Output("expression_plot", "figure"),
     Output("smoothing_fraction", "children")],
    [Input("gene_selector", "value"),
     Input("fraction_slider", "value")])
def update_expression_plot(gene_id, frac):
    """
     Updates the expression plot in the web-app by using gene selector and smoothing fraction input components.
    """
    if not gene_id or not smoothing_fraction: 
    # If any of the inputs are missing then no figure is returned
        return None, f"Smoothing fraction ({0})"
    else:
        expr_gene = expr.loc[gene_id]
        scatter_plot = go.Scatter(x=expr_gene.index,
                           y=expr_gene,
                           mode='markers',
                           name=gene_id)
        smooth_line = sm.nonparametric.lowess(expr_gene,
                                              pd.to_numeric(expr_gene.index),
                                              frac=frac)

        regression_line = go.Scatter(x=smooth_line[:, 0],
                            y=smooth_line[:, 1],
                            mode='lines',
                            line={'shape': 'spline', 'smoothing': 1.3},
                            name='Lowess')

        return {'data': [scatter_plot, regression_line]},  f"Smoothing fraction ({frac})"
```

Try that out!

Finally, while Dash and Plotly provide many basic plots such as [line charts, bar charts, seaborn style charts, contour plots etc.](https://plot.ly/python/), a new and very interesting development is [Dash-Bio](https://dash.plot.ly/dash-bio), a library that provides Dash components for common bioinformatics visualization tasks. You can play around with some examples on their [demo server](https://dash-bio.plotly.host/Portal/) but below we use the `Clustergram` component to add a heatmap with associated dendogram to our expression browser. This is done by updating the layout and adding another callback function:

```python

app.layout = html.Div([
    html.Div([html.H1('Yeast expression browser', style={"text-align": "center"}),
              html.H3('Parameters', style={"text-align": "center"})],
             className="row"),
    html.Div([
        html.Div([
            html.Div([html.Div('Smoothing fraction:', id='smoothing_fraction'), dcc.Slider(
                id='fraction_slider',
                min=0,
                max=1,
                step=0.05,
                marks={0: '0', 1: '1.0'},
                value=0.5 # default smoothing fraction
                )],className="four columns"),
            html.Div([dcc.Dropdown(
                id='gene_selector',
                options=[{'label': gene, 'value': gene} for gene in expr.index],
                value=expr.index[0],
                multi=False)], className="four columns"),
            html.Div([html.Div('correlation_cutoff', id="cor_value"), dcc.Slider(
                id='cor_slider',
                min=0,
                max=1,
                step=0.05,
                marks={0: '0', 1: '1.0'},
                value=0.6 # default correlation cutoff
                )], className="four columns")],
            className="container"),
        html.Div([
            html.Div([html.Br(), html.Br(), html.H3("Expression plot", style={"text-align": "center"}),
                      dcc.Graph(id='expression_plot')],
                     className='six columns'),
            html.Div([html.Br(), html.Br(),
                      html.H3("Heatmap", style={"text-align": "center"}),
                      dcc.Graph(id='heatmap_plot')],
                     className='six columns')], className='row')])])

@app.callback(
    [Output('heatmap_plot', 'figure'),
     Output('cor_value', 'children')],
    [Input('gene_selector', 'value'),
     Input('cor_slider', 'value')])
def update_clustergram(gene_id, cor):
    """
    Updates the clustergram/heatmap in the webapp by using gene selector and correlation slider input components.
    """
    if not gene_id or not cor: 
    # If any of the inputs are missing then no figure is returned
        return None, f"Correlation cutoff ({0})"
    else:
        expr_gene = expr.loc[gene_id]
        cor_expr = expr.loc[np.corrcoef(expr_gene, expr)[0][1:] >= cor]
        if cor_expr.shape[0] > 1:
            heatmap = dashbio.Clustergram(
                data=cor_expr.values,
                column_labels=list(cor_expr.columns),
                row_labels=list(cor_expr.index),
                cluster='row',
                width=600,
                height=600,
                hide_labels='column',
                display_ratio=[0.3, 0.3],
                optimal_leaf_order=True,
                line_width=2
            )
            return {'data': heatmap}, f"Correlation cutoff ({cor})"
        else:
            return {'data': None}, f"Correlation cutoff {cor} is too high!"
```