# mlcrate
A collection of handy python tools and functions, mainly for ML and Kaggle.

The methods in this package aren't revolutionary, and most of them are very simple. They are largely bunch of 'macro' functions which I often end up rewriting across multiple projects, all in one place and easily accessible as a quality of life improvement. Hopefully, they can be some use to others in the community too.

This package has been tested with Python 3.5+, but should work with all versions of Python 3. Python 2 is not officially supported (although most of the functions would work in theory.)

### Installation

Clone the repo and run `python setup.py install` within the top-level folder to install mlcrate.

### Dependencies

Required dependencies: `numpy`, `pandas`, `pathos`  
`mlcrate.xgboost` additionally requires: `scikit-learn`, `xgboost`

## Features

### Save/Load

mlcrate comes with a simple pickle wrapper for fast save/load of arbitrary python objects (with optional compression).
Works with numpy, pandas, etc. and objects >4GB. Also cross-compatible with standard pickle dump/load.

```python
>>> import mlcrate as mlc
>>> x = [1, 2, 3, 4]

>>> mlc.save(x, 'x.pkl.gz')  # Saves using GZIP when .gz extension is used

>>> mlc.load('x.pkl.gz')
[1, 2, 3, 4]
```

###### [mlcrate.save(data, filename)](https://github.com/mxbi/mlcrate/blob/df66daf0a9e7078058aa65a7f42f9509f0d2d300/mlcrate/__init__.py#L9)

Pickles the passed data (with the highest available protocol) to disk using the passed filename.
If the filename ends in '.gz' then the data will additionally be GZIPed before saving.

*Keyword arguments:*  
`data` -- The python object to pickle to disk (use a dict or list to save multiple objects)  
`filename` -- String with the relative filename to save the data to. By convention should end in '.pkl' or 'pkl.gz'

###### [mlcrate.load(filename)](https://github.com/mxbi/mlcrate/blob/df66daf0a9e7078058aa65a7f42f9509f0d2d300/mlcrate/__init__.py#L24)

Loads data saved with save() (or just normally saved with pickle). Uses gzip if filename ends in '.gz'

*Keyword arguments:*  
`filename` -- String with the relative filename of the pickle to load.  
*Returns:*  
`data` -- Arbitrary saved data

### Writing to a csv log one line at a time

```python
>>> log = mlc.LinewiseCSVWriter('log.csv', header=['epoch', 'loss', 'acc'])
>>> for i in range(10):
        # Run something here
        log.write([i, 0, 'nan']) # Results are flushed to file straight away
>>> !head -n 2 log.csv
"epoch","loss","acc"
"0","0","nan"
>>> log.close()
```

###### mlcrate.LinewiseCSVWriter(filename, header=None, sync=True, append=False)

CSV Writer which writes a single line at a time, and by default syncs to disk after every line.
This is useful for eg. log files, where you want progress to appear in the file as it happens (instead of being written to disk when python exists)
Data should be passed to the writer as an iterable, as conversion to string and so on is done within the class.

*Keyword arguments:*  
`filename` -- the csv file to write to  
`header` (default: None) -- An iterator (eg. list) containing an optional CSV header, which is written as the first line of the file.  
`sync` (default: True) -- Flush and sync the output to disk after every write operation. This means data appears in the file instantly instead of being buffered  
`append` (default: False) -- Append to an existing CSV file. By default, the csv file is overwritten each time.

### Easy multi-threaded function mapping with realtime progress bars

mlcrate implements a multiprocessing pool that allows you to easily apply a function to an array using multiple cores, for a linear speedup. In syntax, it is almost identical to multiprocessing.Pool, but has the following benefits:

- Real-time progress bar, showing the combined progress across all cores with tqdm,  where usually using multiprocessing means you don't know how long the process will take.
- Support for functions defined AFTER the pool has been created. With multiprocessing, you can only map functions which were created before the pool was created, meaning if you defined a new function you would need to create a new pool.
- Support for lambda and local functions
- Almost no performance degrading compared to using multiprocessing.

Example:
```python
>>> pool = mlc.SuperPool()  # By default, the number of threads are used

>>> def f(x):
...     return x ** 2

>>> res = pool.map(f, range(1000))  # Apply function f to every value in y
[mlcrate] 8 CPUs: 100%|████████████████████████████████████| 1000/1000 [00:00<00:00, 1183.78it/s]

>>> res[:5]
[0, 1, 4, 9, 16]

>>> # The above map command is equivalent to this, except multithreaded
>>> res = [f(x) for x in tqdm(range(1000)))]
```

### Time

###### [mlcrate.time.Timer()](https://github.com/mxbi/mlcrate/blob/4cf3f95f557886d8fdf97e4a5ab0908edaa51332/mlcrate/time.py#L4)

A class for tracking timestamps and time elapsed since events. Useful for profiling code.

