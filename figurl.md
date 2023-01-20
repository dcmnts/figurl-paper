# Interactive and collaborative scientific visualization using Figurl

## Abstract

Data visualization is an essential tool for scientists to effectively communicate their results and to collaborate with other researchers. However, most scientific visualizations are limited either in their interactivity or with their portability. To address these limitations, we have developed Figurl, a web-based data visualization system that makes it easy to generate, view, share, and archive web-based interactive figures and documents from Python scripts with minimal configuration. Figurl utilizes shareable links, content-addressable storage, domain-specific plugins, and Zenodo to allow scientists to easily view, share, and archive browser-based visualizations. We demonstrate the utility of Figurl in two neuroscience contexts, illustrating the potential of this technology to facilitate collaborative science.

## Overview of existing tools

### Paraview/ParaviewWeb

Open source, desktop, medical imaging datasets, customizable, ParaViewWeb is a framework for creating web applications for scientific visualizations

From GPT-3. Not sure how accurate
Explain how one would use ParaviewWeb to share an interactive visualization.
	ParaviewWeb is an open source software package that allows users to share interactive visualizations through a web browser. To use ParaviewWeb, the user should first create a visualization in Paraview, the powerful 3D data visualization program. Once the visualization is complete, the user can save it as an .pvsm file. This file can be uploaded to a web server and accessed through a web browser using the ParaviewWeb viewer. The user can then share the interactive visualization with others, who can interact with it using the ParaviewWeb viewer. The user can control the visualization by selecting different options from the viewer's menu and manipulating the data. ParaviewWeb also allows users to share their visualization with others by creating a link that can be shared over email, social media, or other web-based platforms.

### VTK / vtkWeb

From GPT-3. Not sure how accurate.
Explain how one would use vtkWeb to share an interactive visualization.
	1. Install a web-server such as Apache, Nginx or IIS.
	2. Download and install the vtkWeb package.
	3. Set up a vtkWeb application on the web-server.
	4. Create the visualization in VTK or ParaView.
	5. Publish the visualization using vtkWeb.
	6. Share the URL of the visualization with your audience.
	7. Your audience can now access the interactive visualization and use it to explore the data.

### Plotly dash

From GPT-3. Not sure how accurate.
Explain how one would use plotly dash to share an interactive visualization.
	Plotly Dash is an open source library that can be used to create interactive web applications for data visualization. To use Plotly Dash to share an interactive visualization, first create the visualization using Plotly's graphing library. Then define a 'layout' object in the Dash application which will contain the figure created using the Plotly library. Once the layout is defined, use the Dash 'app.layout' object to add components such as input controls and other HTML elements. Finally, use the 'dash.run_server()' function to launch the application in a local web server. This will allow users to access the interactive visualization from a web browser.

## HoloViz

From GPT-3. Not sure how accurate.
Explain how one would use holoviz to share an interactive visualization.
	1. Begin by importing the necessary packages to use HoloViz including: hvPlot, HoloViews, and Panel.
	2. Prepare your data and create the appropriate plot type (e.g. scatter plot, bar chart, etc.) using hvplot.
	3. Use HoloViews to add interactive elements to the plot, such as hover-over information, selection of regions, and color palette changes.
	4. Use Panel to turn the plot into an interactive widget that can be shared.
	5. Publish the plot online using either the HoloViz web app or the Panel serve command.
	6. Share the link to the interactive visualization with friends and colleagues.

## Bokeh

From GPT-3. Not sure how accurate.
Explain how one would use bokeh to share an interactive visualization.
	1. Create the visualization: Create your visualization in Bokeh. This can be done with a Bokeh server or by using the Bokeh plotting API.
	2. Embed the visualization: Use the Bokeh embed command to embed the visualization in an HTML page that can be opened with a web browser.
	3. Share the visualization: Share the HTML page with others, or post the page on a website, or share it through social media. This will allow others to view and interact with the visualization.

### Datajoint

### Jupyter -

### Blender

desktop software for 3D visualization (does not run in browser as far as I can tell)

### Three.js / WebGL / Vue.js / d3.js

### Figshare / Open Science Framework (OSF)

### Python packages: matplotlib, seaborn, plotly, bokeh, altair, vispy

### CodeOcean

### Infogram.com

“Create engaging infographics and reports in minutes”

## Introduction

The clickable hyperlink is perhaps the most portable, versatile, and robust method for sharing data. Because web URLs are relatively short strings of text, they can be emailed, texted, uploaded to a database, or posted just about anywhere. As compared to working with files, URL links take up far less disk space, do not require special software for reading, and can be opened/launched from wherever they are found. Provided that the underlying data is stored in a reliable archive, URL permalinks can even be embedded in publications or long-term archives.

