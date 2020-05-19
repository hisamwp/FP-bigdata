# Melakukan Analisa Sales of a Shampoo over three years Menggunakan KNIME

Workflow yang akan dijalankan pada tugas ini adalah sebagai berikut

![](/Sales-of-Shampoo/Screenshoot/1.png)

Workflow ini berisi 3 meta node, yaitu

``Load Data Node``

![](/Sales-of-Shampoo/Screenshoot/1.1.png)

``Extract date-time attributes``

![](/Sales-of-Shampoo/Screenshoot/1.2.png)

``Aggregation and time series``

![](/Sales-of-Shampoo/Screenshoot/1.3.png)

### 1 Business Understanding
Data yang digunakan pada workflow ini adalah data Sales of a Shampoo over three years, dimana data tersebut berisi data penjualan sebuah shampoo selama 3 tahun. Dengan data ini kita dapat mengolahnya menjadi beberapa informasi, salah satunya rata-rata penjualan shampoo di suatu musim.

### 2 Data Understanding
Data ini terdiri dari 2 kolom, yaitu:

- Month, merupakan variabel string yang berisi informasi waktu berupa tanggal dan bulan
- Sales of a Shampoo over three year period, merupakan variabel floating point yang berisi angka penjualan shampoo per bulan.

### 3 Data Preparation

![](/Sales-of-Shampoo/Screenshoot/2.png)

Pada data preparation kita akan mempersiapkan datanya. Node yang dijalankan pertama kali adalah file manager, yaitu melakukan load data Sales of a Shampoo over three years, kemudian membuat local big data env, dan menjalankan Meta Node ``Load Data``.

![](/Sales-of-Shampoo/Screenshoot/3.png)

Melakukan load data Sales of a Shampoo over three years.

![](/Sales-of-Shampoo/Screenshoot/1.1.png)

Meta node ``Load Data`` berisi 3 node, dimana node pertama adalah penambahan kolom tempID, dimana kita menambahkan kolom tersebut untuk memberikan id di tiap record.

![](/Sales-of-Shampoo/Screenshoot/4.png)

Lalu, kita melakukan pembuatan table pada hive dan setelah itu melakukan load table yang telah dibuat, hasil table yang telah di buat adalah sebagai berikut.

![](/Sales-of-Shampoo/Screenshoot/5.png)

Setelah melakukan load data dan membuatnya menjadi table hive, kita ubah table hive tadi menjadi spark dengan menjalankan node ``Hive to Spark``. Hasil dari table spark tersebut adalah

![](/Sales-of-Shampoo/Screenshoot/6.png)


### 4 Modeling
Selanjutnya adalah melakukan modeling untuk merubah isi table yang ada, dengan melakukan pemecahan data untuk dilakukan analisa, workflow yang dijalankan adalah

![](/Sales-of-Shampoo/Screenshoot/10.png)

Pertama kita akan menjalankan meta node ``Extract date-time attributes``, dimana meta node ini akan melakukan pemisahan data yang nantinya akan di lakukan analisa,

![](/Sales-of-Shampoo/Screenshoot/1.2.png)

Ada 3 tahapan pada node ini, dimana tahapan tersebut semua menggunakan node ``Spark SQL Query``, namun dengan pengaturan yang berbeda. Tahap pertama berisi:

![](/Sales-of-Shampoo/Screenshoot/7.1.png)

Pada tahap ini, kita melakukan query select data yang berada pada table, lalu melakukan konversi data Month. Hasilnya adalah sebagai berikut:

![](/Sales-of-Shampoo/Screenshoot/7.png)

Setelah kita mendapatkan kolom eventDate, kita ekstraksi kolom tersebut untuk mendapatkan tahun dan bulan, dengan menjalankan ``Spark SQL Query`` tahap kedua,

![](/Sales-of-Shampoo/Screenshoot/8.1.png)

Hasil dari query tahap kedua adalah sebagai berikut:

![](/Sales-of-Shampoo/Screenshoot/8.png)