```python
>>> t = mlc.time.Timer()
>>> t.elapsed(0)  # Number of seconds since initialisation
3.0880863666534424
>>> t.add('event')  # Log an event (eg. the start of some code you want to measure)
>>> t.elapsed('event')  # Elapsed seconds since the event
4.758380889892578
>>> t.format_elapsed('event')  # Get the elapsed time in a pretty format
'1h03m12s'
>>> t['event']  # Get the timestamp of event
1514476396.0099056
```

###### [mlcrate.time.str_time_now()](https://github.com/mxbi/mlcrate/blob/4cf3f95f557886d8fdf97e4a5ab0908edaa51332/mlcrate/time.py#L33)

Returns the current time as a string in the format `'YYYY_MM_DD_HH_MM_SS'`. Useful for timestamping filenames etc.

```python
>>> mlc.time.str_time_now()
'2017_12_28_16_58_29'
```

###### [mlcrate.time.format_duration(seconds, max_fields=3)](https://github.com/mxbi/mlcrate/blob/4cf3f95f557886d8fdf97e4a5ab0908edaa51332/mlcrate/time.py#L37)

Formats a duration in a pretty readable format, in terms of seconds, minutes, hours and days.

```python
>>> format_duration(3825.21)
'1h03m45s'
>>> format_duration(3825.21, max_fields=2)
'1h03m'
>>> format_duration(259863)
'3d01h17m'
```

*Keyword arguments:*  
`seconds` -- A duration to be nicely formatted, in seconds  
`max_fields` (default: 3) -- The number of units to display (eg. if max_fields is 1 and the time is three days it will only display the days unit)  
*Returns:* A string representing the duration

### Kaggle

###### [mlcrate.kaggle.save_sub(df, filename='sub.csv.gz')](https://github.com/mxbi/mlcrate/blob/4cf3f95f557886d8fdf97e4a5ab0908edaa51332/mlcrate/kaggle.py#L1)

Saves the passed dataframe with settings for a kaggle submission (index=False), and auto-enables GZIP compression if a '.gz' extension is passed.

```python
>>> df
   id  probability
0   0         0.12
1   1         0.38
2   2         0.87
>>> mlc.kaggle.save_sub(df) # Saved as sub.csv.gz with compression
>>> mlc.kaggle.save_sub(df, 'sub_uncompressed.csv')
```

*Keyword arguments:*  
`df` -- The pandas DataFrame of the submission  
`filename` -- The filename to save the submission to. Autodetects '.gz'

### XGBoost

###### [mlcrate.xgb.get_importances(model, features)](https://github.com/mxbi/mlcrate/blob/74742068560ff877d95e2e57fb4f3c854b7d381b/mlcrate/xgb.py#L4)
Get XGBoost feature importances from an xgboost model and list of features.

*Keyword arguments:*  
`model` -- a trained xgboost.Booster object  
`features` -- a list of feature names corresponding to the features the model was trained on.  
*Returns:*  
`importance` -- A list of (feature, importance) tuples representing sorted importance  

###### [mlcrate.xgb.train_kfold(params, x_train, y_train, x_test=None, folds=5, stratify=None, random_state=1337, skip_checks=False, print_imp='final')](https://github.com/mxbi/mlcrate/blob/74742068560ff877d95e2e57fb4f3c854b7d381b/mlcrate/xgb.py#L30)

Trains a set of XGBoost models with chosen parameters on a KFold split dataset, returning full out-of-fold training set predictions (useful for stacking) as well as test set predictions and the models themselves.  
Test set predictions are generated by averaging predictions from all the individual fold models - this means 1 model fewer has to be trained and from my experience performs better than retraining a single model on the full set.

Optionally, the split can be stratified along a passed array. Feature importances are also computed and summed across all folds for convenience.

*Keyword arguments:*  
`params` -- Parameters passed to the xgboost model, as well as ['early_stopping_rounds', 'nrounds', 'verbose_eval'], which are passed to xgb.train(). Defaults: early_stopping_rounds = 50, nrounds = 100000, verbose_eval = 1  
`x_train` -- The training set features  
`y_train` -- The training set labels  
`x_test` (optional) -- The test set features  
`folds` (default: 5) -- The number of folds to perform  
`stratify` (optional) -- An array to stratify the splits along  
`random_state` (default: 1337) -- Random seed for splitting folds  
`skip_checks` -- By default, this function tries to reorder the test set columns to match the order of the training set columns. Set this to disable this behaviour.  
`print_imp` -- One of ``['every', 'final', None]`` - 'every' prints importances for every fold, 'final' prints combined importances at the end, None does not print importance  
*Returns:*  
`models` -- a list of trained xgboost.Booster objects  
`p_train` -- Out-of-fold training set predictions (shaped like y_train)  
`p_test` -- Mean of test set predictions from the models  
`imps` -- dict with \{feature: importance\} pairs representing the sum feature importance from all the models.
