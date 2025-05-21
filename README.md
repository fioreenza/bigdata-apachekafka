# Warehouse Monitoring System with Kafka and PySpark

Sistem ini memantau kondisi suhu dan kelembaban gudang secara real-time menggunakan:
- Kafka untuk komunikasi data streaming
- PySpark untuk analisis real-time
- Docker Compose untuk menjalankan Zookeeper, Kafka, Producer, dan Spark secara berurutan dan saling terhubung.

Struktur foldernya sebagai berikut
```
├── docker-compose.yml
├── producer/
│   ├── producer_suhu.py
│   └── producer_kelembaban.py
└── consumer/
    └── pyspark_consumer.py
```

File docker-compose.yml digunakan untuk menjalankan warehouse system monitoring berbasis Kafka dan Spark secara terintegrasi. Di dalamnya terdapat empat layanan utama: Zookeeper sebagai koordinator Kafka, Kafka sebagai message broker untuk menampung data sensor, Python Producer untuk mensimulasikan pengiriman data suhu dan kelembaban, serta Spark (dengan PySpark) sebagai konsumer yang memproses data secara real-time menggunakan Spark Streaming. Semua layanan berjalan dalam container yang saling terhubung, dan script producer serta consumer disimpan di folder lokal yang dipetakan ke direktori /app di dalam container. Kafka dijalankan menggunakan image confluentinc/cp-kafka:latest dalam mode klasik (dengan Zookeeper), sedangkan Spark menggunakan image bitnami/spark:3.5.0 dan dijalankan manual menggunakan spark-submit untuk memproses stream dari Kafka.

Langkah-langkah Implementasi
1.  Setup Kafka dan Zookeeper
    ```
    docker-compose up -d
    ```
    ![image](https://github.com/user-attachments/assets/c7b6c0f7-8a03-4775-a040-6761406499e6)
    Kafka berjalan di localhost:9092, dan Zookeeper di localhost:2181.
2. Buat Topik Kafka
    ```
    docker exec -it kafka bash
    ```
    ```
    kafka-topics --create --topic sensor-suhu-gudang --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
    kafka-topics --create --topic sensor-kelembaban-gudang --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
    ```
    ![image](https://github.com/user-attachments/assets/668e368b-5c6b-4e2d-8299-117fb90aeade)
3. Simulasikan Data Sensor (Producer Kafka)
    Jalankan dua skrip Python di container producer
    ```
    docker exec -it producer bash
    pip install kafka-python
    python producer_suhu.py
    python producer_kelembaban.py
    ```
    ![Screenshot 2025-05-21 214412](https://github.com/user-attachments/assets/961787b2-0d99-475b-82f3-6c3ab8e3fb43)
    ![Screenshot 2025-05-21 214425](https://github.com/user-attachments/assets/ff0551e8-d29b-446f-ae40-7ab467284c6d)
5. Konsumsi dan Olah Data dengan PySpark
    Jalankan skrip pyspark_consumer.py di container spark
    ```
    docker exec -it spark bash
    pip install kafka-python
    spark-submit --packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.0 pyspark_consumer.py
    ```
    ![Screenshot 2025-05-21 214242](https://github.com/user-attachments/assets/51772854-a845-43c7-951f-ccfbe37368ac)
    ![Screenshot 2025-05-21 214334](https://github.com/user-attachments/assets/c6c53c05-a4d0-40f8-b99c-2ccee9b96639)



Dengan sistem ini, perusahaan dapat mendeteksi secara real-time jika suatu gudang mengalami suhu tinggi, kelembaban tinggi, atau keduanya sekaligus. Hal ini sangat penting untuk menjaga kualitas penyimpanan barang sensitif dan mencegah kerugian logistik.



