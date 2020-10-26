# Predictive Maintenance for Turbofan Jet Engine



Turbofan engines are widely used in the aviation industry to power commercial airliners. Their working principle is analogous to that of a traditional gas turbine engine, with the addition of a *turbine fan*. The turbofan increased the compression of the inlet air and splits it diverting some air to bypass the burner core. This increases thrust with minimal increase in fuel consumption when compared to a gas turbine engine<sup>[1](#myfootnote1)</sup>



## Why Predictive Maintenance?

A large Turbofan engines like the General Electric GE90 designed to be fitted on the Boeing 777 can cost upward of $27.5 Million dollars<sup>[1](#myfootnote2)</sup>. However the expensive machines to purchase and operate. Traditionally preventive maintenance is scheduled based on the number of *flight cycles* (one take-off and landing) to minimize the risk of downtime and failure during operation. Because of the complexity of the engine the maintenance costs are very high. There are two kinds of service events that an engine will need during its life:

- Hot Section Inspection (HSI). This type of service is typically performed on wing and aims at inspecting and ensuring peak performance of the engine. It is typically less expensive and can take 1 or 2 days to complete. 
- Full Engine Overhaul. This type of maintenance is the most expensive and can cost around 1/10th of the purchase price, on top of that it takes around 50/60 days to complete the service thus requiring the engine to be replaced in order not to ground the plane needlessly. This further increases the costs of ownership over the lifetime of the engine.

The Full engine Overhaul happens on a strict schedule dictated by the manufacturer who sets a maximum Time Between Overhauls (TBO) for the engine. Additionally it can be performed on-condition if the HSI reveals premature wear or damage to the engine. By modeling and predicting the failure events, we could transition from a **schedule-based** to an **on-condition TBO** **cycle.** that can result in reduced extended TBO and thus lower operating costs over the life of the engine. 



## Data

To predict the imminent failure of an Engine we use simulated run-to-failure data provided by the Prognostic Data Repository of NASA<sup>[1](#myfootnote3)</sup>. The dataset contains simulations of multiple engine units under different operating parameters. It includes 4 datasets (FD001, FD002, FD003, FD004) with different operating conditions and fault modes.
The database is downloaded directly from the NASA web server through a ``wget`` request, and saved on disk with the following file structure.

```
data/raw
├── Damage Propagation Modeling.pdf
├── readme.txt
├── RUL_FD001.txt
├── RUL_FD002.txt
├── RUL_FD003.txt
├── RUL_FD004.txt
├── test_FD001.txt
├── test_FD002.txt
├── test_FD003.txt
├── test_FD004.txt
├── train_FD001.txt
├── train_FD002.txt
├── train_FD003.txtmodelling
└── train_FD004.txt
```

For ease of use we consolidate the data into a more compact data structure that preserves the train/test/label split. These are located in the main data structure as follows:

```
data 
├── RUL.csv 
├── test.csv 
└── train.csv
```

 The `test.csv` and `train.csv` have identical structure:

- `unit_number` - each individual engine unit
- `cycle_time`  - one flight cycle (take-off/landing) for the specific engine unit
- `op_setting_1`, `op_setting_2` , and  `op_setting_3` - the operational setting parameters
- `s1` through `s21` - the average sensor reading for that specific cycle
- `dataset` - a label that keeps track of what dataset these points belonged to

The only difference between the `test.csv` and `train.csv` data is the fact that the last cycle of each unit in the training set corresponds to the engine failure. While the last cycle on the test set is an arbitrary point in time *before* the engine failure. The third file `RUL.csv` contains a single column with the remaining usable life for each engine in the test dataset. 

## Data Cleaning

Since the data is the result of a simulation there is little to no manipulation that needs to be done in order to get the data ready for modeling. There are no duplicates or missing values. Checking for outliers with a 1.5*IQR window highlights a large amount of points. These are highlighted in red in the figures below. 

![Cycle Time](/home/andrea/Dropbox/PyProjects/Predictive_Maintenance_Fanjet/figures/outliers/train_outliers_cycle_time.png)

![Operating Setting 3](/home/andrea/Dropbox/PyProjects/Predictive_Maintenance_Fanjet/figures/outliers/train_outliers_op_setting_3.png)

![Sensor 14](/home/andrea/Dropbox/PyProjects/Predictive_Maintenance_Fanjet/figures/outliers/train_outliers_s14.png)



By visual analysis of these figures we conclude that these points are simply extreme values of non-normally distributed data. Thus we decide not to remove or otherwise substitute these data points. 

## Exploratory Data Analysis 

### Operating  Settings

The first element which we explore of this dataset is the 3 columns containing the operating settings. By plotting them in a 3D plot we observe that the combination of these settings clusters around 6 distinct points in space, we call these *operating regimes*. We apply the unsupervised learning technique of k-means clustering to automatically label the data. In the image below, each color corresponds to a different label and the cross-hairs are positioned at the center of the cluster. 

![Operating Condition clusters](/home/andrea/Dropbox/PyProjects/Predictive_Maintenance_Fanjet/figures/Op_regime_clustering.png)modelling

By plotting the newly created `op_regime` label against the `dataset` category we observe that dataset FD001 and FD003 only contain data with one operating regime. While dataset FD002 and FD004 contain data in all 6 operating regimes. 

![Operating Regimes in each dataset](/home/andrea/Dropbox/PyProjects/Predictive_Maintenance_Fanjet/figures/Op_regime_vs_dataset.png)

This analysis has been conducted on both training and testing data with identical results. 

Based on these finding we decide to narrow down the scope of the first modeling attempt to the FD001 dataset, which contains a single operating regime. As a consequence of this choice we can discard the `op_setting_1`, `op_setting_2`, and `op_setting_3` columns for modeling purposes.

### Sensor Readings

We know from the data summary that the common point in time for all engine units in the training dataset is the last cycle, which corresponds to the failure of the engine. This means that plotting the sensors traces against the `cycle_time` is not the best way to visualize the data. We decide then to convert the `cycle_time` axis into a Remaining Usable Life `RUL` variable that aligns all the sensor traces to the 'right', i.e. to the engine failure (red line). Plotting all the traces at once would result in a crowded and not clear plot. We randomly select 10 engine units for plotting and we overlay the 10-cycle rolling average to filter out noisy behavior. The resulting plot, for the dataset FD001, is displayed below.

![sensor traces](/home/andrea/Dropbox/PyProjects/Predictive_Maintenance_Fanjet/figures/FD001_sensor_traces.png)

From this we observe how sensors `s1`, `s5`, `s6`,`s10`,`s16`, `s18`, and `s19` have very low variance and do not display any correlation with the `RUL` variable. 



## Method

### Outcome Variable

There are at least two approaches that we can take to model this dataset and obtain a prediction that can inform the maintenance scheduling. One approach would be to train a regression model to learn to predict the number of cycles until failure, in other words the Remaining Usable Life of an engine. However, to begin with we decide, to frame the prediction task as a classification problem. In this case we aim to predict  the urgency of scheduling a maintenance.  To achieve this we divide the data in 3 categories based on their proximity to the failure event:

(1) Normal, more than 100 cycles from failure.
(2) Warning, between 100 and 50 cycles from failure.
(3) Alert, less than 50 cycles from failure.


![Category Lables](/home/andrea/Dropbox/PyProjects/Predictive_Maintenance_Fanjet/figures/Category_Labels_train.png)

The choice of these values has been driven by a visual analysis of the data, however it's reasonable to expect that they can be adjusted to best fit the advance notice needed by the service and maintenance supply chain. 

Intuitively one can observe how there is an imbalance between the classes since class 3 covers a wider range of cycles. If we plot an histogram of the training dataset, we therefore observe that the prevalent class is #3.

![Imbalanced dataset]()

Because of this imbalance we decide to create an alternative dataset, which is balanced by random oversampling, for comparison purposes.

![Balanced dataset]()

### Feature Selection

#### Input Variables

Columns `dataset` and `unit_numer` can be discarded as they bear no relationship with the failing of the engine.

As a result of the observations and choice regarding the operating regime we can discard the `op_setting_1`, `op_setting_2`, and `op_setting_3` columns for modeling purposes.

As a result of low variance and correlation of several sensor readings, observed in the a previous section we can discard  the input variables `s1`, `s5`, `s6`,`s10`,`s16`, `s18`, and `s19` . 

The variable  `RUL` contains information about the failure of the engine and it was used to derive the outcome variable `status` we therefore discard it.

#### Output Variables

The output variable is the `status` column of the dataset.

## Machine Learning Models

blah blah

## Future Improvements

fjfj

## References

<a name="myfootnote1">1</a>: Turbofan Engine. 2020. Grc.Nasa.Gov. https://www.grc.nasa.gov/WWW/k-12/airplane/aturbf.html.

<a name="myfootnote2">2</a>: General Electric GE90. 2020. Wikipedia.Org. https://en.wikipedia.org/wiki/General_Electric_GE90.

<a name="myfootnote3">3</a>: A. Saxena and K. Goebel (2008). "Turbofan Engine Degradation Simulation Data Set", NASA Ames Prognostics Data Repository (http://ti.arc.nasa.gov/project/prognostic-data-repository), NASA Ames Research Center, Moffett Field, CA