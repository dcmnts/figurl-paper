# Collaborative scientific visualization in the browser with Figurl

## Abstract

Data visualization is an essential tool for scientists to understand their measurements, effectively communicate their results, and collaborate with other researchers. Interactive visualizations are particularly powerful, allowing for faster exploration and annotation of complex, multidimensional data sets. Despite their effectiveness, most current scientific visualization tools generate static figures or are difficult to share. To address these limitations, we developed Figurl, a web-based data visualization system that makes it easy to generate, view, share, and archive web-based interactive figures and documents using Python scripts. Figurl utilizes shareable links, content-addressable storage, and domain-specific plugins to allow scientists to easily generate and share browser-based visualizations. We demonstrate the utility of Figurl in various neuroscience applications, illustrating the potential of this technology to facilitate collaborative science.

## Introduction

<!-- Visualization software is important -->
Visualization plays a critical role in every stage of the data-centric scientific process, enabling data exploration, data cleaning, data analysis and interpretation, and communication of findings to the scientific community. Interactive visualizations are particularly effective as they enable scientists to quickly explore large, complex datasets, thereby enhancing understanding of the data and hypothesis generation. For instance, Liu and Heer (2014) found that a 500 millisecond delay between visualizations could reduce the amount of explored dataset and affect the number of hypotheses and observations formed. The ability to comprehend complex datasets is becoming more crucial as the complexity of data collected and analyses increases. The power of interactive visualization has led to the development of advanced plotting libraries (e.g., Bokeh, Plotly) and domain-specific software packages for visualizing complex datasets (e.g., Buzsaki lab matlab tools, Freeview module in Freesurfer, Pysurfer, SPM). 


<!-- Most visualizations cannot be shared - greatly reduces their utility -->
However, these tools have significant limitations in their ability to support shared, interactive visualizations. Typically, these software packages can only operate on data that is available on a local workstation or compute cluster and require local computational resources that may not be accessible to others. Consequently, most visualizations shared in publications are static images or videos that do not enable others to explore the data.

<!-- Advantages of browser-based visualization software -->
Browser-based software solutions offer an array of advantages compared with traditional approaches. The web browser is inherently platform-independent, and available from any computer or device with access to the internet. Delivering software via web applications eliminates the need to install or upgrade software, and integrates well with authentication and collaboration systems. While desktop software may sometimes offer advantages in terms of performance and access to local on-disk datasets, browser-based and cloud-based visualization tools are usually the most convenient for the end user and provide the greatest opportunities for integration and collaboration.

<!-- Examples of browser-based tools including jupyter and Colab -->
The numerous advantages of browser-based visualization software have spurred the development of various web-based approaches to data visualization. These approaches encompass hosted platforms where users can launch interactive notebooks, execute code, and visualize data using cloud-based resources, such as Jupyter and Google Colab [REFS]. Additionally, purpose-built websites have been designed to implement visualizations tailored to specific datasets, catering to the unique needs of different scientific domains [REFS].

<!-- Limitations of jupyter/jupyterhub in terms of sharing -->
While these interactive notebooks are powerful, they also have some limitations. They still require the allocation and support for both cloud-based computational and storage resources, which means that individual laboratories or scientists need to set up and maintain the computational resources or work with an organization (e.g. 2i2c [REF]) to do so. They also require substantial expertise in the specific visualization packages used. These factors limit where and how such software can be used.

An alternative approach capitalizes on the power and simplicity of clickable hyperlinks. In applications such as Neuroglancer [REF], users disseminate their visualizations with simple URLs, with the recipients accessing the content without needing to install additional software or sign in to any service. This approach is facilitated by the employment of low-barrier, readily accessible infrastructure, including cloud storage and serverless APIs. Instead of depending on costly server-side resources, these applications utilize in-browser (client-side) computation and software characterized by its minimal footprint and ease of deployment.

<!-- Figurl description - revise this section to outline exactly what we set out to do -->

<!-- General tool rather than narrow application -->

<!-- User can easily provide their own data for a visualization -->

<!-- Flexibility in choose data + flexibility in choosing the GUI/view -->

Our goal was therefore to develop a browser-based visualization system that addresses these limitations. Specifically, we aimed to build a system where 1) individual users could easily configure the necessary cloud resources, 2) computations for visualization were done in the browser rather than in the cloud, and 3) users only need to specify the dataset and a limited number of parameters for the visualization and are not required to learn the visualization software package itself. The resulting system, Figurl, is a framework for generating shareable URL links from Python scripts that point to browser-based, domain-specific, interactive views of scientific datasets in the cloud. It is designed to be used in collaborative environments, such as scientific research teams, and is built on top of Kachery, a content-addressable storage (CAS) network in the cloud. In this paper we show how Figurl and Kachery facilitate frictionless data sharing, enable insightful dissemination of datasets, and promote scientific reproducibility. We illustrate the usefulness of our software with examples from neuroscience applications, but these approaches could be used in any field.

