# freeCodeCamp: Build a Periodic Table Database

## Instructions
You are started with a `periodic_table` database that has information about some chemical elements. You can connect to it by entering `psql --username=freecodecamp --dbname=periodic_table` in the terminal. 

You may want to get a little familiar with the existing tables, columns, and rows. Read the instructions below and complete user stories to finish the project. Certain tests may not pass until other user stories are complete. Good luck!

## Part 1: Fix the database
There are some mistakes in the database that need to be fixed or changed. See the user stories below for what to change.

## Part 2: Create your git repository
You need to make a small bash program. The code needs to be version controlled with git, so you will need to turn the suggested folder into a git repository.

## Part 3: Create the script
Lastly, you need to make a script that accepts an argument in the form of an `atomic number`, `symbol`, or `name` of an element and outputs some information about the given element. 

In your script, you can create a PSQL variable for querying the database like this: `PSQL="psql --username=freecodecamp --dbname=<database_name> -t --no-align -c"`, add more flags if you need to.

## Requirements & My Solutions

```
# Connect to SQL Database
psql --username=freecodecamp --dbname=periodic_table
```

```
# Check Database
# Check Database
\d

             List of relations
 Schema |    Name    | Type  |    Owner     
--------+------------+-------+--------------
 public | elements   | table | freecodecamp
 public | properties | table | freecodecamp
(2 rows)

# Check Table 'Elements'
\d elements

Table "public.elements"
    Column     |         Type          | Collation | Nullable | Default 
---------------+-----------------------+-----------+----------+---------
 atomic_number | integer               |           | not null | 
 symbol        | character varying(2)  |           |          | 
 name          | character varying(40) |           |          | 
Indexes:
    "elements_pkey" PRIMARY KEY, btree (atomic_number)
    "elements_atomic_number_key" UNIQUE CONSTRAINT, btree (atomic_number)
    
    
# Check Table 'Properties'
\d properties

Table "public.properties"
    Column     |         Type          | Collation | Nullable | Default 
---------------+-----------------------+-----------+----------+---------
 atomic_number | integer               |           | not null | 
 type          | character varying(30) |           |          | 
 weight        | numeric(9,6)          |           | not null | 
 melting_point | numeric               |           |          | 
 boiling_point | numeric               |           |          | 
Indexes:
    "properties_pkey" PRIMARY KEY, btree (atomic_number)
    "properties_atomic_number_key" UNIQUE CONSTRAINT, btree (atomic_number)
```

#### You should rename the `weight` column to `atomic_mass`
```
# Alter Table 'Properties' - Rename Column 'Weight'
ALTER TABLE properties RENAME COLUMN weight to atomic_mass; 
```

#### You should rename the `melting_point` column to `melting_point_celsius` and the `boiling_point` column to `boiling_point_celsius`
```
# Alter Table 'properties' - Rename Column 'melting_point'
ALTER TABLE properties RENAME COLUMN melting_point to melting_point_celsius;

# Alter Table 'properties' - Rename Column 'boiling_point'
ALTER TABLE properties RENAME COLUMN boiling_point to boiling_point_celsius;
```

#### Your `melting_point_celsius` and `boiling_point_celsius` columns should not accept null values
```
# Alter Table 'properties' - Alter column 'melting_point_celsius' to NOT NULL
ALTER TABLE properties ALTER COLUMN melting_point_celsius SET NOT NULL;

# Alter Table 'properties' - Alter column 'boiling_point_celsius' to NOT NULL
ALTER TABLE properties ALTER COLUMN boiling_point_celsius SET NOT NULL;
```

#### You should add the `UNIQUE` constraint to the `symbol` and `name` columns from the `elements` table
```
# Alter table 'elements' - Alter column 'symbol' as UNIQUE
ALTER TABLE elements ADD CONSTRAINT unique_symbol UNIQUE (symbol);

# Alter table 'elements' - Alter column 'name' as UNIQUE
ALTER TABLE elements ADD CONSTRAINT unique_name UNIQUE (name);
```