Tahap ketiga adalah melakukan klasifikasi dari kolom month, dimana saya klasifikasikan berdasarkan season, menggunakan query sebagai berikut:

![](/Sales-of-Shampoo/Screenshoot/9.1.png)

Query ini akan melakukan pembuatan column baru bernama monthClassifier, dimana berisi nilai SPRING apabila 3<=month<=5, berisi nilai SUMMER apabila 6<=month<=8, berisi nilai FALL apabila 9<=month<=11, dan sisanya berisi nilai WINTER. Hasilnya sebagai berikut:

![](/Sales-of-Shampoo/Screenshoot/9.png)

Semua node telah dijalankan, dan hasil column nantinya akan dilakukan analisa pada meta node ``Aggregation and time series``, meta node ini berisi sejumlah node seperti berikut

![](/Sales-of-Shampoo/Screenshoot/1.3.png)

Pada meta node ini, kita menerima input dan menyimpannya didalam memory sementara menggunakan node Persist Spark Dataframe/RDD.

![](/Sales-of-Shampoo/Screenshoot/14.png)

Lalu data tersebut dihitung rata-ratanya per segment yang sesuai (tahun, bulan, season dsb..) menggunakan maksimal 3 node yaitu Spark GroupBy, Spark Pivot dan Spark Column Rename.

![](/Sales-of-Shampoo/Screenshoot/15.png)

Menghitung usage keseluruhan dan menghitung rata-rata per segment tahun.

![](/Sales-of-Shampoo/Screenshoot/15.1.png)

Menghitung rata-rata per segment bulan.

![](/Sales-of-Shampoo/Screenshoot/15.2.png)

Menghitung rata-rata per segment bulanan dalam 1 tahun.

![](/Sales-of-Shampoo/Screenshoot/15.3.png)

Menghitung rata-rata per segment klasifikasi bulan(season).

Setelah itu, data rata-rata tersebut dijoin menggunakan node Spark Joiner dan diteruskan ke general workflow.

Hasil akhir dari table yang telah selesai di proses adalah sebagai berikut:

![](/Sales-of-Shampoo/Screenshoot/16.png)

Setelah mendapatkan data tersebut, kita hitung persentase nya per bulan sesuai dengan segmentnya. Perhitungan ini menggunakan node SQL Spark Query dengan syntax:

![](/Sales-of-Shampoo/Screenshoot/17.1.png)

Hasilnya adalah sebagai berikut:

![](/Sales-of-Shampoo/Screenshoot/17.png)

Dari hasil diatas, masih terdapat beberapa missing value, untuk itu, kita menambahkan node Spark Missing Value untuk mengganti nilai missing value dengan nilai 0. Berikut hasil tabelnya:

![](/Sales-of-Shampoo/Screenshoot/18.png)

### 5 Evaluation
Pada evaluation akan dijalankan workflow

![](/Sales-of-Shampoo/Screenshoot/19.png)

Dapat dilihat pada node Normalizer, semua data kecuali ID, dinormalisasi menjadi range 0 - 1. Pada node setelah Denormalizer data dioutputkan menjadi 2 bentuk yaitu visualisasi dan data table yang diteruskan ke general workflow. Selanjutnya, data output tadi dimasukkan kembali ke Local Big Data Environment menggunakan 2 node, yaitu Spark to Hive untuk load menjadi Apache Hive dan Spark to Parquet untuk load menjadi HDFS.

Hasilnya adalah sebagai berikut:

![](/Sales-of-Shampoo/Screenshoot/20.png)
![](/Sales-of-Shampoo/Screenshoot/20.1.png)


### 6 Deployment
Selanjutnya pada tahap deployment kita akan menjalankan workflow

![](/Sales-of-Shampoo/Screenshoot/21.png)

Pada tahap ini dilakukan perubahan data dari spark kembali mejadi hive serta menyimpan spark kedalam HDFS dalam bentuk parquet, hasil dari data tersebut adalah sebagai berikut

![](/Sales-of-Shampoo/Screenshoot/22.png)
