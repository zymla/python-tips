# python-tips

## Jupyter
### Matplotlib inline image size (in ")
```plt.rcParams['figure.figsize'] = [20, 15]```

## Pandas
### Format
```
pd.set_option('display.float_format', lambda x: '%f' % x)
```
### Summaries
```
df.info()
df.dtypes
```
### Indices
Reset index
```
df.reset_index(drop=True)
```
### read_csv
#### No NAs
```
df = pd.read_csv('df.csv', na_filter=False)
df = pd.read_csv('df.csv', na_values=[], keep_default_na=False)
```

## PySpark
