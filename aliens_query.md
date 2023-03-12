# MySQL Data Analysis for Aliens Study

### Checking table contents

#### 1. Checking the `aliens` table contents:

```
SELECT * FROM aliens LIMIT 5;
```


| id | first_name | last_name  | email                   | gender      | type      | birth_year |
|----|------------|------------|-------------------------|-------------|-----------|------------|
|  1 | Tyrus      | Wrey       | twrey0@sakura.ne.jp     | Agender     | Reptile   |       1717 |
|  2 | Ealasaid   | St Louis   | estlouis1@amazon.co.uk  | Female      | Flatwoods |       1673 |
|  3 | Violette   | Sawood     | vsawood2@yolasite.com   | Female      | Nordic    |       1675 |
|  4 | Rowan      | Saintsbury | rsaintsbury3@rediff.com | Male        | Green     |       1731 |
|  5 | Free       | Ingolotti  | fingolotti4@bbb.org     | Genderfluid | Flatwoods |       1763 |

#### 2. Checking the `details` table contents:

```
SELECT * FROM details LIMIT 5;
```

| detail_id | favorite_food             | feeding_frequency | aggressive |
|-----------|---------------------------|-------------------|------------|
|         1 | White-faced tree rat      | Weekly            |          1 |
|         2 | Lizard, goanna            | Seldom            |          0 |
|         3 | Indian red admiral        | Weekly            |          1 |
|         4 | Bandicoot, southern brown | Often             |          0 |
|         5 | Kangaroo, red             | Once              |          0 |

#### 3. Checking the `location` table contents:

```
SELECT * FROM location LIMIT 5;
```

| loc_id | current_location | state      | country       | occupation             |
|--------|------------------|------------|---------------|------------------------|
|      1 | Cincinnati       | Ohio       | United States | Senior Cost Accountant |
|      2 | Bethesda         | Maryland   | United States | Senior Sales Associate |
|      3 | Oakland          | California | United States | Registered Nurse       |
|      4 | Richmond         | Virginia   | United States | Director of Sales      |
|      5 | Atlanta          | Georgia    | United States | Administrative Officer |

### Creating a temporary custom table:

```
DROP TABLE IF EXISTS alien_data;

CREATE TEMPORARY TABLE alien_data AS (
  SELECT
    a.id,
    LOWER(a.first_name) AS first_name,
    LOWER(a.last_name) AS last_name,
    a.email,
    CASE
      WHEN LOWER(a.gender) <> 'female' AND LOWER(a.gender) <> 'male' THEN 'non-binary'
      ELSE LOWER(a.gender)
    END AS gender,
    LOWER(a.type) AS type,
    a.birth_year,
    (YEAR(NOW()) - a.birth_year) AS age,
    LOWER(d.favorite_food) AS favorite_food,
    LOWER(d.feeding_frequency) AS feeding_frequency,
    d.aggressive,
    LOWER(l.occupation) AS occupation,
    LOWER(l.current_location) AS current_location,
    LOWER(l.state) AS state,
    CASE
      WHEN LOWER(l.state) IN ('maine', 'new hampshire', 'massachusetts', 'connecticut', 'vermont', 'rhode island') THEN 'new england'
      WHEN LOWER(l.state) IN ('alabama', 'arkansas', 'florida', 'georgia', 'kentucky', 'louisiana', 'mississippi', 'north carolina', 'south carolina', 'tennessee', 'virginia', 'west virginia') THEN 'southeast'
      WHEN LOWER(l.state) IN ('wisconsin', 'ohio', 'indiana', 'illinois', 'michigan') THEN 'great lakes'
      WHEN LOWER(l.state) IN ('new mexico', 'arizona', 'texas', 'oklahoma') THEN 'southwest'
      WHEN LOWER(l.state) IN ('north dakota', 'south dakota', 'kansas', 'iowa', 'nebraska', 'missouri', 'minnesota') THEN 'plains'
      WHEN LOWER(l.state) IN ('colorado', 'utah', 'idaho', 'montana', 'wyoming') THEN 'rocky mountain'
      WHEN LOWER(l.state) IN ('new york', 'new jersey', 'pennsylvania', 'delaware', 'maryland', 'district of columbia') THEN 'mideast'
      WHEN LOWER(l.state) IN ('california', 'alaska', 'nevada', 'oregon', 'washington', 'hawaii') THEN 'far west'
    END AS us_region,
    LOWER(l.country) AS country
  FROM aliens AS a
  JOIN details AS d ON a.id = d.detail_id
  JOIN location AS l ON a.id = l.loc_id
);
```