In contrast, traditional software tools for scientific visualization do not usually produce such permalinks by default, but instead typically produce files that are either in non-interactive formats (pdf, png, svg) or in formats specific to certain software that must be installed and configured. Often, these packages do not produce files at all, but interface directly with running processes. These factors limit where and how such software can be used. For example, some plotting libraries only work within Jupyter notebooks, or must be differently configured to operate in other settings. Cloud processes that run as part of continuous integration (CI) systems are also limited in terms of how they can produce interactive visual output.

Beyond the convenience of shareable URLs, web-based visualization software has an array of benefits when compared with desktop tools. The main advantage is that the web browser is inherently platform-independent, available on all desktop operating systems as well as on mobile devices. Another advantage is that web applications can be automatically upgraded without any action on the user's part. Further advantages relate to authentication, collaboration, and integration with cloud systems. While desktop software has advantages in terms of performance and access to local on-disk datasets, for most applications, browser-based and cloud-based visualization tools are the most convenient for the end user and provide the greatest opportunity for integration and collaboration.

Here we introduce Figurl, a framework for generating shareable URL links from Python scripts that point to browser-based, domain-specific, interactive views of scientific datasets in the cloud. It is designed to be used in collaborative environments, such as scientific research teams, and is built on top of Kachery, a content-addressable storage (CAS) network in the cloud. In this paper we show how Figurl and Kachery facilitate frictionless data sharing, enable insightful dissemination of datasets, and promote scientific reproducibility. The usefulness of our software will be highlighted in two neuroscience contexts.

## Methods

### URL generation

The basic flow of operation for Figurl is shown in Figure 1. A Python script creates a piece of data to be visualized and uploads it to the Kachery content-addressable storage database (described below). After this, it generates a URL (figURL) that pairs that data with a visualization module. If the figURL is created on a workstation or in a notebook, the user can view the interactive figure by clicking on the link output by the script, or they may share the link with collaborators if desired. Alternatively, if the figURL is generated in an automated processing pipeline or in a CI workflow, it can be saved in a database or included in an output file, to be viewed at a later time.

