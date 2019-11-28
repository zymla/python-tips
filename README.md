# python-tips

## Jupyter
### Autotime
```
!pip install ipython-autotime
```
```
import autotime
%load_ext autotime
```
### Matplotlib inline image size (in ")
```plt.rcParams['figure.figsize'] = [20, 15]```
### Prompt for password
```
import getpass
password = getpass.getpass()
```
## General
### Iterable unpacking
Will call `print()` with three parameters instead of calling it with one list
```
ls = [0, 1, 2]
print(*ls)
``` 

### Dictionaries
#### Is key in dict?
```
'key' in {'key': 'value'}
```
#### Create dictionary from two lists
```
dict(zip(keys, values))
```
#### Dictionary of lists to list of dictionaries
```
def dictlist2listdict(dl):
    return [dict(zip(dl, i)) for i in list(zip(*dl.values()))]
```

### Regex
#### Keep elements of list matching regex pattern
```
list(filter(re.compile('regex_pattern').search, list_of_strings))
```
#### Exclude elements of list matching regex pattern
```
list(filter(lambda x: re.compile('regex_pattern').search(x) is None, list_of_strings))
```
#### Apply replace with regex to list of strings
```
list(map(lambda f: re.sub('replace_this', 'with_that', f), list_of_strings))
```
#### Named groups to dict
```
re.match(r'^patter(?P<group_a>\d+)_(?P<group2>\w+)', x).groupdict()
```

## Files

### Touch
`os.mknod(os.path.join(path, '.done'))`

### String to file handle
```
io.StringIO('some string')
```
### JSON
#### Read json files
```
with open('./file.json', encoding='UTF8') as json_file:
    data = json.load(json_file)

data_pd = pd.io.json.json_normalize(data)
```

#### Read multiple json files from directory
```
data =  []
for f in files:
    with open(os.path.join(files_path, f), 'r') as json_file:
        data.append({'file': f, 'content': json.load(json_file)})
```

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

### `value_counts()`
```
df['col'].value_counts(dropna=False)
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

### Strip whitespaces on all columns
`.apply(lambda col: col.str.strip())`

### Mutates
#### Chaining method to add a column similar to pySpark's `.withcolumn('col_name', lit('value'))`
```
df.assign(new_col='value')
df.assign(new_col=lambda df: df['old_col']+1)
```
#### Add column total sum by group
```
df = df.merge(df.groupby('grouping_var', as_index=False)['nb'].sum().sort_values('nb', ascending = False).rename(columns={'nb': 'total'}))
```

#### cumsum by group
```
df['cumsum']=df.groupby('group_var')['n'].cumsum()
```

#### String to `datetime`
```
df['datetime'] = pd.to_datetime(df['string_datetime'], errors='coerce')
```
```
df['datetime'] = df['datetime_string'].map(dateutil.parser.parse)
df['hour'] = df['datetime'].map(lambda x: x.hour)
df['doy'] = df['datetime'].map(lambda x: x.dayofyear)
```
#### String to numeric
```
pd.to_numeric(df['col'], errors='coerce')
```
### Filters
#### Strings
```
df[df["string_column"].str.startswith('some patern', na = False)]
df[df["string_column"].str.contains('some patern', na = True)]
df[df["string_column"].str.contains('some|patern$', regex = True)]
df[df["string_column"].str.contains('some|patern$', regex = True, flags=re.IGNORECASE)]
```

### Explode
**Whatch out** `df.explode` is based on index, therefore it is advisable to `reset_index` before calling it
```
df['col_of_list'].explode()
df.explode('col_of_list')
```

## PySpark SQL vs Pandas
| Pandas                                   | PySpark SQL                                     |
| ---------------------------------------- | ----------------------------------------------- |
| df[df['col_a'] in ['val a', 'val b']]    | df.filter(col('col_a').isin(['val a', 'val b']) |
| df.groupby('g').agg({'col_a': 'any'})    | df.groupBy('g').agg({'col_a': 'max'})           |

## PySpark
### pySpqrk SQL
`sql()` can only take one single instruction, no `;`
```
sqlContext = SQLContext(sc)
sqlContext.sql('USE database_name')
```
### Refer to a column with `col()` when chaining
```
from pyspark.sql.functions import col

df.filter(col('col_name').isin(['a', 'b']))
```

### Rename multiple columns at once
```
df.select([col(c).alias(old_name_new_name_dict.get(c, c)) for c in old_name_new_name_dict])
```

### create a udf wrapper and then apply it to the columns
```
from pyspark.sql.types import StringType
from pyspark.sql.functions import udf

def map_func(col1_val, col2_val):
   return col1_val + '-' + col2_val

df2 = df.withColumn('new_field', udf(map_func, StringType())(df.col1, df.col2))
```

### Check what substrings are in a string column
```
sqlContext.sql('SELECT string_list FROM data_table').rdd.flatMap(lambda x: x['string_list'].split(' ')).countByValue()
```

### Add a method to `DataFrame`
```
from pyspark.sql.dataframe import DataFrame

def new_method(self, params):
    return self;
DataFrame.new_method = new_method
```

### Coalesce to constant (replace Null values by a constant)
```
from pyspark.sql.functions import *

df.withColumn('new_col', coalesce(df['old_col'], lit('some constant value'))
```
### Partition over
#### Create a Window
```
from pyspark.sql.window import Window
w = Window.partitionBy(df.id).orderBy(df.time)
```
#### Use partition
```
import pyspark.sql.functions as F
df = df.withColumn("timeDelta", df.time - F.lag(df.time,1).over(w))
```
### Misc Spark issues
#### Kerberos shell commands
- `klist` (to check ticket )
- `kdestroy` (to destroy ticket)
- `kinit` (to create new ticket)

# Misc
### Get dates of DST switch
```
list(
  map(
    lambda x: x.isoformat(), 
    map(
      datetime.datetime.date, 
      list(filter(
        lambda x: (x.year >= 2012) & (x.year <= 2022) & (x.month == 3), 
        pytz.timezone('Europe/Paris')._utc_transition_times
      ))
    )
  )
)
```

# Command line template with `click`
```
import json
import click
import time

@click.command()
@click.option('--input-file', type=click.File('rt'), default='-', show_default=True)
@click.option('--config-file', type=click.File('rt'), required=True)
@click.option('--output-file', type=click.File('at'), default='-', show_default=True)
def my_fun(input_file, config_file, output_file):
  """Comments"""

  click.echo(time.asctime() + " Reading config file", err=True)
  config_data = json.load(config_file)

  click.echo(time.asctime() + " Reading input file", err=True)
  input_data = json.load(input_file)

  click.echo(time.asctime() + " Writing output_file", err=True)
  output_file.write('[\n')
  json.dump(input_data[0], output_file)
  with click.progressbar(input_data, file = click.get_text_stream('stderr')) as bar: 
    for m in bar:
      output_file.write(',\n')
      json.dump(m, output_file)
  output_file.write(']')

  click.echo(time.asctime() + " Done.", err=True)

if __name__ == '__main__':
  my_fun()
```
# Sftp
```
import pysftp

with pysftp.Connection(host=hostname, username=username, private_key=pk_path, private_key_pass=getpass.getpass()) as sftp:
    sftp.get_d(remote_path, local_path)
```