Here, we combine data from all 3 table with some additional changes:

1. For uniformity, we convert the text data type columns to lowercase. 

2. Assign a region to each state. 

3. Calculate the age column based on the birth_year column.

### Data Analysis

#### 1. Checking the number of records in alien_data

```
SELECT count(*) AS n_records FROM alien_data;
```

**Result:**

| n_records |
|-----------|
|     50000 |

#### 2. Checking duplicate email addresses in our data

```
SELECT email,
    count(*) as email_count
FROM alien_data
GROUP BY email
HAVING email_count > 1;
```

**Result:**

|email|email_count|
|------|-----|

An empty table is returned. This implies all email ids in our data are unique.

#### 3. Unique countries in our data

```
SELECT country AS countries
FROM alien_data
GROUP BY country;
```

**Result:**


| countries     |
|---------------|
| united states |

#### 4. Number of unique states inhabitated by aliens

```
SELECT count(DISTINCT state) AS number_of_states
FROM alien_data;
```

**Result:**

| number_of_states |
|------------------|
|               51 |

Aliens are all across the 50 states and the District of Columbia 

#### 5. For each state, list the aliens count, average age, % hostile aliens, % friendly aliens

```
SELECT state,
  COUNT() AS state_count,
  ROUND(AVG(age)) AS avg_age,
  ROUND(
    (
      (
        SUM(
          CASE
            WHEN aggressive = 0 THEN 1
            ELSE 0
          END
        ) / COUNT() * 100
      )
    ),
    2
  ) AS friendly_alien_percentage,
  ROUND(
    (
      (
        SUM(
          CASE
            WHEN aggressive = 1 THEN 1
            ELSE 0
          END
        ) / COUNT(*) * 100
      )
    ),
    2
  ) AS hostile_alien_percentage
FROM alien_data
GROUP BY state
ORDER BY state_count DESC
LIMIT 5;
```

**Result:**
| state      | state_count | avg_age | friendly_alien_percentage | hostile_alien_percentage |
|------------|-------------|---------|---------------------------|--------------------------|
| texas      |        5413 |     201 |                     49.53 |                    50.47 |
| california |        5410 |     203 |                     50.15 |                    49.85 |
| florida    |        4176 |     200 |                     50.36 |                    49.64 |
| new york   |        2690 |     203 |                     50.56 |                    49.44 |
| ohio       |        1851 |     200 |                     49.43 |                    50.57 |


#### 6. Checking the minimum, maximum and average age (in years)

```
SELECT MAX(age) AS maximum_age,
  MIN(age) AS minimum_age,
  ROUND(AVG(age)) AS average_age
FROM alien_data;
```

**Result:**

| maximum_age | minimum_age | average_age |
|------------|--------------|-------------|
|        351 |           51 |         201 |


#### 7. Getting the regionwise alien count and percentage

```
SELECT COUNT(*) INTO @total_count FROM alien_data;

SELECT us_region, COUNT(*) AS regional_count, ROUND((COUNT(*)/@total_count)*100,2) AS alien_percentage
FROM alien_data
GROUP BY us_region
ORDER BY alien_percentage DESC;
```

**Result:**
| us_region      | regional_count | alien_percentage |
|----------------|----------------|------------------|
| southeast      |          13856 |            27.71 |
| far west       |           7885 |            15.77 |
| southwest      |           7600 |            15.20 |
| mideast        |           7205 |            14.41 |
| great lakes    |           5725 |            11.45 |
| plains         |           4052 |             8.10 |
| rocky mountain |           2006 |             4.01 |
| new england    |           1671 |             3.34 |


#### 8. Getting the gender distribution in each region

```
SELECT t.us_region,
    t.gender,
    t.gender_count,
    ROUND(
        (
            t.gender_count / SUM(t.gender_count) OVER (PARTITION BY t.us_region)
        ) * 100,
        2
    ) AS gender_percentage,
    RANK() OVER (
        PARTITION BY t.us_region
        ORDER BY t.gender_count DESC
    ) AS ranking
FROM (
        SELECT us_region,
            gender,
            COUNT(gender) AS gender_count
        FROM alien_data
        GROUP BY us_region,
            gender
    ) AS t;
```

#### Shorter code:

