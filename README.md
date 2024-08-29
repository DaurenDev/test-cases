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
In order to determine whether an item is relevant or not, we need a data source from which information will be retrieved and tested using test cases. The table shown below will provide with necessary data set:
| group_id        | item                                       | score | identity score | risk_strategy_type       | payment_is_test | product_type       |
|-----------------|--------------------------------------------|-------|----------------|--------------------------|-----------------|--------------------|
| 6ef811855e53    | camera                                     | 1450  | 105             | ["middle", "high"]       | false           | installments       |
| 5a9d3f7292a7    | blender                                    | 980   | 62             | ["light"]                | true            | kitchen utensils   |
| 4bc72a689e31    | air conditioner                            | 1760  | 90             | ["middle"]               | false           | home appliances    |
| 7cf9462101b2    | headphones                                 | 2100  | 78             | ["high"]                 | true            | electronics        |
| 2de8a473a0d6    | refrigerator                               | 1300  | 100             | ["middle", "light"]      | false           | installments       |
| 3af927e4b569    | vacuum cleaner                             | 1520  | 82             | ["high"]                 | false           | home appliances    |
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
    and item in ("video", "camera", "kitchen_dining", "bath", "baby_product")
    and (
      (score <= 350) or
      (score <= 400 and amount > 600) or
      (score <= 420 and amount > 1200) or
      (identity_score > 100 and amount > 750 and score <= 420)
      ):
  
    then 
      decision = "reject" and stop_factors = 'SC_N;'
  ```
- EXP_D

  