#### Your `symbol` and `name` columns should have the `NOT NULL` constraint
```
# Alter table 'elements' - Alter column 'symbol' as NOT NULL
ALTER TABLE elements ALTER COLUMN symbol SET NOT NULL;

# Alter table 'elements' - Alter column 'name' as NOT NULL
ALTER TABLE elements ALTER COLUMN name SET NOT NULL;
```

#### You should set the `atomic_number` column from the `properties` table as a foreign key that references the column of the same name in the `elements` table
```
# Alter table 'properties' - Alter column 'atomic_number' (set foreign key)
ALTER TABLE properties ADD FOREIGN KEY(atomic_number) REFERENCES elements(atomic_number);
```

### **You should create a `types` table that will store the three types of elements**
```
# Create Table 'Types'
CREATE TABLE types();
```

#### Your `types` table should have a `type_id` column that is an integer and the primary key
#### Your `types` table should have a `type` column that's a `VARCHAR` and cannot be `null`. It will store the different types from the `type` column in the `properties` table
```
# Alter Table 'types' - Add Column 'type_id'
ALTER TABLE types ADD COLUMN type_id SERIAL PRIMARY KEY;

# Alter Table 'types' - Add Column 'type'
ALTER TABLE types ADD COLUMN type VARCHAR(255) NOT NULL; 
```

#### You should add three rows to your `types` table whose values are the three different types from the `properties` table
```
# Check the three different types in the table 'properties'
SELECT * FROM properties;

periodic_table=> SELECT * FROM properties;
 atomic_number |   type    | atomic_mass | melting_point_celsius | boiling_point_celsius 
---------------+-----------+-------------+-----------------------+-----------------------
             1 | nonmetal  |    1.008000 |                -259.1 |                -252.9
             2 | nonmetal  |    4.002600 |                -272.2 |                  -269
             3 | metal     |    6.940000 |                180.54 |                  1342
             4 | metal     |    9.012200 |                  1287 |                  2470
             5 | metalloid |   10.810000 |                  2075 |                  4000
             6 | nonmetal  |   12.011000 |                  3550 |                  4027
             7 | nonmetal  |   14.007000 |                -210.1 |                -195.8
             8 | nonmetal  |   15.999000 |                  -218 |                  -183
          1000 | metalloid |    1.000000 |                    10 |                   100
(9 rows)

# Insert into table 'types' column 'type' - Add values 'nonmetal', 'metal', 'metalloid'
INSERT INTO types (type) VALUES ('nonmetal');
INSERT INTO types (type) VALUES ('metal');
INSERT INTO types (type) VALUES ('metalloid');
```

#### Your `properties` table should have a `type_id` foreign key column that references the `type_id` column from the `types` table. It should be an `INT` with the `NOT NULL` constraint
#### Each row in your `properties` table should have a `type_id` value that links to the correct type from the `types` table
```
# This is a workaround solution
# Since we cannot add the 'type_id' column (idk why), we do this:
ALTER TABLE properties RENAME COLUMN type TO type_id;
ALTER TABLE properties ADD COLUMN temp_type INT NULL;
UPDATE properties SET temp_type = 1 WHERE type_id = 'metal';
UPDATE properties SET temp_type = 2 WHERE type_id = 'nonmetal';
UPDATE properties SET temp_type = 3 WHERE type_id = 'metalloid';
ALTER TABLE properties DROP COLUMN type_id;
ALTER TABLE properties RENAME COLUMN temp_type TO type_id;

# Alter table 'properties' - Add foreign key 'type_id'
ALTER TABLE properties ALTER COLUMN type_id SET NOT NULL;
ALTER TABLE properties ADD FOREIGN KEY(type_id) REFERENCES types(type_id);
```

