# The worst neighborhood to be a Women in Mexico City

This repo serves as source for maintaining the final assignment for Geospatial Analysis course page. See the whole project here https://cecilia-sartori.github.io/gs-exam/

This project aims to map violence against women (VAW) in Mexico City neighborhoods and identify what areas are most at risk within the city. After a brief introduction on VAW rates in the world, it will evaluate the spatial dependence between geographic locations of reported cases by using a smaller spatial unit, the census blocks (AGEB). In conclusion, it will investigate access to women’s shelters by computing the average travel distance from place of living.


# How to Run the Project

The files are written using Jupyter Notebooks. All the notebooks for Colab environment are located in Colab-folder and all notebooks for run the project on your machine are located in notebooks-folder.
Docs-folder is set as the GitHub Pages source.

The files must be executed in a specific order, and each has a goal:

0. datasets: data pre-processing
1. intro: plot choropleth maps with no geoinformation
2. mapping: create interactive map
3. spatialCorr: prove spatial dependence
4. street Network: compute distance in a graph


## Running in Google Colab

This option requires you to have a Google account. 
Navigate to the folder 'Colab'. For any `.ipynb` extension file in file list, copy and paste GitHub URL to the Open Notebook from Github functionality provided by [Colab](https://colab.research.google.com/).

In Colab, you can pass on the order of the files.  


## Running on Your Own Machine

First you need to clone the repo: 
```bash
git clone https://github.com/cecilia-sartori/gs-exam.git
```

### Requirements ###

Install all the packages

```bash
pip install -r requirements.txt
```

### Download Dataset Locally ###

Because of GitHub size limitation in uploading, some data need to be downloaded from original source.
Run `run_local.ipynb` as first step. Then follow the step described above. 


## License

**[Creative Commons Attribution-ShareAlike 4.0](https://creativecommons.org/licenses/by-sa/4.0/legalcode)**


