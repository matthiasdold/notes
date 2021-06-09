# Dash
## Multi-Page app
Based of my general workflow, I find it desireable to build individual apps for a dedicated purpose (single page) and later concatenate them to one multi-page app.
The general starting point for a dash multi-page app would be in the [official dash example](https://dash.plotly.com/urls).


### Folder Structure
For the first setup, we will follow this layout:
````
run_app.py
app.py
assets/
    my_style.css
src/
    combined_apps.py
    app1.py
    app2.py
    __init__.py
````

This setup provides a simply trick: It is paramount to have the app.py, which contains an instance of `dash.Dash(__name__)` in the folder where the `assets` folder is located.
The creation of this instance will change the path for looking for the assets to where the script is located (via `__name__`).
Now all sub apps get their app instance from `app.py` and set the layout to it as well as registrating the callbacks to this app.
This has two nice side effects:

1. There is a global `my_style.css` or multiple others for individual apps. Note that all css from the assets folder are loaded
2. The callbacks are all getting registered to on app. So no need to do this manually ex post


With the following contents

`app1.py`
````python
import dash
import dash_core_components as dcc
import dash_html_components as html

from app import app

layout = html.Div([
        html.Div(id='app1_page_content', children=[])
    ])

app.layout = layout

if __name__ == '__main__':
    app.run_server(debug=True)
````

`app2.py`
````python
import dash
import dash_core_components as dcc
import dash_html_components as html

from dash.dependencies import Input, Output

from app import app

layout = html.Div([
    html.Div(id='app2_page_content',
        children=[
            html.Div(id='app2_value_div', children=['1']),
            html.Button(id='app2_button1', n_clicks=1, children='clickme')
        ])
])

app.layout = layout

@app.callback([Output('app2', children)],
              [Input('app2_button1', 'n_clicks')]
)
def increment_data(n_clicks):
    return [str(n_clicks)]

if __name__ == '__main__':
    app.run_server(debug=True)
````

Note: It is important, that the layouts are accessible from the outside (e.g. as a global variable)
so that the `combined_app.py` can serve the correct layout depending on the url.

`combined_apps.py`
````python
import dash
import dash_core_components as dcc
import dash_html_components as html

from dash.dependencies import Input, Output
from src.app1 import layout as layout_app1
from src.app2 import layout as layout_app2

from app import app

app.layout = html.Div([
    dcc.Location(id='url', refresh=False),
    html.Div(id='main_page_body',
             children=[
                html.Div(id='main_page_content',
                         children=[]
                         )
             ])
])

# Callback for page switching
@app.callback(Output('page-content', 'children'),
              Input('url', 'pathname'))
def display_page(pathname):
    if pathname == '/apps/app1':
        return app1.layout
    elif pathname == '/apps/app2':
        return app2.layout
    else:
        return '404'

````

`run_app.py`
````python
from src.combined_apps import app

app.run_server(debug=True, use_reloader=False)
````

Note that it is important to have individual `id`s accross all apps which should be integrated! Else the combination will fail
### Downside of this approach
What I do not like about this structure is, that for comining the apps, I will import a global layout only.
What this implicitly does however, is set the callbacks to the same app instance since the `from app import app` will refere
to the same app instance. This is unclear and did take too much debugging to find out.
