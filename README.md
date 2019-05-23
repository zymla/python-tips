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

### Coalesce to constant (replace Null values bya constant)
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