```
SELECT us_region,
    gender,
    COUNT(gender) AS gender_count,
    ROUND(
        (
            COUNT(gender) / SUM(COUNT(gender)) OVER (PARTITION BY us_region)
        ) * 100,
        2
    ) AS gender_percentage,
    RANK() OVER (
        PARTITION BY us_region
        ORDER BY COUNT(gender) DESC
    ) AS ranking
FROM alien_data
GROUP BY us_region,
    gender
ORDER BY us_region,
    gender;
```

**Result:**

| us_region      | gender     | gender_count | gender_percentage | ranking |
|----------------|------------|--------------|-------------------|---------|
| far west       | female     |         3540 |             44.90 |       1 |
| far west       | male       |         3526 |             44.72 |       2 |
| far west       | non-binary |          819 |             10.39 |       3 |
| great lakes    | female     |         2615 |             45.68 |       1 |
| great lakes    | male       |         2531 |             44.21 |       2 |
| great lakes    | non-binary |          579 |             10.11 |       3 |
| mideast        | female     |         3251 |             45.12 |       1 |
| mideast        | male       |         3229 |             44.82 |       2 |
| mideast        | non-binary |          725 |             10.06 |       3 |
| new england    | female     |          791 |             47.34 |       1 |
| new england    | male       |          716 |             42.85 |       2 |
| new england    | non-binary |          164 |              9.81 |       3 |
| plains         | female     |         1818 |             44.87 |       2 |
| plains         | male       |         1849 |             45.63 |       1 |
| plains         | non-binary |          385 |              9.50 |       3 |
| rocky mountain | female     |          935 |             46.61 |       1 |
| rocky mountain | male       |          872 |             43.47 |       2 |
| rocky mountain | non-binary |          199 |              9.92 |       3 |
| southeast      | female     |         6332 |             45.70 |       1 |
| southeast      | male       |         6175 |             44.57 |       2 |
| southeast      | non-binary |         1349 |              9.74 |       3 |
| southwest      | female     |         3448 |             45.37 |       1 |
| southwest      | male       |         3425 |             45.07 |       2 |
| southwest      | non-binary |          727 |              9.57 |       3 |

#### 9. Identifying the distinct species 

```
SELECT DISTINCT type FROM alien_data;
```

**Result:**

| type      |
|-----------|
| reptile   |
| flatwoods |
| nordic    |
| green     |
| grey      |


#### 10. For each species, identifying the top 3 regions with respect to their population

```
WITH temp AS (
    SELECT type AS species,
        us_region,
        COUNT(us_region) AS species_count,
        RANK() OVER (
            PARTITION BY type
            ORDER BY COUNT(us_region) DESC
        ) AS ranking
    FROM alien_data
    GROUP BY type,
        us_region
)
SELECT species,
    us_region,
    species_count
FROM temp
WHERE ranking <= 3
ORDER BY species,
    species_count DESC;
```

**Result:**

| species   | us_region | species_count |
|-----------|-----------|---------------|
| flatwoods | southeast |          2848 |
| flatwoods | far west  |          1620 |
| flatwoods | southwest |          1530 |
| green     | southeast |          2752 |
| green     | far west  |          1608 |
| green     | southwest |          1461 |
| grey      | southeast |          2799 |
| grey      | southwest |          1532 |
| grey      | far west  |          1501 |
| nordic    | southeast |          2768 |
| nordic    | far west  |          1548 |
| nordic    | southwest |          1506 |
| reptile   | southeast |          2689 |
| reptile   | far west  |          1608 |
| reptile   | southwest |          1571 |

#### 11. Finding out the favourite food for each alien species

```
WITH temp AS (
    SELECT type AS species,
        favorite_food,
        COUNT(favorite_food) AS fav_count,
        RANK() OVER (
            PARTITION BY type
            ORDER BY COUNT(favorite_food) DESC
        ) AS ranking
    FROM alien_data
    GROUP BY type,
        favorite_food
)
SELECT species,
    favorite_food
FROM temp
WHERE ranking = 1
ORDER BY species;
```

**Result:**

| species   | favorite_food             |
|-----------|---------------------------|
| flatwoods | eagle, bateleur           |
| green     | gray duiker               |
| grey      | openbill stork            |
| nordic    | pine snake (unidentified) |
| nordic    | scaly-breasted lorikeet   |
| nordic    | two-toed tree sloth       |
| reptile   | gonolek, burchell's       |