#### You should capitalize the first letter of all the `symbol` values in the `elements` table. Be careful to only capitalize the letter and not change any others
```
# Display all the values in the table 'elements' and column 'symbol'
\d elements;

                        Table "public.elements"
    Column     |         Type          | Collation | Nullable | Default 
---------------+-----------------------+-----------+----------+---------
 atomic_number | integer               |           | not null | 
 symbol        | character varying(2)  |           | not null | 
 name          | character varying(40) |           | not null | 
Indexes:
    "elements_pkey" PRIMARY KEY, btree (atomic_number)
    "elements_atomic_number_key" UNIQUE CONSTRAINT, btree (atomic_number)
    "unique_name" UNIQUE CONSTRAINT, btree (name)
    "unique_symbol" UNIQUE CONSTRAINT, btree (symbol)
Referenced by:
    TABLE "properties" CONSTRAINT "properties_atomic_number_fkey" FOREIGN KEY (atomic_number) R
EFERENCES elements(atomic_number)

SELECT * FROM elements
 atomic_number | symbol |   name    
---------------+--------+-----------
             1 | H      | Hydrogen
             2 | he     | Helium
             3 | li     | Lithium
             4 | Be     | Beryllium
             5 | B      | Boron
             6 | C      | Carbon
             7 | N      | Nitrogen
             8 | O      | Oxygen
          1000 | mT     | moTanium
(9 rows)

# Capitalise them one by one (where necessary)
UPDATE elements SET symbol = 'He' WHERE symbol ='he';
UPDATE elements SET symbol = 'Li' WHERE symbol ='li';
UPDATE elements SET symbol = 'Mt' WHERE symbol ='mT';
```

#### You should remove all the trailing zeros after the decimals from each row of the `atomic_mass` column. You may need to adjust a data type to `DECIMAL` for this. The final values they should be are in the `atomic_mass.txt` file
```
# Display all values from table 'properties' column 'atomic_mass'
SELECT * FROM properties

 atomic_number | atomic_mass | melting_point_celsius | boiling_point_celsius | type_id 
---------------+-------------+-----------------------+-----------------------+---------
             3 |    6.940000 |                180.54 |                  1342 |       1
             4 |    9.012200 |                  1287 |                  2470 |       1
             1 |    1.008000 |                -259.1 |                -252.9 |       2
             2 |    4.002600 |                -272.2 |                  -269 |       2
             6 |   12.011000 |                  3550 |                  4027 |       2
             7 |   14.007000 |                -210.1 |                -195.8 |       2
             8 |   15.999000 |                  -218 |                  -183 |       2
             5 |   10.810000 |                  2075 |                  4000 |       3
          1000 |    1.000000 |                    10 |                   100 |       3
(9 rows)

# Change the constraint for the column
ALTER TABLE properties ALTER COLUMN atomic_mass SET DATA TYPE DECIMAL;

# Update each row under the 'atomic_mass' column
UPDATE properties SET atomic_mass = 6.94 WHERE atomic_number = 3;
UPDATE properties SET atomic_mass = 9.0122 WHERE atomic_number = 4;
UPDATE properties SET atomic_mass = 1.008 WHERE atomic_number = 1;
UPDATE properties SET atomic_mass = 4.0026 WHERE atomic_number = 2;
UPDATE properties SET atomic_mass = 12.011 WHERE atomic_number = 6;
UPDATE properties SET atomic_mass = 14.007 WHERE atomic_number = 7;
UPDATE properties SET atomic_mass = 15.999 WHERE atomic_number = 8;
UPDATE properties SET atomic_mass = 10.81 WHERE atomic_number = 5;
UPDATE properties SET atomic_mass = 1 WHERE atomic_number = 1000;
```

