Complete the tasks below:

- You should create a database named `salon`
```sql
CREATE DATABASE salon;
```
- You should connect to your database, then create tables named `customers`, `appointments`, and `services`. Hint: `\c salon`
- Each table should have a primary key column that automatically increments
- Each primary key column should follow the naming convention, `table_name_id`. For example, the `customers` table should have a `customer_id` key. Note that there’s no s at the end of customer
```sql
CREATE TABLE customers(customer_id SERIAL PRIMARY KEY);
CREATE TABLE appointments(appointment_id SERIAL PRIMARY KEY);
CREATE TABLE services(service_id SERIAL PRIMARY KEY);
```

Now, let's check the list of relations
```txt
                         List of relations
 Schema |              Name               |   Type   |    Owner     
--------+---------------------------------+----------+--------------
 public | appointments                    | table    | freecodecamp
 public | appointments_appointment_id_seq | sequence | freecodecamp
 public | customers                       | table    | freecodecamp
 public | customers_customer_id_seq       | sequence | freecodecamp
 public | services                        | table    | freecodecamp
 public | services_service_id_seq         | sequence | freecodecamp
(6 rows)
```
- Your `appointments` table should have a `customer_id` foreign key that references the `customer_id` column from the customers table
```sql
ALTER TABLE appointments ADD COLUMN customer_id INT REFERENCES customers(customer_id);
```
- Your `appointments` table should have a `service_id` foreign key that references the `service_id` column from the services table
```sql
ALTER TABLE appointments ADD COLUMN service_id INT REFERENCES services(service_id);
```
- Your `customers` table should have `phone` that is a `VARCHAR` and must be unique
```sql
ALTER TABLE customers ADD COLUMN phone VARCHAR(20) UNIQUE;
```
- Your `customers` and `services` tables should have a `name` column
```sql
ALTER TABLE customers ADD COLUMN name VARCHAR(20) NOT NULL;
ALTER TABLE services ADD COLUMN name VARCHAR(20) NOT NULL;
```
- Your `appointments` table should have a `time` column that is a `VARCHAR`
```sql
ALTER TABLE appointments ADD COLUMN time VARCHAR(10);
```
- You should have at least three rows in your `services` table for the different services you offer, one with a `service_id` of 1
```
INSERT INTO services(name) VALUES('nails'),('hair'),('waxing');
```
- Your script file should have executable permissions
- You should create a script file named `salon.sh` in the project folder. Hint: `touch salon.sh`
- Your script file should have a “shebang” that uses bash when the file is executed (use `#! /bin/bash`).
- You should not use the `clear` command in your script
- You should display a numbered list of the services you offer before the first prompt for input, each with the format `#) <service>`. For example, `1) cut`, where 1 is the `service_id`
```sh
#! /bin/bash

PSQL="psql -X --username=freecodecamp --dbname=salon --tuples-only -c"

echo -e "\n~~~~~ MY SALON ~~~~~\n"

echo -e "Welcome to My Salon, how can I help you?\n"

MAIN_MENU(){
  if [[ $1 ]]
  then
    echo -e "\n$1"
  fi

  AVAILABLE_SERVICES=$($PSQL  "SELECT service_id, name FROM services ORDER BY service_id")
  
  if [[ -z $AVAILABLE_SERVICES ]]
  then
    echo "Sorry, we don't have any service available"
  else
    echo "$AVAILABLE_SERVICES" | while read SERVICE_ID BAR NAME
    do
      echo "$SERVICE_ID) $NAME"
    done
  fi
}

MAIN_MENU
```
Output:
```txt
~~~~~ MY SALON ~~~~~

Welcome to My Salon, how can I help you?

1) nails
2) hair
3) waxing
```
- If you pick a service that doesn't exist, you should be shown the same list of services again
- Your script should prompt users to enter a service_id, phone number, a name if they aren’t already a customer, and a time. You should use read to read these inputs into variables named `SERVICE_ID_SELECTED`, `CUSTOMER_PHONE`, `CUSTOMER_NAME`, and `SERVICE_TIME`
- If a phone number entered doesn’t exist, you should get the `customers` name and enter it, and the phone number, into the `customers` table
- You can create a row in the appointments table by running your script and entering `1`, `555-555-5555`, `Fabio`, `10:30` at each request for input if that phone number isn’t in the `customers` table. The row should have the `customer_id` for that customer, and the `service_id` for the service entered
- You can create another row in the `appointments` table by running your script and entering `2`, `555-555-5555`, `11am` at each request for input if that phone number is already in the `customers` table. The row should have the `customer_id` for that `customer`, and the `service_id` for the service entered

Here is my full source code:
```sh
#! /bin/bash

PSQL="psql -X --username=freecodecamp --dbname=salon --tuples-only -c"

echo -e "\n~~~~~ MY SALON ~~~~~\n"

echo -e "Welcome to My Salon, how can I help you?\n"

MAIN_MENU(){
  if [[ $1 ]]
  then
  echo -e "\n$1"
  fi

  AVALIABLE_SERVICES=$($PSQL  "SELECT service_id, name FROM services ORDER BY  service_id")
 
  if [[ -z $AVALIABLE_SERVICES ]]
  then
  echo "Sorry, we dont have any service available right now"
  else
  echo "$AVALIABLE_SERVICES" | while read SERVICE_ID BAR NAME
    do
      echo "$SERVICE_ID) $NAME"
    done

    read SERVICE_ID_SELECTED
    if [[ ! $SERVICE_ID_SELECTED =~ ^[0-9]+$ ]]
    then
     MAIN_MENU "That is not a number"
     else
      SERV_AVAIL=$($PSQL "SELECT service_id FROM services WHERE service_id = $SERVICE_ID_SELECTED ")
      NAME_SERV=$($PSQL "SELECT name FROM services WHERE service_id = $SERVICE_ID_SELECTED ")
      if [[ -z $SERV_AVAIL ]]
        then
         MAIN_MENU "I could not find that service. What would you like today?"
         else
         echo -e "\nWhat's your phone number?"
         read CUSTOMER_PHONE
         CUSTOMER_NAME=$($PSQL "SELECT name FROM customers WHERE phone = '$CUSTOMER_PHONE'")
         if [[ -z $CUSTOMER_NAME ]]
         then
            echo -e "\nWhat's your name?"
          read CUSTOMER_NAME
          INSERT_CUSTOMER_RESULT=$($PSQL "INSERT INTO customers(name, phone) VALUES('$CUSTOMER_NAME', '$CUSTOMER_PHONE')")
          
         fi
        echo -e "\nWhat time would you like your $NAME_SERV, $CUSTOMER_NAME?"
        read SERVICE_TIME
        CUSTOMER_ID=$($PSQL "SELECT customer_id FROM customers WHERE phone='$CUSTOMER_PHONE'")
        if [[ $SERVICE_TIME ]]
        then
          INSERT_SERV_RESULT=$($PSQL "INSERT INTO appointments(customer_id, service_id, time) VALUES($CUSTOMER_ID, $SERVICE_ID_SELECTED,'$SERVICE_TIME')")
          if [[ $INSERT_SERV_RESULT ]]
          then
            echo -e "\nI have put you down for a $NAME_SERV at $SERVICE_TIME, $(echo $CUSTOMER_NAME | sed -r 's/^ *| *$//g')."
          fi
        fi
      fi
    fi
  fi
}

MAIN_MENU
```
