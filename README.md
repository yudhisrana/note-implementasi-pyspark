
# 📊 Visualisasi Data Retail Menggunakan Hadoop, Spark, dan Python

## 1. Jalankan HDFS

```bash
start-all.cmd
```

## 2. Buat Folder di HDFS

```bash
hdfs dfs -mkdir -p /user/hadoop/retail/input
```

> Gunakan nama `retail` sesuai keinginan.

## 3. Masukkan Dataset ke HDFS

```bash
hdfs dfs -put online_retail.csv /user/hadoop/retail/input/
```

## 4. Verifikasi Data

```bash
hdfs dfs -ls /user/hadoop/retail/input/
```

## 5. Buat Folder spark-job dan Buka di VSCode

```bash
mkdir spark-job
cd spark-job
code .
```

## 6. Buat file dengan nama retail_analysis.py
```bash
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, count, sum, to_timestamp, hour
spark = SparkSession.builder.appName("RetailAnalysis").getOrCreate()
# Baca data dari HDFS
df = spark.read.csv("hdfs://localhost:9000/user/hadoop/retail/input/online_retail.csv", header=True, inferSchema=True)
# Tambahkan kolom TotalAmount
df = df.withColumn("TotalAmount", col("Quantity") * col("UnitPrice"))
# 1. Produk Terlaris
top_products = df.groupBy("Description") \
    .agg(count("Quantity").alias("TotalBeli")) \
    .orderBy(col("TotalBeli").desc())
top_products.write.mode("overwrite").csv("hdfs://localhost:9000/user/hadoop/retail/output/top_products", header=True)
# 2. Pelanggan Paling Aktif
top_customers = df.groupBy("CustomerID") \
    .agg(count("InvoiceNo").alias("TotalTransaksi")) \
    .orderBy(col("TotalTransaksi").desc())
top_customers.write.mode("overwrite").csv("hdfs://localhost:9000/user/hadoop/retail/output/top_customers", header=True)

# 3. Waktu Pembelian Tersibuk
df = df.withColumn("Hour", hour(to_timestamp("InvoiceDate", "yyyy-MM-dd HH:mm:ss")))
busy_hours = df.groupBy("Hour") \
    .agg(count("InvoiceNo").alias("JumlahTransaksi")) \
    .orderBy(col("JumlahTransaksi").desc())

busy_hours.write.mode("overwrite").csv("hdfs://localhost:9000/user/hadoop/retail/output/busy_hours", header=True)

# 4. Rata-rata Pembelian per Pelanggan
avg_purchase = df.groupBy("CustomerID") \
    .agg(sum("Quantity").alias("TotalItem")) \
    .orderBy(col("TotalItem").desc())
avg_purchase.write.mode("overwrite").csv("hdfs://localhost:9000/user/hadoop/retail/output/avg_purchase", header=True)
# 5. Produk Paling Menghasilkan
top_revenue = df.groupBy("Description") \
    .agg(sum("TotalAmount").alias("TotalRevenue")) \
    .orderBy(col("TotalRevenue").desc())
top_revenue.write.mode("overwrite").csv("hdfs://localhost:9000/user/hadoop/retail/output/top_revenue", header=True)

spark.stop()
```

## 7. Jalankan Spark Job

```bash
spark-submit retail_analysis.py
```

## 8. Verifikasi Output

```bash
hdfs dfs -ls /user/hadoop/retail/output/
```

Output folder yang dihasilkan:
- `avg_purchase`
- `busy_hours`
- `top_customers`
- `top_products`
- `top_revenue`

## 9. Lihat Isi File Output

```bash
hdfs dfs -ls /user/hadoop/retail/output/avg_purchase/
hdfs dfs -cat /user/hadoop/retail/output/avg_purchase/part-00000-xxxx.csv
```

## 10. Pindahkan ke Lokal

```bash
hdfs dfs -get /user/hadoop/retail/output/nama_folder ./nama_folder
```

> Ganti `nama_folder` sesuai folder output di HDFS.

## 11. Rename File agar Pendek dan Mudah Diakses

## 12. Buka Folder di VSCode dan Buat Virtual Environment

```bash
python -m venv .venv
.\.venv\Scriptsctivate
pip install pandas matplotlib seaborn
pip list
```

## 13. Buat dan Jalankan Visualisasi

### a. `visual_avg_purchase.py`

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

df = pd.read_csv("avg_purchase/avg_purchase.csv")
df.columns = ["CustomerID", "TotalItem"]
df = df.dropna()
df = df.sort_values(by="TotalItem", ascending=False).head(10)

plt.figure(figsize=(10, 6))
sns.barplot(data=df, x="CustomerID", y="TotalItem", palette="coolwarm")
plt.title("Top 10 Pelanggan dengan Total Item Terbanyak")
plt.xlabel("Customer ID")
plt.ylabel("Total Item")
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

```bash
python visual_avg_purchase.py
```

### b. `visual_top_products.py`

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

df = pd.read_csv("top_products/top_products.csv")
df.columns = ["Description", "TotalBeli"]
df = df.dropna()
df = df.sort_values(by="TotalBeli", ascending=False).head(10)

plt.figure(figsize=(12, 6))
sns.barplot(data=df, x="TotalBeli", y="Description", palette="mako")
plt.title("Top 10 Produk Terlaris")
plt.xlabel("Total Pembelian")
plt.ylabel("Nama Produk")
plt.tight_layout()
plt.show()
```

```bash
python visual_top_products.py
```

### c. `visual_top_customers.py`

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

df = pd.read_csv("top_customers/top_customers.csv")
df.columns = ["CustomerID", "TotalTransaksi"]
df = df.dropna()
df = df.sort_values(by="TotalTransaksi", ascending=False).head(10)

plt.figure(figsize=(10, 6))
sns.barplot(data=df, x="CustomerID", y="TotalTransaksi", palette="flare")
plt.title("Top 10 Pelanggan Paling Aktif")
plt.xlabel("Customer ID")
plt.ylabel("Jumlah Transaksi")
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

```bash
python visual_top_customers.py
```

### d. `visual_top_revenue.py`

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

df = pd.read_csv("top_revenue/top_revenue.csv")
df.columns = ["Description", "TotalRevenue"]
df = df.dropna()
df = df.sort_values(by="TotalRevenue", ascending=False).head(10)

plt.figure(figsize=(12, 6))
sns.barplot(data=df, x="TotalRevenue", y="Description", palette="crest")
plt.title("Top 10 Produk dengan Revenue Tertinggi")
plt.xlabel("Total Revenue")
plt.ylabel("Nama Produk")
plt.tight_layout()
plt.show()
```

```bash
python visual_top_revenue.py
```

### e. `visual_busy_hours.py`

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

df = pd.read_csv("busy_hours/busy_hours.csv")
df.columns = ["Hour", "JumlahTransaksi"]
df = df.dropna()
df["Hour"] = df["Hour"].astype(int)

plt.figure(figsize=(10, 5))
sns.lineplot(data=df.sort_values("Hour"), x="Hour", y="JumlahTransaksi", marker="o", linewidth=2)
plt.title("Jam Pembelian Tersibuk")
plt.xlabel("Jam (0-23)")
plt.ylabel("Jumlah Transaksi")
plt.xticks(range(0, 24))
plt.grid(True)
plt.tight_layout()
plt.show()
```

```bash
python visual_busy_hours.py
```
