# Melakukan Analisa Electric Production Menggunakan KNIME

Workflow yang akan dijalankan pada tugas ini adalah sebagai berikut

![](/Electric-Production/screenshoot/1.png)

Workflow ini berisi 3 meta node, yaitu

``Load Data Node``

![](/Electric-Production/screenshoot/1.1.png)

``Extract date-time attributes``

![](/Electric-Production/screenshoot/1.2.png)

``Aggregation and time series``

![](/Electric-Production/screenshoot/1.3.png)

### 1 Business Understanding
Data yang digunakan pada workflow ini adalah data Electric Production, dimana data tersebut berisi data produksi peralatan elektrik untuk pabrik industri per hari dari tahun 1985 hingga awal tahun 2018. Dalam data ini terdapat nilai produksi per harinya, sehingga kita dapat mengolahnya menjadi sebuah informasi, seperti rata-rata per tahun, dsb.


### 2 Data Understanding
Data ini terdiri dari 2 kolom, yaitu:

- Date, merupakan variabel string yang berisi informasi waktu
- IPG2211A2N, merupakan variabel floating point yang berisi informasi nilai produksi pada Date yang bersesuaian

### 3 Data Preparation

![](/Electric-Production/screenshoot/2.png)

Pada data preparation kita akan mempersiapkan datanya. Node yang dijalankan pertama kali adalah file manager, yaitu melakukan load data Electric Production, kemudian membuat local big data env, dan menjalankan Meta Node ``Load Data``.

![](/Electric-Production/screenshoot/3.png)

Melakukan load data Electric Production.

![](/Electric-Production/screenshoot/1.1.png)

Meta node ``Load Data`` berisi 3 node, dimana node pertama adalah penambahan kolom productionID, dimana kita menambahkan kolom tersebut untuk memberikan id di tiap record.

![](/Electric-Production/screenshoot/4.png)

Lalu, kita melakukan pembuatan table pada hive dan setelah itu melakukan load table yang telah dibuat, hasil table yang telah di buat adalah sebagai berikut.

![](/Electric-Production/screenshoot/5.png)

Setelah melakukan load data dan membuatnya menjadi table hive, kita ubah table hive tadi menjadi spark dengan menjalankan node ``Hive to Spark``. Hasil dari table spark tersebut adalah

![](/Electric-Production/screenshoot/6.png)


### 4 Modeling
Selanjutnya adalah melakukan modeling untuk merubah isi table yang ada, dengan melakukan pemecahan data untuk dilakukan analisa, workflow yang dijalankan adalah

![](/Electric-Production/screenshoot/10.png)

Pertama kita akan menjalankan meta node ``Extract date-time attributes``, dimana meta node ini akan melakukan pemisahan data yang nantinya akan di lakukan analisa,

![](/Electric-Production/screenshoot/1.2.png)

Ada 3 tahapan pada node ini, dimana tahapan tersebut semua menggunakan node ``Spark SQL Query``, namun dengan pengaturan yang berbeda. Tahap pertama berisi:

![](/Electric-Production/screenshoot/7.1.png)

Pada tahap ini, kita melakukan query select data yang berada pada table, lalu melakukan konversi data Date. Hasilnya adalah sebagai berikut:

![](/Electric-Production/screenshoot/7.png)

Setelah kita mendapatkan kolom eventDate, kita ekstraksi kolom tersebut untuk mendapatkan tahun, bulan, minggu, hari, dengan menjalankan ``Spark SQL Query`` tahap kedua,

![](/Electric-Production/screenshoot/8.1.png)

Hasil dari query tahap kedua adalah sebagai berikut:

![](/Electric-Production/screenshoot/8.png)

Tahap ketiga adalah melakukan klasifikasi dari kolom dayOfWeek, menggunakan query sebagai berikut:

![](/Electric-Production/screenshoot/9.1.png)

Query ini akan melakukan pembuatan column baru bernama dayOfClassifier, dimana berisi nilai WE apabila dayOfWeek bernilai ('Saturday', 'Sunday'), atau berisi nilai BD apabila selain ('Saturday', 'Sunday'), sehingga mendapatkan hasil:

![](/Electric-Production/screenshoot/9.png)

Semua node telah dijalankan, dan hasil column nantinya akan dilakukan analisa pada meta node ``Aggregation and time series``, meta node ini berisi sejumlah node seperti berikut

![](/Electric-Production/screenshoot/1.3.png)

Pada meta node ini, kita menerima input dan menyimpannya didalam memory sementara menggunakan node Persist Spark Dataframe/RDD.

![](/Electric-Production/screenshoot/14.png)

Lalu data tersebut dihitung rata-ratanya per segment yang sesuai (tahun, bulan, hari, dsb..) menggunakan maksimal 3 node yaitu Spark GroupBy, Spark Pivot dan Spark Column Rename.

![](/Electric-Production/screenshoot/15.png)

Menghitung usage keseluruhan dan menghitung rata-rata per segment tahun.

![](/Electric-Production/screenshoot/15.1.png)

Menghitung rata-rata per segment bulan.

![](/Electric-Production/screenshoot/15.2.png)

Menghitung rata-rata per segment minggu.

![](/Electric-Production/screenshoot/15.3.png)

Menghitung rata-rata per segment hari di 1 minggu.

![](/Electric-Production/screenshoot/15.4.png)

Menghitung rata-rata per segment harian.

![](/Electric-Production/screenshoot/15.6.png)

Menghitung rata-rata per segment klasifikasi hari.

Setelah itu, data rata-rata tersebut dijoin menggunakan node Spark Joiner dan diteruskan ke general workflow.

Hasil akhir dari table yang telah selesai di proses adalah sebagai berikut:

![](/Electric-Production/screenshoot/16.png)

Setelah mendapatkan data tersebut, kita hitung persentase nya per minggu / hari sesuai dengan segmentnya, contohnya apabila segmentnya hari, maka dihitung presentasinya per minggu. Perhitungan ini menggunakan node SQL Spark Query dengan syntax:

![](/Electric-Production/screenshoot/17.1.png)

Hasilnya adalah sebagai berikut:

![](/Electric-Production/screenshoot/17.png)

Dari hasil diatas, masih terdapat beberapa missing value, untuk itu, kita menambahkan node Spark Missing Value untuk mengganti nilai missing value dengan nilai 0. Berikut hasil tabelnya:

![](/Electric-Production/screenshoot/18.png)

### 5 Evaluation
Pada evaluation akan dijalankan workflow

![](/Electric-Production/screenshoot/19.png)

Dapat dilihat pada node Normalizer, semua data kecuali ID, dinormalisasi menjadi range 0 - 1. Pada node setelah Denormalizer data dioutputkan menjadi 2 bentuk yaitu visualisasi dan data table yang diteruskan ke general workflow. Selanjutnya, data output tadi dimasukkan kembali ke Local Big Data Environment menggunakan 2 node, yaitu Spark to Hive untuk load menjadi Apache Hive dan Spark to Parquet untuk load menjadi HDFS.

Hasilnya adalah sebagai berikut:

![](/Electric-Production/screenshoot/20.png)
![](/Electric-Production/screenshoot/20.1.png)


### 6 Deployment
Selanjutnya pada tahap deployment kita akan menjalankan workflow

![](/Electric-Production/screenshoot/21.png)

Pada tahap ini dilakukan perubahan data dari spark kembali mejadi hive serta menyimpan spark kedalam HDFS dalam bentuk parquet, hasil dari data tersebut adalah sebagai berikut

![](/Electric-Production/screenshoot/22.png)