## Methods

### Creating and sharing a figure

<!-- How does figurl work? Script -> URL -> browser -->
The basic flow of operation for Figurl is shown in Figure D1. A Python script creates a piece of data to be visualized and uploads it to the Kachery, a content-addressable storage (CAS) database in the cloud (described below). After this, it generates a URL (figURL) that pairs that data with a visualization module. If the figure URL is created on a workstation or in a notebook, the user can view the interactive figure by clicking on the link output by the script, or they may share the link with collaborators if desired. Alternatively, if the URL is generated in an automated processing pipeline or in a CI workflow, it can be saved in a database or included in an output file, to be viewed at a later time.

![Figurl diagram (1)](https://user-images.githubusercontent.com/3679296/232157256-9ab63fa5-6151-4b7f-b580-18b1b20ae361.png)

***Figure D1.** Diagram showing the basic flow of operation for Figurl. A Python script creates a piece of data to be visualized and uploads it to Kachery. After this, it generates a URL (figURL) that pairs that data with a visualization module.*

<!-- An example script for creating creating a static figure -->

An example Python script for creating a simple static figure using the Plotly [REF] library is shown in Figure P1. The critical line of code prepares a serialized version of the figure, uploads it to Kachery in exchange for a content URI, and generates a URL that pairs the data with the Plotly visualization plugin (a wrapper around plotly.js). The output of this script is linked in the figure caption. In general, this URL can be output to the terminal or stored in a database. Because plotly.js is fully supported in the browser, any Plotly chart can be converted into a figURL in this way. Similarly, since the Altair Python package uses the VegaLite browser-based graphics system, Altair charts may also be shared using Figurl. Other plotting frameworks could be used as well, assuming the appropriate visualization wrappers are created.

```python
import plotly.express as px
import figurl as fig

# Load the iris dataset and create a Plotly figure
iris = px.data.iris()
ff = px.scatter_3d(iris, x='sepal_length',
		y='sepal_width', z='petal_width',
		color='species')

# Create and print the figURL
url = fig.Plotly(ff).url(label='plotly example - iris 3d')
print(url)

# Output: 
# https://figurl.org/f?v=gs://figurl/plotly-1&d=sha1://5c6ec276ce9a3b20b208aaff911b037ce4052e51&label=plotly%20example%20-%20iris%203d
```
***Figure P1.** Example Python script for creating a shareable Figurl figure using Plotly. The output of this script is a URL that can be clicked to view the figure in a browser.*

<!-- Description of the d and v query parameters in the figURL -->

In the generated URL of Figure P1, the data is specified with the d query parameter, which is the data content URI, and the visualization is specified with the v query parameter, which points to the HTML/JavaScript code hosted on the web. The URL is simply a pairing between these two resources.

## The Figurl web application

<!-- Overview of the Figurl web app -->

Figurl is hosted as a serverless app on Vercel - scalable

Communication between the web app and visualization plugin via postMessage API

### Kachery network

<!-- Overview of Kachery network as a collection of zones -->
Figurl is built on the Kachery network, which is composed of content-addressable cloud storage databases called Kachery zones. These zones are hosted and maintained by individual research labs or collaborative teams, with a public zone maintained by the authors' institution. This decentralization allows labs with large datasets to provision and access their own resources without overloading the central public server. With that said, the public zone is open to everyone for moderate usage, enabling researchers to get started generating shareable links from their Python scripts with minimal configuration.

<!-- Description of a Kachery zone -->
A Kachery zone is made up of a cloud bucket and other cloud resources, allowing for communication between clients uploading to and downloading from the zone. The main purpose of the bucket is to hold individual data files indexed by their content hashes. For example, the file containing the text "kachery is scalable" has a SHA-1 hash 04046d2579ead593eb93904774f1571e6b1a4f8b and is stored in the default bucket associated with that key. The Kachery URI referring to that piece of data would be sha1://04046d2579ead593eb93904774f1571e6b1a4f8b.

After a one-time configuration step, users can immediately get started storing and retrieving data from the Kachery cloud using the Python commands shown in Figure K1.

```python
import numpy as np
import kachery_cloud as kcl

# Store data in the Kachery cloud
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
***Figure K1.** Example Python script for storing and retrieving data from the Kachery cloud.*

<!-- Advantages of Content-addressable storage -->

Content addressing offers several advantages for reproducible science. By not relying on location-specific information to retrieve files, we are free to change the data's underlying storage location without invalidating the associated links. This is especially useful when publishing visualizations, as it may be desirable to move the associated data to a long-term archival Kachery zone while keeping the figURL references the same. Furthermore, content hashes provide a means of verifying the integrity of data files. When a file is downloaded, Figurl (or a Python client) confirms that the content hash matches the URI, ensuring that the retrieved file or visualization is identical to the original version. Other practical advantages of CAS include automatic deduplication, straightforward backup and caching strategies, and technology independence.

### Visualization plugins

<!-- Overview of visualization plugins -->

As described above, the Figurl web app pairs the data object defined by the d query parameter in the figURL with the visualization plugin (v query parameter). The visualization plugin is a static HTML bundle, containing all the HTML and JavaScript files that have been compiled down from the ReactJS/typescript application. This bundle is comparable to a binary executable, but instead of being executed by an operating system, it gets downloaded and executed by the web browser. The Figurl web app loads the plugin into an embedded iframe and manages the interaction between the plugin and the kachery-cloud network (authentication, file downloads, etc).

<!-- Versioning of plugins -->

Usually the visualization plugin is hosted on a cloud storage bucket. For example, in the Altair plot of the example from Figure 2, the plugin is found at gs://figurl/vegalite-2 on a Google bucket. The -2 at the end of the URI represents the version. If we want to make updates to the visualization, such as improving the layout or adding features that will not affect existing links, then the new HTML bundle can be uploaded to the same location as the existing plugin. However, if the changes would break existing links, for example due to a change in data spec, then the version number should be incremented, the new bundle uploaded to gs://figurl/vegalite-3, and all future figURLs pointed to the new location.

<!-- Plugins are simply static HTML bundles (no server) which is important for scaling and archiving -->

Plugins are simply static websites that are embedded in the parent figurl.org web application. This is a significant simplification compared with traditional websites that usually require a running server providing a live API. The responsibility for loading data, verifying the integrity, and managing other aspects of the user interface are removed from the visualization itself, and handled by the parent figurl.org app. This design allows us to store visualization plugins on storage buckets for long-term availability; this is crucial for allowing figURLs to stay valid even as the visualization plugins are updated and improved over time.

### Rtcshare

### Saving data to Kachery or GitHub

TODO

## Examples

### Spike sorting analysis and curation

Spike sorting is an essential clustering technique in neurophysiology, used to identify and classify individual neurons based on multi-electrode extracellular recordings. As input, a spike sorting algorithm takes a multi-channel time series. In a typical implementation, the algorithm first uses signal processing to detect spikes in the dataset which are associated with individual neural firing events. Features of the signal near each spike are then used to cluster the events into units that correspond to distinct putative neurons. The output of the procedure is a list of event timestamps together with neuron labels, where events with the same label are assumed to have been generated by the same neuron.

Once data have been collected and spike sorting performed, the researcher will want to explore the data to gain insights about the experiment and the quality of the spike sorting, and to select clusters (neural units) for further analysis. To address this need, we have developed a Figurl visualization plugin specifically designed for spike sorting analysis (see Figure E1). This plugin enables the interactive visualization of various aspects of the spike sorting process, including autocorrelograms, cross-correlograms, average waveforms, spike raster plots, and other relevant views. By presenting a rich and informative visualization environment, researchers can delve into the data and gain a deeper understanding of the experiment and the effectiveness of the spike sorting algorithm.

The plugin allows users to seamlessly explore these different views and manually select units to accept or reject based on the quality and relevance of the identified clusters. After refining the selection of neural units, researchers can then easily export the results of their analysis. Figurl provides flexibility in exporting, offering users the option to save the results to the Kachery network or to GitHub for version control and integration with other projects. This streamlined process fosters a more efficient and effective workflow, promoting better understanding and interpretation of complex neuroscience data.

<!-- Screenshot goes here -->
***Figure E1.** Screenshot of the Figurl spike sorting visualization plugin. The plugin is designed to be intuitive and easy to use, with a simple interface that allows users to quickly navigate between different views and select units for further analysis. The plugin also provides a variety of options for exporting the results of the analysis, including saving the results to the Kachery network or to GitHub for version control and integration with other projects.*

### SpikeInterface integration

### Animal pose annotation

### Sound source localization

### Decoded animal position



## Discussion

## Conclusion

## References


