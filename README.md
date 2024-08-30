# Content

1. [Introduction](#Introduction)
2. [Data](#datatable)
3. [Rules and exceptions to apply](#Rules)
4. [Test cases](#conclusion)

## Introduction
The decision-making system (DMS) is used by E-Market, considering products it has to offer, thus giving corresponding recommendations of products to buy. The relevance of items-for-sale depends on rules and exceptions that test coverage contains in itself, which means users will get recommendations based upon standards of the DMS. 
<br>
<br>
**Note:** The relevance of items is not determined by user's personal preferences.
<br>
<br>
## Data
In order to determine whether an item is relevant or not, we need a data source from which information will be retrieved and tested using test cases. The table shown below will provide with necessary data sets:
| group_id        | item                                       | amount | score | identity score | risk_strategy_type       | payment_is_test | product_type       |
|-----------------|--------------------------------------------|--------|-------|----------------|--------------------------|-----------------|--------------------|
| 6ef811855e53    | photo_camera                               | 1300   | 410   | 85             | high                     | False           | electronics        |
| 5a9d3f7292a7    | bathtub                                    | 800    | 390   | 105            | high                     | False           | installments       |
| 4bc72a689e31    | ear_plugs                                  | 1500   | 1760  | 90             | high                     | False           | mixed              |
| 7cf9462101b2    | book                                       | 2000   | 2100  | 78             | high                     | True            | electronics        |
| 2de8a473a0d6    | refrigerator                               | 950    | 1300  | 55             | middle                   | False           | kitchen utensils   |
| 3af927e4b569    | vacuum cleaner                             | 1700   | 1520  | 82             | high                     | False           | home appliances    |

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
  if group_id == "6ef811855e53"
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
- EXP_D (not often bought, unpopular)

    ```python
  if (group_id == "4bc72a689e31" or group_id == "7cf9462101b2")
    and (score <= 200 and amount > 1200)
    and item in ("battery", "book", "lamp", "ear_plugs", "tea_pot"):
    
      then 
        decision = "reject" and stop_factors = 'EXP_D';
    ```
<br>

**Exceptions**

```python
if risk_strategy_type in ("light", "middle") or payment_is_test == True or product_type = "installments":
```
  

  