#### You should add the element with atomic number `9` to your database. Its name is `Fluorine`, symbol is `F`, mass is `18.998`, melting point is `-220`, boiling point is `-188.1`, and it's a `nonmetal`
```
# Add a new row to the table 'elements'
INSERT INTO elements(symbol, name, atomic_number) VALUES ('F', 'Fluorine', 9);
# Add a new row to the table 'properties'
INSERT INTO properties(atomic_number, atomic_mass, melting_point_celsius, boiling_point_celsius, type_id) VALUES (9, 18.998, -220, -188.1, 2); 
```

#### You should add the element with atomic number `10` to your database. Its name is `Neon`, symbol is `Ne`, mass is `20.18`, melting point is `-248.6`, boiling point is `-246.1`, and it's a `nonmetal`
```
# Add a new row to the table 'elements'
INSERT INTO elements(symbol, name, atomic_number) VALUES ('Ne', 'Neon', 10);
# Add a new row to the table 'properties'
INSERT INTO properties(atomic_number, atomic_mass, melting_point_celsius, boiling_point_celsius, type_id) VALUES (10, 20.18, -248.6, -246.1, 2); 
```

#### You should create a `periodic_table` folder in the `project` folder and turn it into a git repository with `git init`
```
## Split terminal

# Create a folder
mkdir periodic_table

# Turn it into a git repo
cd periodic_table
git init
```

#### Your repository should have a `main` branch with all your commits
```
git checkout -b main
```

#### You should create an `element.sh` file in your repo folder for the program I want you to make
- The message for the first commit in your repo should be `Initial commit`
```
# Create a file
touch element.sh

# Make git repo comit (1st) - to main branch
git add .
git commit -m "Initial commit"
```

- Your script (.sh) file should have executable permissions
```
chmod +x element.sh
```

- If you run `./element.sh`, it should output only `Please provide an element as an argument.` and finish running.
```
# Open the element.sh file and paste the following line
echo "Please provide an element as an argument."
```

- The rest of the commit messages should start with `fix:`, `feat:`, `refactor:`, `chore:`, or `test:`
```
# Make git repo comit (2nd)
git add .
git commit -m "feat: Add prompt"
```

- You should delete the non existent element, whose atomic_number is 1000, from the two tables
```
DELETE FROM properties WHERE atomic_number = 1000;
DELETE FROM elements WHERE atomic_number = 1000;
```

- If the argument input to your `element.sh` script doesn't exist as an `atomic_number`, `symbol`, or `name` in the database, the only output should be `I could not find that element in the database.`
- If you run `./element.sh 1`, `./element.sh H`, or `./element.sh Hydrogen`, it should output only `The element with atomic number 1 is Hydrogen (H). It's a nonmetal, with a mass of 1.008 amu. Hydrogen has a melting point of -259.1 celsius and a boiling point of -252.9 celsius.`
- If you run `./element.sh` script with another element as input, you should get the same output but with information associated with the given element.

```
# Run these command lines on terminal
./element.sh 1

The element with atomic number 1 is Hydrogen (H). It's a metal, with a mass of 1.008 amu. Hydrogen has a melting point of -259.1 celsius and a boiling point of -252.9 celsius.

./element.sh H
The element with atomic number 1 is Hydrogen (H). It's a metal, with a mass of 1.008 amu. Hydrogen has a melting point of -259.1 celsius and a boiling point of -252.9 celsius.

./element.sh Hydrogen
The element with atomic number 1 is Hydrogen (H). It's a metal, with a mass of 1.008 amu. Hydrogen has a melting point of -259.1 celsius and a boiling point of -252.9 celsius.
```

- Your `periodic_table` repo should have at least five commits
- The rest of the commit messages should start with `fix:`, `feat:`, `refactor:`, `chore:`, or `test:`
- You should finish your project while on the `main` branch. Your working tree should be clean and you should not have any uncommitted changes

```
# Make git repo comit (3rd)
git add .
git commit -m "fix: something"

# Make git repo comit (4th)
git add .
git commit -m "refactor: something"

# Make git repo comit (5th)
git add .
git commit -m "chore: something"

# Make git repo comit (6th)
git add .
git commit -m "test: something"
```
