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
| 6ef811855e53    | photo_camera                               | 1300   | 410   | 85             | high                     | False           | electronics        |          |              |
| 5a9d3f7292a7    | bathtub                                    | 800    | 390   | 105            | high                     | False           | installments       |          |              |
| 4bc72a689e31    | ear_plugs                                  | 1500   | 1760  | 90             | high                     | False           | mixed              |          |              |
| 7cf9462101b2    | book                                       | 2000   | 2100  | 78             | high                     | True            | mixed              |          |              |
| 2de8a473a0d6    | refrigerator                               | 1100   | 1300  | 50             | middle                   | False           | kitchen utensils   |          |              |
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
    and item in ("photo_camera", "bathtub", "vacuum_cleaner", "refrigerator", "vacuum_cleaner")
    and identity_score <= 50:
    
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
Test cases will be executed based on given rules and table shown above in order to determine values of "decision" and "stop_factors" for each row, which means every record on the table will be evaluated through these rules.

**Case 1: SC_N rule**

```sql
-- handling the first rule;
UPDATE records -- selecting the table which will be updated;
SET decision = 'reject', stop_factors = 'SC_N' -- using SET clause to determine fields for update;
WHERE (group_id = '6ef811855e53' OR group_id = '5a9d3f7292a7') -- choosing 1st and 2nd rows in the table;
  AND item IN ('video_camera', 'photo_camera', 'dining_table', 'bathtub', 'baby_product') -- condition expressions
  AND (
    (score <= 350) OR
    (score <= 400 AND amount > 600) OR
    (score <= 420 AND amount > 1200) OR
    (identity_score > 100 AND amount > 750 AND score <= 420)
  )
  AND NOT (risk_strategy_type IN ('light', 'middle') OR payment_is_test = True OR product_type = 'installments');

-- handling the exception if the first rule is triggered
UPDATE records
SET stop_factors = 'SC_N'
WHERE (risk_strategy_type IN ('light', 'middle') OR payment_is_test = True OR product_type = 'installments')
  AND (group_id = '6ef811855e53' OR group_id = '5a9d3f7292a7')
  AND item IN ('video_camera', 'photo_camera', 'dining_table', 'bathtub', 'baby_product')
  AND (
    (score <= 350) OR
    (score <= 400 AND amount > 600) OR
    (score <= 420 AND amount > 1200) OR
    (identity_score > 100 AND amount > 750 AND score <= 420)
  );
```

**Case 2: EXP_D**

```sql
-- handling the second rule;
UPDATE records -- selecting the table which will be updated;
SET decision = 'reject', stop_factors = 'EXP_D' -- using SET clause to determine fields for update;
WHERE (group_id = '4bc72a689e31' OR group_id = '7cf9462101b2') -- choosing 3rd and 4th rows in the table;
  AND (score <= 200 AND amount > 1200) -- condition expressions
  AND item IN ('battery', 'book', 'lamp', 'ear_plugs', 'tea_pot')

  AND NOT (risk_strategy_type IN ('light', 'middle') OR payment_is_test = True OR product_type = 'installments');

-- handling the exception if the second rule is triggered
UPDATE records
SET stop_factors = 'EXP_D'
WHERE (risk_strategy_type IN ('light', 'middle') OR payment_is_test = True OR product_type = 'installments')
  AND (group_id = '4bc72a689e31' OR group_id = '7cf9462101b2')
  AND (score <= 200 AND amount > 1200)
  AND item IN ('battery', 'book', 'lamp', 'ear_plugs', 'tea_pot');  
```

**Case 3: LOW_R**

```sql
-- handling the third rule;
UPDATE records -- selecting the table which will be updated;
SET decision = 'reject', stop_factors = 'LOW_R' -- using SET clause to determine fields for update;
WHERE (group_id = '2de8a473a0d6' OR group_id = '3af927e4b569') -- choosing 5th and 6th rows in the table;
  AND (score <= 100 AND amount >= 1000) -- condition expressions
  AND item IN ('photo_camera', 'bathtub', 'vacuum_cleaner', 'refrigerator', 'vacuum_cleaner')
  AND identity_score <= 50

  AND NOT (risk_strategy_type IN ('light', 'middle') OR payment_is_test = True OR product_type = 'installments');

-- handling the exception if the third rule is triggered
UPDATE records
SET stop_factors = 'LOW_R'
WHERE (risk_strategy_type IN ('light', 'middle') OR payment_is_test = True OR product_type = 'installments')
  AND (group_id = '2de8a473a0d6' OR group_id = '3af927e4b569')
  AND (score <= 100 AND amount >= 1000)
  AND item IN ('photo_camera', 'bathtub', 'vacuum_cleaner', 'refrigerator', 'vacuum_cleaner')
  AND identity_score <= 50;

```
Now, when all rows are evaluated with rules, we can fill in those records where rules weren't applied:
```sql
UPDATE example_records
SET decision = 'approve', stop_factors = NULL
WHERE decision IS NULL;
```

**Expected result**<br>
The updated version of the table would look like this:<br>
| group_id        | item                                       | amount | score | identity_score | risk_strategy_type       | payment_is_test | product_type       | decision | stop_factors |
|-----------------|--------------------------------------------|--------|-------|----------------|--------------------------|-----------------|--------------------|----------|--------------|
| 6ef811855e53    | photo_camera                               | 1300   | 410   | 85             | high                     | False           | electronics        | reject   | SC_N         |
| 5a9d3f7292a7    | bathtub                                    | 800    | 390   | 105            | high                     | False           | installments       | approve  | NULL         |
| 4bc72a689e31    | ear_plugs                                  | 1500   | 1760  | 90             | high                     | False           | mixed              | approve  | NULL         |
| 7cf9462101b2    | book                                       | 2000   | 2100  | 78             | high                     | True            | mixed              | approve  | NULL         |
| 2de8a473a0d6    | refrigerator                               | 1100   | 1300  | 50             | middle                   | False           | kitchen utensils   | approve  | NULL         |
| 3af927e4b569    | vacuum_cleaner                             | 1000   | 100   | 22             | high                     | False           | home appliances    | reject   | LOW_R        |
