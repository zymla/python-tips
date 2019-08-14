# python-tips

## Jupyter
### Matplotlib inline image size (in ")
```plt.rcParams['figure.figsize'] = [20, 15]```

## General
### Is key in dict?
```
'key' in {'key': 'value'}
```

## Files
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

### Mutates
#### String to `datetime`
```
df['datetime'] = pd.to_datetime(df['string_datetime'], errors='coerce')
```

### Filters
#### Strings
```
df[df["string_column"].str.startswith('some patern', na = False)]
```

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
