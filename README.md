#### Project
# Hair Salon Business App
#### Team Projects Overview
- This is a team project. Through GitHub Classroom you should have created a shared GitHub repository with your teammates. As long as you upload your finished code to that team repository, each team member will get credit. You will also need to fill out a peer review survey on Learning Suite to recieve credit (if you're in a team by yourself, you don't need to).
- I do not provide automated tests for projects. You will need to determine yourself whether the code meets the requirements provided in the rubric. It is important for you to be able to determine whether a program you write meets requirements (in the real world there won't be pre-written tests to tell you if you did your job right).
#### Overview
- For this project, assume a local hair salon, Incredible Cuts, asked your team to create an app prototype for them. Up until now, they've just been recording their business data manually. They provided you a sample of that data in the `starting_hair_salon_data.xlsx` file. They want you to create a tool that lets their customers check in, as well as give managers access to important data.
- [Click here](https://www.youtube.com/watch?v=amOjyOzAcgI) for a video of me running through the logic of the program.
- This project is split up into 4 parts in 4 different files:
    - **Part 1: Create a database and classes with peewee**
        - Use the `p03_1_db_classes.py` file
    - **Part 2: Clean and import data from Excel into your database**
        - Use the `p03_2_data_import.py` file
    - **Part 3: Write the customer check-in process**
        - Use the `p03_3_hair_salon.py` file
    - **Part 4: Write some manager tools**
        - Use the `p03_4_manager_tools.py` file
- Important: Because you'll have multiple .py files open, make sure you save all your open files before running any single file.
    - You can go to File > Save All in VS Code to do this.
    - Or, the shortcut on Mac is `cmd + opt + s`
    - Or, the shortcut on Windows is `ctrl + k` then release the keys, then press `s`
- Part 1 should be done before the others, and it will be easier to do part 3 & 4 if part 2 is done first, but you could work on parts 2, 3, and 4 simultaneously if you want to split work up among group members.

## Libraries Required:
- If you are using a virtual environment, you can use `pip install -r requirements.txt` to install all necessary external libraries.
- Part 1:
    - `peewee`
- Part 2:
    - `pandas`
    - `sqlite3`
    - import function you make in part 1
- Part 3:
    - `random`
    - `datetime`
        - For `datetime`, I recommend importing it like this: `from datetime import datetime`
    - import classes you make in part 1
- Part 4:
    - import classes you make in part 1


## Part 1: Create a database and classes with peewee
- Write all your code for part 1 in the `p03_1_db_classes.py` file
- Using `peewee`, create an SQLite database called `hair_salon.db`.
- Your database should contain 3 tables: `Customer`, `Stylist`, and `Appointment`.
    - This means in python you need to create 3 classes with those same names that inherit from the `peewee` class `Model`.
- Eventually, in Part 2 of the project, you will fill the database with data from the provided Excel file. Part 2 is easier to do if you keep the column names you write now in part 1 consistent with the Excel file column names. Suggested column names are provided below, but if you want to use different column names, feel free.
- `Customer` Class fields:
    - `c_id`: AutoField, should also be a primary key
    - `c_first_name`: CharField
    - `c_last_name`: CharField
    - `c_phone_number`: CharField or IntegerField
    - `c_gender`: CharField
- `Stylist` Class fields:
    - `s_id`: AutoField, should also be a primary key
    - `s_first_name`: CharField
    - `s_last_name`: CharField
    - `s_hire_date`: DateField
    - `s_gender`: CharField
- `Appointment` Class fields:
    - `a_id`: AutoField, should also be a primary key
    - `a_haircut_type`: CharField
    - `a_date_time`: DateTimeField
    - `a_payment`: FloatField
    - `a_satisfied`: BooleanField
    - `c_id`: ForeignKeyField, needs to reference `Customer`, call the backreference 'appointments'. set on_delete to be 'CASCADE'
    - `s_id`: ForeignKeyField, needs to reference `Stylist`, call the backreference 'appointments'. set on_delete to be 'CASCADE'
- Include code to start from scratch (for convenience)
    - I recommend pasting the code below at the bottom of your file. It shouldn't be part of any of your classes (meaning the def should be outdented all the way to the left). It will delete any tables you've made and recreate them.
    - This way, if you ever need to make a change to your database (like if you misspelled a column name or something) you can just rerun this file and it will start the database from scratch.
    - the `if __name__` portion of the code is telling python to only run the `start_from_scratch()` function if you are starting python from this file specifically. Feel free to look it up how `__name__` works if you're curious.
```
# This assumes you called your peewee sqlite database connection variable "db"
def start_from_scratch(db_to_restart = db):
    """
    Drops specified tables ('appointment', 'stylist', 'customer') in the SQLite database if they exist, then recreates them.
    """
    db_to_restart.connect() # connect to the SQLite database global variable
    tables_to_drop = ['appointment', 'stylist', 'customer']
    with db_to_restart.atomic():
        for table in tables_to_drop:
            db.execute_sql(f"DROP TABLE IF EXISTS {table};")
    db_to_restart.create_tables([Customer, Stylist, Appointment]) # creates the tables for the classes written if they don't exist.
    db_to_restart.close()
    print("Recreated database structure from scratch")

if __name__ == '__main__':
    start_from_scratch(db)
```
## Part 2: Clean and import data from Excel into your database
- Write all your code for part 2 in the `p03_2_data_import.py` module. The point of this module is to:
    - Import the starting data to python using pandas
    - Clean up the data (make sure it is formatted how we want it)
    - Separate the data out so we can store it in tables
    - Export the data into your SQLite database.
- All of the stages above are very important, but we haven't practiced most of them. So, I will give you the code one piece at a time. But take a minute with each piece of code and try to understand what the code is doing. As you gain more experience, a big part of programming is figuring out how to structure data in a way that makes it easier to work with. This part of the project will give you a small taste of what that process might look like.

#### 1. Import the packages/functions you need:
```
# 1. import everything you need
import pandas as pd
import sqlite3
from p03_1_db_classes import start_from_scratch
```

#### 2. Import from Excel to Pandas
```
# 2. Import the starting data as a dataframe
combined_df = pd.read_excel(r"starting_hair_salon_data.xlsx")
print("STARTING DATA:\n", combined_df.head(), "\n")
```

#### 3. Clean the data
```
# 3. Clean the data: Get rid of the dashes in phone numbers. This makes it easier
#    to look up phone numbers in part 3 of the project.
combined_df['c_phone_number'] = combined_df['c_phone_number'].str.replace('-', '')
```

#### 4. Normalize the data
In database terminology, "normalizing" means separating out data into separate tables (and sometimes separate columns). Here, we need to have separate Customer, Stylist, and Appointment data, so we should first create 3 different dataframes.
```
# 4. Normalize the data: Separate out the customer and stylist data to their own dataframes.
# 4.1 Customer and stylist data is duplicated multiple times in the original Excel file. Having one row per customer and stylist is more efficient.
#     To do this, we are just grabbing the columns that relate to either customers or stylists, then dropping the duplicates.
#     Resetting the index is optional, but makes it so it starts counting from 0 again. It just makes it look cleaner.
c_df = combined_df[['c_first_name', 'c_last_name', 'c_phone_number', 'c_gender']].drop_duplicates().reset_index(drop=True)
s_df = combined_df[['s_first_name', 's_last_name', 's_gender', 's_hire_date']].drop_duplicates().reset_index(drop=True)

# 4.2 To make a new primary key column, we can just use the index that pandas already creates. We just need to store it in a new column
#     that matches the column name we put in the Customer and Stylist classes in part 1.
c_df['c_id'] = c_df.index
s_df['s_id'] = s_df.index
```

#### 5. Add foreign keys to Appointment
Now, instead of of storing customer or stylist data in the same table as appointments, we will instead store a "foreign key" or a reference to one of the primary keys in the customer or stylist table.
- The reason to do this is that instead of repeating customer/stylist data over an over again, we just store a reference to the table holding the data where it is only stored once.
- For example if "Bob" gets a haircut 50 times at our salon, without foreign keys, we would need to store his name, phone number, etc in 50 separate rows. Now, we can store Bob once in the Customer table, and just reference his primary key in the Appointment table as a foreign key as many times as we want. Much more efficient, and less prone to data errors in the long run!
- You'll learn more about this if you take a database class.
```
# 5. Add foreign keys to the appointments table
# 5.1 Each appointment has 1 customer. Rather than repeatedly storing the customer data in the appointments
#     table, we will store a foreign key (the c_id column) that points to the customer data in the Customer table.
#     We can use .merge() to combine the original dataframe with the new customers dataframe (that now has c_id)  
a_df = combined_df.merge(c_df, on=['c_first_name', 'c_last_name', 'c_phone_number', 'c_gender'])

# 5.2 Do the same thing, but with stylist data, to add the s_id to appointments data.
a_df = a_df.merge(s_df, on=['s_first_name', 's_last_name', 's_gender', 's_hire_date'])

# 5.3 Now get rid of c_ and s_ columns except for the s_id and c_id foreign keys
a_df.drop(['c_first_name', 'c_last_name', 'c_phone_number', 'c_gender', 's_first_name', 's_last_name', 's_hire_date', 's_gender'], axis=1, inplace=True)

# 5.4 Create a primary key column for appointments
a_df["a_id"] = a_df.index
```

#### 6. Display the results
This is optional, but I recommend doing this to see if you actually did everything correctly
```
# 6. Display the results (optional, but good to check if you did it right)
print("CUSTOMER DATA:\n", c_df.head(), "\n")
print("STYLIST DATA:\n", s_df.head(), "\n")
print("APPOINTMENT DATA:\n", a_df.head(), "\n")
```
It should look like this (depending on how zoomed in you are in VS Code it might cut some parts off though):
```
CUSTOMER DATA:
   c_first_name c_last_name c_phone_number c_gender  c_id
0      Kliment      Genney     1234567890        M     0
1       Elmira      Oldred     1287067297        F     1
2      Desmund      Wallis     7039155340        M     2
3     Courtney  Missington     7166163251        F     3
4         Ferd  Theunissen     1061671013        M     4 

STYLIST DATA:
   s_first_name s_last_name s_gender s_hire_date  s_id
0       Cammie     Ashbolt        F  2020-04-27     0
1     Eldredge    O'Connor        M  2022-10-15     1
2      Miguela       Pizey        F  2022-03-16     2
3      Isadora        Puig        F  2022-11-24     3
4        Gregg     Sprowle        M  2023-06-09     4 

APPOINTMENT DATA:
   a_haircut_type         a_date_time  a_payment  a_satisfied  c_id  s_id  a_id
0       Undercut 2024-11-21 19:10:00         22         True     0     0     0
1       Undercut 2024-11-21 01:27:00         22         True     1     1     1
2         Mullet 2024-11-15 18:19:00         18         True     2     0     2
3           Fade 2024-11-13 00:05:00         20         True     3     2     3
4       Buzz Cut 2024-11-08 02:21:00         15        False     4     3     4 
```
#### 7. Recreate the database from scratch
Call the function you created in part 1.
```
# 7. Recreate the database from scratch
start_from_scratch()
```

#### 8. Connect to the database
```
# 8. Make a connection to the SQLite database using python's built in
# SQLite library. The built in library works with pandas better than peewee does.
db = sqlite3.connect('hair_salon.db')
```

#### 9. Export data from Pandas to SQLite database
Use pandas' nifty .to_sql() function to export the data to your database using the connection you made previously
```
# 9. Send the data from the pandas dataframes to your SQLite database
#    Use lowercase for the table names, that is how SQLite table names are stored
#    We use the 'append' option, because if we used 'replace' it would overwrite
#    the settings we made in our peewee classes.
c_df.to_sql('customer', db, if_exists='append', index=False)
s_df.to_sql('stylist', db, if_exists='append', index=False)
a_df.to_sql('appointment', db, if_exists='append', index=False)
print("Exported cleaned and separated data to SQLite database")
```

At the end of all this, you should have a Customer table with 368 rows, a Stylist table with 10 rows, and an Appointment table with 1000 rows.


## Part 3: Check In Process
- The point of this module is to simulate the check in process for a customer walking in to the hair salon. Customers will enter in their phone number to check in. If the phone number is already in the database, they are recognized as a returning customer and can choose to use their previously stored data. Otherwise, they are a new customer and more info will be gathered about them.
- You can watch [this video](https://www.youtube.com/watch?v=amOjyOzAcgI) going over this part of the project if you haven't already, or you can also check out the `check_in_process.pdf` file in this repository for an overview of the logical flow of this module if you're a fan of process models.
- Write all your code for part 3 in the `p03_3_check_in.py` module.
- This module should import the `Customer`, `Stylist`, `Appointment` classes you made in your `p3_db_classes.py` module:
    - `from p03_solution_1_db_classes import Customer, Stylist, Appointment`

#### 1. Ask user to enter in a 10-digit phone number:
- First ask the user to enter a phone number:
    - `Hello! Welcome to Incredible Cuts. Please enter your phone number to check in: `
- > Note: To make your code easier to test, the dataset includes a customer with 3 appointments that has the phone number `1234567890` Just slide your finger across the number pad!
- You should check whether the input is a valid 10 digit phone number, and it should be able to handle any spaces <code>&nbsp;</code> or any dashes `-` inside the input. 
- To simplify the project, you can assume that no user will ever enter any other invalid characters (like `asdf 123-456-asdf-7890!!`), though feel free to add logic to handle those cases too if you want extra optional practice.
- For example, these 3 examples should all be considered valid:
    - `1234567890`
    - `123 456 7890`
    - `123-456-7890`
- > **Hint**: Here are 3 potential ways you might check the input:
    > -  Use `.replace()` multiple times to get rid of spaces and dashes.
    > - Try to convert each character of the input to an `int()` (using try/except). Only keep the characters that are capable of becoming ints.
    > - Check whether `.isdecimal()` returns `True` for each character of the input, and only keep those characters.
- Any input that doesn't include 10 digits should be rejected, and you should ask the user to reinput the number.
    - For example, if the user enters something like this:
        - `123`
    - It should print out:
        `That wasn't a 10 digit phone number. Please try again.`
    - Any then ask again until a valid 10 digit number is provided.

#### 2. If phone number is in the database, get that customer as an object
- If the phone number corresponds to an existing customer in the database, retrieve that customer as a `Customer` object. Otherwise, you'll create a new customer object (described in section `3` below)
- Using the customer object you got, call the method `returning_customer_message()`, which is method you need to write in the `Customer` class in `p03_1_db_classes.py`
    - `returning_customer_message()` should print out:
        - `Welcome! You've had <number of appointments> appointments with us!`
        - Remember, if you made your `Appointment` class with a foreign key backreference called `appointments`, then that means in your `Customer` class, you can do `self.appointments` to get access to all of the appointments for that `Customer` object. You can do any peewee stuff with that, for example, `self.appointments.order_by(Appointment.a_date_time.desc())` would work, etc.
    - If they've had no appointments, then the function should just end there and return `None`.
    - But, it they've had at least 1 appointment, get the data from their most recent appointment and print this out:
        - `At your last haircut on <appointment date time> you got a <haircut type> haircut with <stylist first name> as your stylist.`
        - Your method should return an appointment object of their most recent appointment.
- If `returning_customer_message()` returned an `Appointment` object, then ask:
    - `Do you want to continue with these same options? Enter 'Y' if so, otherwise enter 'N': `
    - If they enter `Y`, then use the same `haircut_type` and `Stylist` when you create the `Appointment` in step 5.
- If either:
    - `returning_customer_message()` does not return an `Appointment` object, or
    - The user enters `N` (or anything other than `Y`) when asked about continuing with the previous options
- Then just proceed to choosing a hairstyle and stylist in step 4.

#### 3. If phone number isn't in the database, create a new customer
- If the phone number entered at the start is valid, but is not in the database, then do the following:
    - print `Thanks for joining us! Enter the following information:`
    - Get a first name: `Enter your first name: `
    - Get a last name: `Enter your last name: `
    - Get their gender: `Enter your gender (M or F): `
- Then using that information and the phone number originally entered, create a new customer in the database, and store the customer object in a variable.

#### 4. Choose a hairstyle and stylist
- In the following 3 situations, a Customer will need to choose a hairstyle and stylist:
    - 1. New customers
    - 2. Returning customers that do not have any appointment data (this is rare, but could happen)
    - 3. Returning customers that did not respond `Y` to using the options from their last haircut
- *First, choose a hairstyle*
    - You can use the included dictionary's keys as the choices for hairstyle
        - ```
            hairstyles_dict = {
            "Undercut": 22.0,
            "Mullet": 18.0,
            "Fade":	20.0,
            "Buzz Cut":	15.0,
            "Pixie Cut": 30.0,
            "Bob": 28.0,
            "Pompadour": 40.0,
            "Layered Cut": 25.0,
            "Crew Cut": 16.0,
            "Shag": 35.0
            }
            ```
    - Ask the user to enter the number of hairstyle they want:
        - ```
            These are the available hairstyles: 
            1: Undercut
            2: Mullet
            3: Fade
            4: Buzz Cut
            5: Pixie Cut
            6: Bob
            7: Pompadour
            8: Layered Cut
            9: Crew Cut
            10: Shag
            Enter the number of the hairstyle you want:
            ```
        - To simplify the project, you can assume that the user will always enter a valid number of hairstyle, meaning you don't need to handle errors from improper inputs here to get full credit.
- *2nd, choose an option for selecting a stylist:*
    - `To get the first available stylist regardless of gender, enter 1. If you want your stylist to be the same gender as you, enter 2: `
    - To simplify the project, assume the user will always enter either 1 or 2.
    - Remember that you can get all Stylists by using `Stylist.select()`. You might find that easier to work with if you convert that to a list by using `list(Stylist.select())`
    - If they enter `1`, just grab a random `Stylist`.
    - If they enter `2`, grab a random `Stylist`, but make sure that the gender of the stylist matches the gender of the `Customer`.
    - Either way, you should have a `Stylist` object for the appointment

#### 5. Create an `Appointment`
- At this point, you should have a `Customer` object, as well a `Stylist` object, and a string of the hairstyle they chose (whether by manually choosing or by using the same options as their last haircut)
- Print a message stating that the haircut was performed:
    - `<stylist first name> <stylist last name> gave you a <hairstyle> haircut today!`
- Ask whether they were satisfied. If they enter `Y`, store it as a `True` boolean value, otherwise `False`
    - `Are you satisfied? Enter Y or N: `
- Create an `Appointment` entry in the database using the `Appointment` class you already made in your `p03_1_db_classes.py` file.
    - for `a_date_time`, set it equal to `datetime.today()`
    - `a_haircut_type` should be a string of the hairstyle they chose.
    - `a_payment` should be how much the haircut cost. See the included dictionary for what the price should be for each hairstyle.
    - `a_satisfied` should be a `True` or `False` boolean.
    - `c_id` should be a customer object
    - `s_id` should be a stylist object.
- It should then print out:
    - `Thank you for choosing Incredible Cuts! We hope to see you again soon.`
    - The program can then end.

## Part 4: Manager Tools
- The point of this module is to simulate a few of the capabilities and reports that a manager of the hair salon might want access to.
- Write all your code for part 4 in the `p03_4_manager_tools.py` file.
#### Make a menu:
- Print out the following:
    - ```
      Welcome to Manager Tools. Choose an option below: 
      1. See the 5 most recent appointments that weren't satisfied
      2. Delete a stylist from the database
      3. Exit
      Enter an option: 
      ```
    - Repeatedly print out this menu after an option is chosen, until the user enters `3` to exit.

#### Option 1:
- If the user enters `1`, then get the 5 most recent appointments where the `a_satisfied` is equal to `False`.
- Run a method called `get_appointment_info()` on each `Appointment` object that you get and print the result.
    - Create `get_appointment_info()` in the `Appointment` class as a method in `p03_1_db_classes.py`.
    - If the appointment's `a_satisfied` is `True` then return:
        - `Appointment <appointment id> on <date time>: <customer first name> <customer last name> got a <haircut type> from <stylist first name> <stylist last name>. <customer first name> was satisified.`.
    - If the appointemnt's `a_satisfied` is `False` then return (identical except for the very end):
        - `Appointment <appointment id> on <date time>: <customer first name> <customer last name> got a <haircut type> from <stylist first name> <stylist last name>. <customer first name> was not satisified.`

#### Option 2:
- If the user enters `2`, then it should display the id, first name, and last name of all the Stylists and ask for an ID to enter. If no other Stylists have been deleted (or added), it will probably look like this:
    - ```
        0: Cammie Ashbolt
        1: Eldredge O'Connor
        2: Miguela Pizey
        3: Isadora Puig
        4: Gregg Sprowle
        5: Addison Hawler
        6: Paxon Katzmann
        7: Zorana Slym
        8: Karna Mac
        9: Allix Kivelle
        Enter the ID of a stylist to delete: 
        ```
    - To simplify the project, you can assume that a proper ID will always be provided.
- After the user enters an ID, it should delete that Stylist from the datababse and then print a message:
    - `<stylist first name> <stylist last name> was deleted from the database.`
- If you are running into errors when doing this, make sure your `Appointment` class `s_id` variable has `on_delete='CASCADE'` listed in the parentheses of the `ForeignKeyField()`

#### Option 3:
- If the user enters `3`, print out:
    - `Thank you for using the program.`
- Then the program should end.

#### Any other input:
- If they enter any invalid choice, just print out `Invalid selection. Try Again.`

#### Extra challenges:
- If you want some extra challenges for no extra points, you could try doing more reports for managers, such as:
    - See which stylists have brought in the most money
    - See which stylists have the highest satisfaction
    - See which hair cut styles have brought in the most money.
- To do any of the above, you probably need to be familiar with using group by, which you can do in peewee, pandas, or raw SQL.

## Grading Rubric
Remember there are no automated tests for projects. `See RUBRIC.md`. Remember to right click and select "Open Preview" to view the file in a nice format. The TAs will update this file with your grade and any comments they have when they are done grading your submission.

## Example Output

### Part 1: p03_1_db_classes.py
Nothing necessary prints out here, but the file should create your database and all your models that you'll use in the 3 other files. If you run it with the `start_from_scratch()` function, it will print:
```
Recreated database structure from scratch
```

### Part 2: p03_2_data_import.py
Nothing necessary prints out here, but this file should fill your database with data in all 3 tables. If you print out some of the data like the code I provided shows, it would look like this:
```
STARTING DATA:
   c_first_name c_last_name c_phone_number c_gender s_first_name s_last_name s_gender s_hire_date a_haircut_type         a_date_time  a_payment  a_satisfied
0      Kliment      Genney   123-456-7890        M       Cammie     Ashbolt        F  2020-04-27       Undercut 2024-11-21 19:10:00         22         True
1       Elmira      Oldred   128-706-7297        F     Eldredge    O'Connor        M  2022-10-15       Undercut 2024-11-21 01:27:00         22         True
2      Desmund      Wallis   703-915-5340        M       Cammie     Ashbolt        F  2020-04-27         Mullet 2024-11-15 18:19:00         18         True
3     Courtney  Missington   716-616-3251        F      Miguela       Pizey        F  2022-03-16           Fade 2024-11-13 00:05:00         20         True
4         Ferd  Theunissen   106-167-1013        M      Isadora        Puig        F  2022-11-24       Buzz Cut 2024-11-08 02:21:00         15        False 

CUSTOMER DATA:
   c_first_name c_last_name c_phone_number c_gender  c_id
0      Kliment      Genney     1234567890        M     0
1       Elmira      Oldred     1287067297        F     1
2      Desmund      Wallis     7039155340        M     2
3     Courtney  Missington     7166163251        F     3
4         Ferd  Theunissen     1061671013        M     4 

STYLIST DATA:
   s_first_name s_last_name s_gender s_hire_date  s_id
0       Cammie     Ashbolt        F  2020-04-27     0
1     Eldredge    O'Connor        M  2022-10-15     1
2      Miguela       Pizey        F  2022-03-16     2
3      Isadora        Puig        F  2022-11-24     3
4        Gregg     Sprowle        M  2023-06-09     4 

APPOINTMENT DATA:
   a_haircut_type         a_date_time  a_payment  a_satisfied  c_id  s_id  a_id
0       Undercut 2024-11-21 19:10:00         22         True     0     0     0
1       Undercut 2024-11-21 01:27:00         22         True     1     1     1
2         Mullet 2024-11-15 18:19:00         18         True     2     0     2
3           Fade 2024-11-13 00:05:00         20         True     3     2     3
4       Buzz Cut 2024-11-08 02:21:00         15        False     4     3     4 

Recreated database structure from scratch
Exported cleaned and separated data to SQLite database
```

### Part 3: p03_3_check_in.py

Here is a returning customer choosing to use the same hair cut and stylist as before:

```
Hello! Welcome to Incredible Cuts. Please enter your phone number to check in: 1234567890
Welcome! You've had 3 appointments with us!
At your last haircut on 2024-11-21 19:10:00 you got a Undercut haircut with Cammie as your stylist.
Do you want to continue with these same options? Enter 'Y' if so, otherwise enter 'N': Y
Cammie Ashbolt gave you a Undercut haircut today!
Are you satisfied? Enter Y or N: Y

Thank you for choosing Incredible Cuts! We hope to see you again soon.
```

Here is a returning customer choosing to enter in a new hair cut and stylist. Notice that Using dashes with the phone number works just fine:
```
Hello! Welcome to Incredible Cuts. Please enter your phone number to check in: 123-456-7890
Welcome! You've had 4 appointments with us!
At your last haircut on 2024-12-07 01:58:09.617900 you got a Undercut haircut with Cammie as your stylist.
Do you want to continue with these same options? Enter 'Y' if so, otherwise enter 'N': N
These are the available hairstyles: 
1: Undercut
2: Mullet
3: Fade
4: Buzz Cut
5: Pixie Cut
6: Bob
7: Pompadour
8: Layered Cut
9: Crew Cut
10: Shag
Enter the number of the hairstyle you want: 4
To get the first available stylist regardless of gender, enter 1. If you want your stylist to be the same gender as you, enter 2: 2
Eldredge O'Connor gave you a Buzz Cut haircut today!
Are you satisfied? Enter Y or N: N

Thank you for choosing Incredible Cuts! We hope to see you again soon.
```

Here is a new customer with a phone number not recognized in the database:

```
Hello! Welcome to Incredible Cuts. Please enter your phone number to check in: 0987654321

Thanks for joining us! Enter the following information:
Enter your first name: Jimmy
Enter your last name: John
Enter your gender (M or F): M
These are the available hairstyles: 
1: Undercut
2: Mullet
3: Fade
4: Buzz Cut
5: Pixie Cut
6: Bob
7: Pompadour
8: Layered Cut
9: Crew Cut
10: Shag
Enter the number of the hairstyle you want: 9
To get the first available stylist regardless of gender, enter 1. If you want your stylist to be the same gender as you, enter 2: 1

Allix Kivelle gave you a Crew Cut haircut today!
Are you satisfied? Enter Y or N: Y

Thank you for choosing Incredible Cuts! We hope to see you again soon.
```

### Part 4: p03_4_manager_tools.py

Here's an example of choosing all 3 options:

```
Welcome to Manager Tools. Choose an option below: 
1. See the 5 most recent appointments that weren't satisfied
2. Delete a stylist from the database
3. Exit
Enter an option: 1

Appointment 1001 on 2024-12-07 01:59:05.786888: Kliment Genney got a Buzz Cut from Eldredge O'Connor. Kliment was not satisified.
Appointment 4 on 2024-11-08 02:21:00: Ferd Theunissen got a Buzz Cut from Isadora Puig. Ferd was not satisified.
Appointment 9 on 2024-10-31 19:26:00: Horacio Morpeth got a Fade from Zorana Slym. Horacio was not satisified.
Appointment 12 on 2024-10-27 19:30:00: Etheline Latimer got a Pixie Cut from Paxon Katzmann. Etheline was not satisified.
Appointment 13 on 2024-10-27 01:32:00: Beatriz Timmis got a Pompadour from Cammie Ashbolt. Beatriz was not satisified.

Welcome to Manager Tools. Choose an option below: 
1. See the 5 most recent appointments that weren't satisfied
2. Delete a stylist from the database
3. Exit
Enter an option: 2
0: Cammie Ashbolt
1: Eldredge O'Connor
2: Miguela Pizey
3: Isadora Puig
4: Gregg Sprowle
5: Addison Hawler
6: Paxon Katzmann
7: Zorana Slym
8: Karna Mac
9: Allix Kivelle

Enter the ID of a stylist to delete: 9
Allix Kivelle was deleted from the database.

Welcome to Manager Tools. Choose an option below: 
1. See the 5 most recent appointments that weren't satisfied
2. Delete a stylist from the database
3. Exit
Enter an option: 3
Thank you for using the program.
```
