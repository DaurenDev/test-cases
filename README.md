# Content

1. [Introduction](#Introduction)
2. [Data](#Datatable)
3. [Rules and exceptions to apply](#Rules)
4. [Test cases](#Test-cases)

## Introduction
The decision-making system (DMS) is used by E-Market, considering products it has to offer, thus giving corresponding recommendations of products to buy. The relevance of items-for-sale depends on rules and exceptions that test coverage contains in itself, which means users will get recommendations based upon standards of the DMS. 
<br>
<br>
**Note:** The relevance of items is not determined by user's personal preferences.
<br>
<br>
## Datatable
In order to determine whether an item is relevant or not, we need a data source from which information will be retrieved and tested using test cases. The table shown below will provide with necessary data sets:
| group_id        | item                                       | amount | score | identity_score | risk_strategy_type       | payment_is_test | product_type       | decision | stop_factors |
|-----------------|--------------------------------------------|--------|-------|----------------|--------------------------|-----------------|--------------------|----------|--------------|
| 6ef811855e53    | photo_camera                               | 1300   | 410   | 105            | high                     | False           | electronics        |          |              |
| 5a9d3f7292a7    | bathtub                                    | 800    | 390   | 85             | high                     | False           | bathroom_objects   |          |              |
| 4bc72a689e31    | ear_plugs                                  | 1500   | 200   | 90             | high                     | False           | mixed              |          |              |
| 7cf9462101b2    | book                                       | 2000   | 150   | 78             | high                     | True            | mixed              |          |              |
| 2de8a473a0d6    | refrigerator                               | 1100   | 350   | 50             | high                     | False           | kitchen utensils   |          |              |
| 3af927e4b569    | vacuum_cleaner                             | 1000   | 100   | 22             | high                     | False           | home appliances    |          |              |

<br>
<br>

## Rules
Rules and exceptions are represented as condition expressions with operands (columns/fields) respectively.
<br>
<br>
**Rules**
<br>
- SC_N

  ```python
  if group_id == "6ef811855e53" or group_id == "5a9d3f7292a7" 
    and item in ("video_camera", "photo_camera", "dining_table", "bathtub", "baby_product")
    and (
      (score <= 350) or
      (score <= 400 and amount > 600) or
      (score <= 420 and amount > 1200) or
      (identity_score > 100 and amount > 750 and score <= 420)
      ):
  
    then 
      decision = "reject" and stop_factors = 'SC_N';
  ```
<br>
  
- EXP_D (not often bought, unpopular)

    ```python
  if (group_id == "4bc72a689e31" or group_id == "7cf9462101b2")
    and (score <= 200 and amount > 1200)
    and item in ("battery", "book", "lamp", "ear_plugs", "tea_pot"):
    
      then 
        decision = "reject" and stop_factors = 'EXP_D';
    ```
<br>

- LOW_R (low rate)

  ```python
  if (group_id == "2de8a473a0d6" or group_id == "3af927e4b569")
    and (score <= 100 and amount >= 1000)
    and item in ("photo_camera", "bathtub", "vacuum_cleaner", "refrigerator", "vacuum_cleaner"):
    
      then 
        decision = "reject" and stop_factors = 'LOW_R';
    ```
<br>

**Exceptions**

```python
if risk_strategy_type in ("light", "middle") or payment_is_test == True or product_type == "installments":
```

<br>

## Test-cases
***Test cases will be executed based on given rules and table shown above in order to determine values of "decision" and "stop_factors" for each row, which means every record on the table will be evaluated through these rules.***

<br>

**Case 1:**<br> Due to the high percentage of fraud, a record with the "photo_camera" item should be handled with the "SC_N" rule since the "identity_score" value is greater than 100. The exceptions should not be followed.

<br>

***Steps:***
- Determine the table that will be used for update (line 1)
  <br>
  ```sql
  UPDATE records
  ```
  <br>
- Use SET clause to choose record fields to be changed thus determining record's status (line 2):<br>
  "decision" field value should be set to "reject"<br>
  "stop_factors" field value should be set to "SC_N"
  <br>
  ```sql
  UPDATE records
  SET decision = 'reject', stop_factors = 'SC_N'
  ```
  <br>
- Use WHERE clause and AND operator to define multiple conditions that should be met in order to update the found record. In this case, conditions of the "SC_N" rule are used (lines 3-10):
  <br>
  ```sql
  UPDATE records
  SET decision = 'reject', stop_factors = 'SC_N'
  WHERE (group_id = '6ef811855e53' OR group_id = '5a9d3f7292a7') -- choosing 1st and 2nd rows in the table;
    AND item IN ('video_camera', 'photo_camera', 'dining_table', 'bathtub', 'baby_product')
    AND (
      (score <= 350) OR
      (score <= 400 AND amount > 600) OR
      (score <= 420 AND amount > 1200) OR
      (identity_score > 100 AND amount > 750 AND score <= 420)
    )
  ```
  <br>
- Use NOT operator to ensure that the found record doesn't match any predefined exception (line 11):
  ```sql
    UPDATE records
    SET decision = 'reject', stop_factors = 'SC_N'
    WHERE (group_id = '6ef811855e53' OR group_id = '5a9d3f7292a7') -- choosing 1st and 2nd rows in the table;
      AND item IN ('video_camera', 'photo_camera', 'dining_table', 'bathtub', 'baby_product')
      AND (
      (score <= 350) OR
      (score <= 400 AND amount > 600) OR
      (score <= 420 AND amount > 1200) OR
      (identity_score > 100 AND amount > 750 AND score <= 420)
    )
      AND NOT (risk_strategy_type IN ('light', 'middle') OR payment_is_test = True OR product_type = 'installments');
  ```
  <br>
- Display the result of the found (if it is) and updated record:
  ```sql
  SELECT *
  FROM records
  WHERE 
    (group_id = '6ef811855e53' OR group_id = '5a9d3f7292a7') -- choosing 1st and 2nd rows in the table;
    AND decision = 'reject'
    AND stop_factors = 'SC_N'
  ```
<br>

***Expected result:*** The first row of the table should be returned:<br>
- The "decision" field value is "reject"
- The "stop_factors" field value is "SC_N"<br>

| group_id        | item                                       | amount | score | identity_score | risk_strategy_type       | payment_is_test | product_type       | decision | stop_factors |
|-----------------|--------------------------------------------|--------|-------|----------------|--------------------------|-----------------|--------------------|----------|--------------|
| 6ef811855e53    | photo_camera                               | 1300   | 410   | 105            | high                     | False           | electronics        |          |              |


<br>

**Case 2:**<br> Two records that have low indicators of the "score" value and high indicators of the "amount" value should be handled by the "EXP_D" rule since these products have decreased in purchase frequency. The exceptions should handle 1 record. 

***Steps:***
- Determine the table that will be used for update (line 1)
  <br>
  ```sql
  UPDATE records
  ```
  <br>
- Use SET clause to choose record fields to be changed thus determining record's status (line 2):<br>
  "decision" field value should be set to "reject" (if there's an exception - nothing should be entered) <br>
  "stop_factors" field value should be set to "EXP_D"
  <br>
  ```sql
  UPDATE records
  SET decision = 'reject', stop_factors = 'EXP_D'
  ```
  <br>
- Use WHERE clause and AND operator to define multiple conditions that should be met in order to update the found record. In this case, conditions of the "EXP_D" rule are used (lines 3-5):
  <br>
  ```sql
  UPDATE records
  SET decision = 'reject', stop_factors = 'EXP_D'
  WHERE (group_id = '4bc72a689e31' OR group_id = '7cf9462101b2') -- choosing 3rd and 4th rows in the table;
    AND (score <= 200 AND amount > 1200)
    AND item IN ('battery', 'book', 'lamp', 'ear_plugs', 'tea_pot'))
  ```
  <br>
- Use NOT operator to ensure that the found record doesn't match any predefined exception (line 7): <br>
  ```sql
    UPDATE records
    SET decision = 'reject', stop_factors = 'EXP_D'
    WHERE (group_id = '4bc72a689e31' OR group_id = '7cf9462101b2') -- choosing 3rd and 4th rows in the table;
      AND (score <= 200 AND amount > 1200)
      AND item IN ('battery', 'book', 'lamp', 'ear_plugs', 'tea_pot')
    )
      AND NOT (risk_strategy_type IN ('light', 'middle') OR payment_is_test = True OR product_type = 'installments');
  ```
  <br>
- Update (by the separate query) record which has a match for any predefined exception: <br>
  ```sql
  UPDATE records
  SET stop_factors = 'EXP_D'
  WHERE (risk_strategy_type IN ('light', 'middle') OR payment_is_test = True OR product_type = 'installments')
    AND (group_id = '4bc72a689e31' OR group_id = '7cf9462101b2')
    AND (score <= 200 AND amount > 1200)
    AND item IN ('battery', 'book', 'lamp', 'ear_plugs', 'tea_pot');  
  ```
  <br>
- Display the result of the found (if they are) and updated records:
  ```sql
  SELECT *
  FROM records
  WHERE 
    (group_id = '6ef811855e53' OR group_id = '5a9d3f7292a7')
    AND (decision = 'reject' AND stop_factors = 'EXP_D')
    AND (decision IS NULL AND stop_factors = 'EXP_D')
  ```
  <br>
  
***Expected result:*** Two rows should be returned:<br><br>
For the first row (handled by the rule): <br>
  - The "decision" field value is "reject"
  - The "stop_factors" field value is "EXP_D"
  <br>
  
For the second row (handled by the exception): <br>
  - The "decision" field value is "NULL" 
  - The "stop_factors" field value is "EXP_D" <br>

| group_id        | item                                       | amount | score | identity_score | risk_strategy_type       | payment_is_test | product_type       | decision | stop_factors |
|-----------------|--------------------------------------------|--------|-------|----------------|--------------------------|-----------------|--------------------|----------|--------------|
| 4bc72a689e31    | ear_plugs                                  | 1500   | 200   | 90             | high                     | False           | mixed              | reject   | EXP_D        |
| 7cf9462101b2    | book                                       | 2000   | 150   | 78             | high                     | True            | mixed              | NULL     | EXP_D        |
<br>


**Case 3: LOW_R**<br> With negative reviews, that are measured by the low value of the "score" field and the high value of the "amount" field, the record with the "vacuum_cleaner" item should be handled with the "LOW_R" rule since the "score" value is lesser than 100. The exceptions should not be followed.

<br>

***Steps:***
- Determine the table that will be used for update (line 1)
  <br>
  ```sql
  UPDATE records
  ```
  <br>
- Use SET clause to choose record fields to be changed thus determining record's status (line 2):<br>
  "decision" field value should be set to "reject"<br>
  "stop_factors" field value should be set to "LOW_R"
  <br>
  ```sql
  UPDATE records
  SET decision = 'reject', stop_factors = 'LOW_R'
  ```
  <br>
- Use WHERE clause and AND operator to define multiple conditions that should be met in order to update the found record. In this case, conditions of the "LOW_R" rule are used (lines 3-5):
  <br>
  ```sql
  UPDATE records
  SET decision = 'reject', stop_factors = 'LOW_R'
  WHERE (group_id = '2de8a473a0d6' OR group_id = '3af927e4b569')
    AND (score <= 100 AND amount >= 1000)
    AND item IN ('photo_camera', 'bathtub', 'vacuum_cleaner', 'refrigerator', 'vacuum_cleaner')
  ```
  <br>
- Use NOT operator to ensure that the found record doesn't match any predefined exception (line 11):
  ```sql
    UPDATE records
    SET decision = 'reject', stop_factors = 'LOW_R'
    WHERE (group_id = '2de8a473a0d6' OR group_id = '3af927e4b569') -- choosing 5th and 6th rows in the table;
      AND (score <= 100 AND amount >= 1000)
      AND item IN ('photo_camera', 'bathtub', 'vacuum_cleaner', 'refrigerator', 'vacuum_cleaner')
  
      AND NOT (risk_strategy_type IN ('light', 'middle') OR payment_is_test = True OR product_type = 'installments');
  ```
  <br>
- Display the result of the found (if it is) and updated record:
  ```sql
  SELECT *
  FROM records
  WHERE 
    (group_id = '2de8a473a0d6' OR group_id = '3af927e4b569') -- choosing 5th and 6th rows in the table;
    AND decision = 'reject'
    AND stop_factors = 'LOW_R'
  ```
<br>

***Expected result:*** The last row of the table should be returned:<br>
- The "decision" field value is "reject"
- The "stop_factors" field value is "LOW_R"<br>

| group_id        | item                                       | amount | score | identity_score | risk_strategy_type       | payment_is_test | product_type       | decision | stop_factors |
|-----------------|--------------------------------------------|--------|-------|----------------|--------------------------|-----------------|--------------------|----------|--------------|
| 3af927e4b569    | vacuum_cleaner                             | 1000   | 100   | 22             | high                     | False           | home appliances    | reject   | LOW_R        |

<br>

Now, when all rows are evaluated with rules, we can fill in those records where rules weren't applied to:
```sql
UPDATE records
SET decision = 'approve', stop_factors = NULL
WHERE decision IS NULL;
```

**Updated table**<br>

The overall result of updated records in the table would like this <br>

| group_id        | item                                       | amount | score | identity_score | risk_strategy_type       | payment_is_test | product_type       | decision | stop_factors |
|-----------------|--------------------------------------------|--------|-------|----------------|--------------------------|-----------------|--------------------|----------|--------------|
| 6ef811855e53    | photo_camera                               | 1300   | 410   | 105            | high                     | False           | electronics        | reject   | SC_N         |
| 5a9d3f7292a7    | bathtub                                    | 800    | 390   | 85             | high                     | False           | bathroom_objects   | approve  | NULL         |
| 4bc72a689e31    | ear_plugs                                  | 1500   | 200   | 90             | high                     | False           | mixed              | reject   | EXP_D        |
| 7cf9462101b2    | book                                       | 2000   | 150   | 78             | high                     | True            | mixed              | NULL     | EXP_D        |
| 2de8a473a0d6    | refrigerator                               | 1100   | 350   | 50             | high                     | False           | kitchen utensils   | approve  | NULL         |
| 3af927e4b569    | vacuum_cleaner                             | 1000   | 100   | 22             | high                     | False           | home appliances    | reject   | LOW_R        |
