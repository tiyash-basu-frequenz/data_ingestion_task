## Frequenz Data-Engineering coding task
This is the take-home coding task for you as our potential data engineer. We
have outlined the requirements for the task in the following section. We expect
you to use spark to implement the solution.

In your solution, we will look for correctness, quality, and scalability. Plus
points will be awarded for any optimization technique that you apply.

Write your code with the awareness that other people may have to maintain or
modify it in future: try to make their job easier.

Feel free to ask us whatever questions you like, just as if you were a colleague
working remotely.

## Task

### Data sources
* [renewable_power_plants_DE.csv](https://data.open-power-system-data.org/renewable_power_plants/2020-08-25/renewable_power_plants_DE.csv)
is a table of renewable power plants in Germany.
    * You have to download this.
    * You can find the description of the table's fields [in this webpage](https://data.open-power-system-data.org/renewable_power_plants/2020-08-25),
    under the headings `Field documentation`, `renewable power plants DE.csv`.
* Time-series data of household power usage will arrive as a csv file once every
5 minutes in a directory.
    * In reality this directory would probably exist in a distributed file
    system, but for this task you can mock this as a local directory on your
    computer.
    * We have provided 3 such data files with mocked up data in the
    `household_data_mock` directory.
    * The data in these files are supposed to be sampled at their corresponding
    timestamps.
    * For a given household, between two consecutive data points, you can assume
    that the power consumption remains constant.
    * Field description:

    | Field Name          | Type           | Description                                                                                   |
    |---------------------|----------------|-----------------------------------------------------------------------------------------------|
    | `ts`                | Timestamp      | The timestamp of this data sample.                                                            |
    | `household_id`      | Integer        | The household ID of the household.                                                            |
    | `postcode`          | Integer        | The postcode of the household, and the postcode of the power plants which supply power to it. |
    | `power_usage_in_kw` | Floating-Point | The power drawn by this household in kilowatts at the given timestamp.                        |

### Assumptions
* There is only one household per postcode.
* Households can be powered by both renewable and non-renewable power plants,
but only from the ones in their postcodes.
* Households' power needs are first supplied by their local renewable power
plants. If the power usage of a household exceeds the capacity of its local
renewable power plant, then the additional power is supplied by its local
non-renewable power plant.
* All power plants are able to output their full power capacity at all times.

### Objectives
You would need to process the two data sources and you can store the results in
any way you wish. However, the resulting data should be able to handle the
following user queries:

a. Total energy consumption (in kWh) for given households over a given period of
time, e.g., for household_id `1` and `3` from `2020-01-01-12:01:00` to
`2020-01-01-12:04:00`. You can assume that the power consumption of a household
remains fixed until its next data point.

E.g., for the following records:
```
ts,household_id,postcode,power_usage_in_kw
2020-01-01 12:00:00,1,4389,3.684
2020-01-01 12:01:00,1,4389,3.681
2020-01-01 12:02:00,1,4389,3.224
2020-01-01 12:03:00,1,4389,3.664
2020-01-01 12:04:00,1,4389,3.375
```
the power usage of household 1 is assumed to be `3.684 kW` at all times between
`2020-01-01 12:00:00` and `2020-01-01 12:01:00`, and the total energy
consumption of the household from `2020-01-01 12:00:00` to `2020-01-01 12:04:00`
was `(3.684 + 3.681 + 3.224 + 3.664) kW * (1 / 60) h = 0.23755 kWh`.

b. Did a household, given a household_id, receive 100% renewable energy at a
given time or not, e.g., for household_id `1` at `2020-01-01 12:03:10`.
Again, you can assume that the power consumption of a household remains fixed
until it's next data point.

E.g., for the following records:
```
ts,household_id,postcode,power_usage_in_kw
2020-01-01 12:03:00,2,15337,3.069
2020-01-01 12:04:00,2,15337,2.683
2020-01-01 12:05:00,2,15337,2.964
```
since the total power output of the renewable power plants in postcode `15337`
is about `3 kW`, at `2020-01-01 12:03:10`, the household was not fully powered
by its local renewable power plants, but, at `2020-01-01 12:04:10` it was.

You are free to choose how the queries written and received. You should,
however, provide examples of how a user can query these values. SQL queries
that can be run with spark-sql are acceptable.
