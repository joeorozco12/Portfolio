
```python
import pandas as pd
import numpy as np
import os
import matplotlib.pyplot as plt

from bokeh.io import output_file, output_notebook, export_png
from bokeh.plotting import figure, show
from bokeh.models import ColumnDataSource, NumeralTickFormatter, HoverTool,  PrintfTickFormatter

cols = ['Frequency', 'VSWR', 'S11(DB)','a ',' b',' c',' d',' e','h']
data_path = 'data/'

def get_file_paths(data_folder):
    list_csv_files = []
    file_index = []
    paths = []
    for root, folders, files in os.walk(data_folder): 
        files = [f for f in files if not f[0] == '.']
        folders[:] = [d for d in folders if not d[0] == '.']
        if files == []:
            continue
        for index, value in enumerate(files):
            list_csv_files.append(value)
            file_index.append(index)
            p = root + value
            paths.append(p)
    return file_index, list_csv_files, paths     

def plot_figures(df,ax):
    df.plot(ax=ax,
            x='Frequency',
            y='VSWR',
           )

file_index,files,paths = get_file_paths(data_path)
fig, ax1 = plt.subplots(figsize=(11,8))
select_tools = ['box_select', 'lasso_select', 'poly_select', 'tap', 'reset', 'wheel_zoom', 'box_zoom']
linecolor = 'royalblue','green','red'
DUT = 'DUT1', 'DUT2','DUT3'

fig = figure(plot_height=820,
             plot_width=1800,
             x_axis_label='Frequency (GHz)',
             y_axis_label='VSWR',
             title='VSWR',
             toolbar_location='below',
             tools=select_tools,
            )

for index, value in enumerate(paths):
    df = pd.read_csv(value,
        # skiprows=8,
        delimiter=',',
        # header = 19,
        names = cols,
        )
    if any(df['VSWR'] < 1) :
        df['VSWR'] = (10**(-df['VSWR']/20) + 1) / (10**(-df['VSWR']/20) - 1)
    x= df['Frequency']
    y = df['VSWR']
    plot_figures(df,ax1)
    data_cds = ColumnDataSource(df)
    fig.line(x='Frequency',
            y='VSWR',
            source=data_cds,
            color=linecolor[index],
            selection_color='deepskyblue',
            nonselection_color='lightgray',
            nonselection_alpha=0.3,
            legend_label=DUT[index],
            line_width=2.0,
  
            )

output_file('DynamicGraph.html', title='Jose\'s Dynamic Graphing Skills!!')
output_notebook()

fig.title.align = 'center'
fig.xaxis[0].formatter = PrintfTickFormatter(format="%1.0f")
fig.yaxis[0].formatter = NumeralTickFormatter(format='0.00')
fig.legend.title = 'DUT\'s'
fig.legend.location = 'top_left'
fig.add_tools(HoverTool(
    tooltips=[
              ( 'Frequency',   '$x GHz'),
              ( 'VSWR',        '@{VSWR}' ),
             ],
    formatters={
                '@{Frequency}' : 'numeral',
                '@{VSWR}' : 'numeral',
                },
    mode='vline'
                        )
                )

show(fig)
```
