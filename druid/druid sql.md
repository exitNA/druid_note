# druid sql by spark druid olap

---
## get package
1. get source code by ```git clone https://github.com/SparklineData/spark-druid-olap.git```
2. build and assembly by ```sbt assembly```
3. copy `scripts/*.sh` to spark `sbin/`

### start spark-druid-olap
```./sbin/start-sparklinedatathriftserver.sh ~/lab/spark-druid-olap/spl-accel-assembly-0.5.0-SNAPSHOT.jar --master spark://<host>:<port>```


### create base table
```
CREATE TABLE if not exists wikitickerBase (
    channel string,
    cityName string,
    comment string,
    countryIsoCode string,
    countryName string,
    isAnonymous string,
    isMinor string,
    isNew string,
    isRobot string,
    isUnpatrolled string,
    metroCode string,
    namespace string,
    page string,
    regionIsoCode string,
    regionName string,
    user string,
    commentLength string,
    deltaBucket string,
    flags string,
    diffUrl string,
    timestamp string,
    added integer,
    deleted integer,
    delta integer
);
```

### create druid table
```
CREATE TABLE if not exists wikiticker
    USING org.sparklinedata.druid
    OPTIONS (sourceDataframe "wikitickerBase",
        timeDimensionColumn "timestamp",
        druidDatasource "wikiticker",
        druidHost "<host>",
        zkQualifyDiscoveryNames "true",
        starSchema '{"factTable" : "wikiticker","relations" : []}');
```

## query by sql

### use beeline
- first go to spark home by `cd spark-2.0.2-bin-hadoop2.7`
- run beeline by `./bin/beeline`
- then in the `beeline>` client connect spark-druid-olap by
```
!connect jdbc:hive2://<host>:10000
```
that's all, you have connected to spark-druid-olap, and you can query whatever you like.

### use python with impyla
as [spark sql][1] can act as Distributed SQL Engine which implemented HiveServer2 in Hive 1.2.1. we can use impyla to connect the spark-sql-druid, then we can use python/pandas play with data.
1. install [impyla][2]
2. in your jupyter notebook
import impyla package
```
from impala.dbapi import connect
```
connect to your spark Thrift JDBC/ODBC server
```
conn = connect(host='hostname', port=10000, auth_mechanism='PLAIN')
cursor = conn.cursor()
```
do some query
```
cursor.execute('''
select ts, count(*) from wikiticker where ts >= "2016-12-07T10:00+0800" and ts < "2016-12-08T00:00+0800" group by ts''')
```
you can get the result
```
results = cursor.fetchall()
```
or you can get into pandas's dataframe
```
from impala.util import as_pandas
df = as_pandas(cursor)
```



  [1]: http://spark.apache.org/docs/latest/sql-programming-guide.html#running-the-thrift-jdbcodbc-server
  [2]: https://github.com/cloudera/impyla