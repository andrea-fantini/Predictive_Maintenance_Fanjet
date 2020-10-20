Predictive Maintenance for Turbofan Jet Engine
==============================

This model aims at predicting the imminent failure of a jet engine. It is based on Engine Degradation Simulation data released in 2008 by the Prognostics Center of Excellence at NASA’s Ames research center.

- For the summary of the business relevance, data analysis and results see the [Slide Deck](reports/presentation.pdf)
- For the write up on the techniques used for the data analysis and modeling see the [White Paper](reports/white_paper.md)
- For the python code walk-through from importing the data to making predictions see the [Jupyter notebooks](notebooks/)
- To replicate the results see the [Set up](#set-up) instructions below

Project Organization
------------

    │
    ├── data               <- This folder contains both raw and processed data. 
    |
    ├── figures            <- Figures saved by scripts or notebooks.
    |
    ├── models             <- Contains the serialized models and final model specifications.
    |
    ├── notebooks          <- Jupyter notebooks. 
    │   ├── 1 - Importing Data.ipynb
    │   ├── 2 - Data Organization and Cleaning.ipynb
    │   ├── 3 - Exploratory Data Analysis.ipynb
    │   ├── 4 - Data Pre-processing.ipynb
    │   └── 5 - Modeling.ipynb
    |
    ├── reports          <- Contains the slide deck and white paper.
    |
    ├── environment.yml
    |
    ├── LICENSE
    |
    └── README.md           <- The top-level README for developers using this project.


Set up
------------

Install the virtual environment with ``conda`` and activate it:

```bash
$ conda env create -f environment.yml
$ conda activate Jet_clean 
```

 







