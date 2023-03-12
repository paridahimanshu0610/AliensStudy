### Creating tables for Aliens Study

#### 1. Creating `aliens` table:
```
DROP TABLE IF EXISTS aliens;

CREATE TABLE aliens (
  id INT NOT NULL,
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  email VARCHAR(250),
  gender VARCHAR(50),
  type VARCHAR(50),
  birth_year INT,
  PRIMARY KEY (id)
);
```
#### 2. Creating `details` table:

```
DROP TABLE IF EXISTS details;
CREATE TABLE details (
  detail_id INT NOT NULL,
  favorite_food VARCHAR(250),
  feeding_frequency VARCHAR(50),
  aggressive BOOLEAN,
  PRIMARY KEY (detail_id)
);
```

#### 3. Creating `location` table:
```
DROP TABLE IF EXISTS location;
CREATE TABLE location (
  loc_id INT NOT NULL,
  current_location VARCHAR(100),
  state VARCHAR(50),
  country VARCHAR(150),
  occupation VARCHAR(250),
  PRIMARY KEY (loc_id)
);

```

### Inserting Data into those tables

```
LOAD DATA INFILE 'C:\\ProgramData\\MySQL\\MySQL Server 8.0\\Uploads\\Projects\\AliensStudy\\aliens.csv'
INTO TABLE aliens 
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 ROWS;

LOAD DATA INFILE 'C:\\ProgramData\\MySQL\\MySQL Server 8.0\\Uploads\\Projects\\AliensStudy\\details.csv'
INTO TABLE details 
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 ROWS;

LOAD DATA INFILE 'C:\\ProgramData\\MySQL\\MySQL Server 8.0\\Uploads\\Projects\\AliensStudy\\location.csv'
INTO TABLE location 
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 ROWS;
```