<!-- https://docs.google.com/drawings/d/1mbB9KZ2Tq-PWOxIRYPc4OpPPDE5YF9U1HjIQ7H5bKJo/edit -->
![figurl overview](https://user-images.githubusercontent.com/3679296/174104842-cc3956bc-734b-4d38-85cc-f506e61092ec.png)

Figure 1. Diagram showing Figurl usage.

An example Python script for creating a simple static figure using the Altair library is shown in Figure 2. The `fig.Altair(chart).url(...)` line of code prepares a Vega-Lite spec of the chart, uploads it to Kachery in exchange for a content URI, and generates a URL that pairs the data with the `vegalite` visualization plugin. In the generated URL, the data is specified with the `d`  query parameter, which equals the data content URI, and the visualization is specified with the `v` query parameter, which points to the HTML/JavaScript code hosted on the web. This URL can then be output to the terminal or stored in a database. Because Altair uses the Vega-Lite graphics system, which is fully supported in the browser, any Altair chart can be converted into a figURL in this way.

```python
import figurl as fig

import altair as alt
from vega_datasets import data

# Create an Altair chart
# This one comes from the Altair Example Gallery
stocks = data.stocks()

chart = alt.Chart(stocks).mark_line().encode(
  x='date:T',
  y='price',
  color='symbol'
).interactive(bind_y=False)

# Create and print the figURL
url = fig.Altair(chart).url(label='stocks chart')
print(url)

# Output: 
# https://figurl.org/f?v=gs://figurl/vegalite-2&d=sha1://0369af9f1a54a5a410f99e63cb08b6b899d1c92f&label=stocks%20chart
```

Figure 2. Example script demonstrating basic Figurl usage.

In general, this procedure depends on the existence of a suitable visualization plugin that is capable of rendering a view of the data. In this paper, we discuss two domain-specific plugins to render a view of the data.

### Kachery

Figurl is built on the Kachery network, which is composed of content-addressable cloud storage databases called Kachery zones. These zones are hosted and maintained by individual research labs or collaborative teams, with a public zone maintained by the authors' institution. This decentralization allows labs with large datasets to provision and access their own resources without overloading the central public server. With that said, the public zone is open to everyone for moderate usage, enabling researchers to get started generating shareable links from their Python scripts with minimal configuration.

A Kachery zone is made up of a cloud bucket and other cloud resources, allowing for communication between clients uploading and downloading from the zone. The main purpose of the bucket is to hold individual data files indexed by their content hashes. For example, the file containing the text "kachery is scalable" has a SHA-1 hash `04046d2579ead593eb93904774f1571e6b1a4f8b` and is stored in the default bucket associated with that key. The Kachery URI referring to that piece of data would be `sha1://04046d2579ead593eb93904774f1571e6b1a4f8b`.

After a one-time configuration step, users can easily get started storing and retrieving data from the Kachery cloud using the Python commands shown in Figure 3.

```python
import numpy as np
import kachery_cloud as kcl

# Store data in the Kacher cloud
uri1 = kcl.store_file('/path/to/filename.dat')

uri2 = kcl.store_text('kachery is scalable')
# `sha1://04046d2579ead593eb93904774f1571e6b1a4f8b`

uri3 = kcl.store_json({'kachery': 'scalable'})
# sha1://5f48db59936131d6b995d999271ed0daf6dfafc8

array = np.array([[1, 2, 3], [4, 5, 6]], dtype=np.int16)
uri4 = kcl.store_npy(array, label='example.npy')
# sha1://bb55205a2482c6db2ace544fc7d8397551110701?label=example.npy

uri5 = kcl.store_pkl({'example': array})
# sha1://20d178d5a1264fc3267e38ca238c23f3e2dcd5d2

# On a remote computer retrieve the data
file_path = kcl.load_file(uri1)
text = kcl.load_text(uri2)
obj = kcl.load_json(uri3)
array = kcl.load_npy(uri4)
pkl = kcl.load_pkl(uri5)
```
Figure 3. Simple Python commands are used to store data to Kachery and retrieve it on a remote computur.

Content addressing offers several advantages for reproducible science. By not relying on location-specific information to retrieve files, we are free to change the data's underlying storage location without invalidating the associated links. This is especially useful when publishing visualizations, as it may be desirable to move the associated data to a long-term archival Kachery zone while keeping the figURL references the same. Furthermore, content hashes provide a means of verifying the integrity of data files. When a file is downloaded, Figurl (or a Python client) confirms that the content hash matches the URI, ensuring that the retrieved file or visualization is identical to the original version. Other practical advantages of CAS include automatic deduplication, straightforward backup and caching strategies, and technology independence.

### Visualization plugins

As described above, the Figurl web app pairs the data object defined by the `d` query parameter in the figURL with the visualization plugin (`v` query parameter). The visualization plugin is a static HTML bundle, containing all the HTML and JavaScript files that have been compiled down from the ReactJS/typescript application. This bundle is comparable to a binary executable, but instead of being executed by an operating system, it gets downloaded and executed by the web browser. The Figurl web app loads the plugin into an embedded iframe and manages the interaction between the plugin and the kachery-cloud network (authentication, file downloads, etc).

Usually the visualization plugin is hosted on a cloud storage bucket. For example, in the Altair plot of the example from Figure 2, the plugin is found at `gs://figurl/vegalite-2` on a Google bucket. The `-2` at the end of the URI represents the version. If we want to make updates to the visualization, such as improving the layout or adding features that will not affect existing links, then the new HTML bundle can be uploaded to the same location as the existing plugin. However, if the changes would break existing links, for example due to a change in data spec, then the version number should be incremented, the new bundle uploaded to `gs://figurl/vegalite-3`, and all future figURLs pointed to the new location.

Visualization plugins are simply static websites that are embedded in the parent figurl.org web application. This is a significant simplification compared with traditional websites that usually require a running server providing a live API. The responsibility for loading data, verifying the integrity, and managing other aspects of the user interface are removed from the visualization itself, and handled by the parent figurl.org app. This design allows us to store visualization plugins on storage buckets for long-term availability; this is crucial for allowing figURLs to stay valid even as the visualization plugins are updated and improved over time.

### Electrophysiology and spike sorting

Examples from spikeinterface -- Python code

### Sound source localization

### Github integration: collaborative curation

### Zenodo integration

### Edge integration

In certain instances, laboratories may possess a substantial amount of data stored on local servers that they would like to open up to a larger scientific audience, yet the sheer size of the data may make it infeasible to upload all the data to Kachery in advance. Perhaps it is expected that only small subset of the files will actually be requested, but it is not known ahead of time which files those would be.

In such a scenario the lab can host a Kachery resource, which is a service that runs on the lab computer and listens for file requests containing the desired content URIs. Upon receipt of a request, the associated file is uploaded to Kachery, enabling the requesting client to download it via the standard procedure.
