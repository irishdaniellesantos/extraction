### RAFM Data Extraction

<details>
  <summary>OCS RAFM Data Extraction (Top-Ups Jan–Apr 2026)</summary>

<br>

<details>
  <summary> Request Summary </summary>

### **Request Summary**

| **Field**        | **Details**                                                                                                 |
| ---------------- | ----------------------------------------------------------------------------------------------------------- |
| Request ID       | REQ2468669                                                                                                  |
| RITM Number      | RITM2536242                                                                                                 |
| Request Type     | One-Time Data Extraction (BI / Analytics / Big Data)                                                        |
| Business Unit    | Fraud Management                                                                                            |
| Sub-unit         | SMART                                                                                                       |
| Application      | Data Hub                                                                                                    |
| Description      | Extraction of RAFM-related data from `u_oocs_vou`, `u_oocs_mon`, `u_oocs_cm`                                |
| Scope & Criteria | MIN-based filtering with date coverage from **2026-01-01 to 2026-04-16**                                    |
| Output Details   | File Name: RAFM_TopUpsJanApr<br>Format: Text File<br>Delimiter: Tab<br>Content Format: RAFM_VOU_format.xlsx |
| Purpose          | Data extraction for analysis and investigation                                                              |

  
</details>


### **Process Overview**

1. **MIN List Preparation** <br>
   Downloaded the list of MINs and imported it into ROC (`oracle.rocref.temp_min_list`) to be used as a reference table for filtering.

2. **Table Validation** <br>
   Identified the required source tables:

   * `u_oocs_vou`
   * `u_oocs_mon`
   * `u_oocs_cm`

   Checked each table in Presto to confirm that the requested date range (**Jan 1, 2026 – Apr 16, 2026**) is available.

3. **Field Validation** <br>
   Used `DESC <table_name>` to verify that the required fields match the provided file format (`RAFM_VOU_format.xlsx`) and exist in the tables.

4. **Data Extraction Query** <br>
   Executed the following query in Presto to extract and aggregate the required data:<br>
   > The same query structure was also applied to other OCS tables (u_oocs_vou, u_oocs_mon) since they share the same field names.

   ```sql
   SELECT 
       ocs_msisdn, 
       SUM(ocs_amount) AS total_ocs_amount, 
       MIN(ocs_time_stamp) AS min_ocs_time_stamp, 
       MAX(ocs_time_stamp) AS max_ocs_time_stamp, 
       ocs_source_channel, 
       ocs_plan_code_desc, 
       ocs_sub_brand_name, 
       ocs_other_party_number 
   FROM u_oocs_cm a
   INNER JOIN oracle.rocref.temp_min_list b 
       ON SUBSTR(a.ocs_msisdn, -10) = SUBSTR(b.msisdn, -10)
   WHERE partition_col BETWEEN DATE '2026-01-01' AND DATE '2026-04-16'
   GROUP BY 
       ocs_msisdn, 
       ocs_source_channel, 
       ocs_plan_code_desc, 
       ocs_sub_brand_name, 
       ocs_other_party_number;
   ```

6. **Extraction Execution (Default Path)** <br>
   After validating the query, exited Presto and executed the extraction script:

   ```bash
   ./extraction.sh "<QUERY>" "RAFM_TopUpsJanApr"
   ```

7. **Alternative Execution (Custom HDFS Path)** <br>
   If saving to a specific directory, navigated to the target path:

   ```bash
   cd /ifs/hdfs/datahub/uds/rafm/adhoc
   ```

   Then executed the script using:

   ```bash
   ~/idhs/extraction.sh "<QUERY>" "RAFM_TopUpsJanApr"
   ```

8. **File Retrieval** <br>
   After the extraction completes:

   * Located the output file (tab-delimited `.text` file) in the target directory
   * Used the folder terminal interface to download the file for delivery via email

</details>

