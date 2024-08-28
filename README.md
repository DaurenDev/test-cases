# Content

1. [Usage](#usage)
2. [Data](#datatable)
3. [Rules and exceptions to apply](#rules)
4. [Test cases](#conclusion)

## Usage
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
| 6ef811855e53    | ["smartphone", "laptop"]                   | 1450  | 85             | ["middle", "high"]        | false           | electronics        |
| 5a9d3f7292a7    | ["blender", "microwave"]                   | 980   | 62             | ["light"]                 | true            | kitchen utensils   |
| 4bc72a689e31    | ["air conditioner"]                        | 1760  | 90             | ["middle"]                | false           | home appliances    |
| 7cf9462101b2    | ["headphones", "gaming console"]           | 2100  | 78             | ["high"]                  | true            | electronics        |
| 2de8a473a0d6    | ["refrigerator", "coffee maker"]           | 1300  | 55             | ["middle", "light"]       | false           | kitchen utensils   |
| 3af927e4b569    | ["dishwasher", "vacuum cleaner"]           | 1520  | 82             | ["high"]                  | false           | home appliances    |

