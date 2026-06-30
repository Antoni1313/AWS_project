# Batch vs Near-Real-Time Processing Comparison

## Project Goal
The project compares two weather data processing models sourced from an external REST API, running on an AWS EMR cluster and persisting data to Amazon S3:
1. **Near-Real-Time (NRT):** Continuous streaming pipeline (PySpark) with ultra-low latency.
2. **Batch:** Periodic processing of accumulated data batches on-demand.

---

## Pipeline Architecture

[REST API] ──(Python 3)──> [S3 raw (JSON)] ──(PySpark Stream/Batch)──> [S3 curated (Parquet)] ──(Pandas)──> [Charts]

1. **Ingestion:** Collect JSON data from the REST API and save it to `/raw/`.
2. **Enrichment:** Add a real-time ingestion/processing timestamp (`processing_time`).
3. **Persistence:** Convert and save data to the optimized Parquet format in `/curated/`.
4. **Cleaning & Analytics:** Deduplicate and calculate latency (processing time minus weather event time).
5. **Visualization:** Generate temperature and latency charts using `matplotlib`.

---

## Execution Steps

Run the Jupyter notebooks (on EMR) in the following order:

1. **`data_processing_nrt.ipynb` (Kernel: PySpark):**
   * It processes data into `/curated_nrt/`.
2. **`data_gathering.ipynb` (Kernel: Python 3):**
   * Fetches data from the API and saves JSONs to `/raw_nrt/`. **Leave this running in the background.**
3. **`data_processing_batch.ipynb` (Kernel: PySpark):**
   * One-time, asynchronous batch processing of the entire historical dataset on-demand.
4. **`charts.ipynb` (Kernel: Python 3):**
   * Before the first run, execute `%pip install` in a cell and restart the kernel.
   * Reads Parquet files directly from S3 and generates comparative charts.

---

## Results & Complexity

### Sample Statistics:
* **Unique records:** 35
* **Average temperature:** 17.34 °C
* **Average NRT latency:** 1.23 seconds
* **Average Batch latency:** 111.55 seconds

### Comparison Table:

| Feature | Near-Real-Time (NRT) | Batch |
| :--- | :--- | :--- |
| **Code Complexity** | High (Structured Streaming, checkpoints) | Low (static read and overwrite) |
| **Runtime** | Continuous (24/7 – constant cost) | Temporary (on-demand – lower cost) |
| **Latency** | Very low (seconds) | High (dependent on execution intervals) |
| **Deduplication** | Difficult in stream | Simple and global |

---

## Recommendation & Conclusion

* **Recommendation:** Implement a system to automatically run the Batch pipeline periodically to optimize infrastructure costs.
* **Conclusion:** The project demonstrated that the NRT architecture is crucial for immediate reaction to dynamic weather events due to second-level processing latency, while the significantly simpler and cheaper Batch structure remains the optimal solution for regular, aggregate analysis of historical data.
