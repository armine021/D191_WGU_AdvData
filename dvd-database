-- programming environment: pgAdmin procedural programming language PL/pgSQL
-- enhanced PostGRE SQL to create server objects that stores multiple statements

-- create procedure to generate detail and summary reports on customers
CREATE OR REPLACE PROCEDURE total_rentals_reports()
language PLpgSQL
AS $BODY$
begin

-- erase the table if it already exists (ensures data is current to the date query is performed)
DROP TABLE IF EXISTS customer_details;

-- create the empty details table
CREATE TABLE IF NOT EXISTS customer_details (
	customer_id INT NOT NULL,
	email VARCHAR,
	postal_code VARCHAR,
	national_area VARCHAR,
	customer_total_rentals INT,
	customer_sent_discount BOOLEAN DEFAULT false
);

-- add values to the table from existing ones in the database
-- join using the customer ID to unite information on a customer, their rentals, and their address
INSERT INTO customer_details
SELECT
	customer.customer_id,
	customer.email,
	address.postal_code,
	substr(address.postal_code, 1, 1),
	count(rental.customer_id) as customer_total_rentals
FROM
	rental
-- inner join using the customer ID across 3 tables (customer, rental, and address)
INNER JOIN customer
	ON customer.customer_id = rental.customer_id
INNER JOIN address
	ON customer.address_id = address.address_id
GROUP BY customer.customer_id, address.postal_code
;

-- troubleshoot issues with the tables - will return 5 random records from the specified table
-- select * from customer_details LIMIT 5;

-- erase the table if it already exists (ensures data is current to the date query is performed)
DROP TABLE IF EXISTS customer_summary;

-- create the empty summary table
CREATE TABLE IF NOT EXISTS customer_summary (
	customer_id INT NOT NULL,
	customer_total_rentals INT,
	PRIMARY KEY (customer_id),
	FOREIGN KEY (customer_id)
		REFERENCES customer (customer_id)
); 

-- add values to the table from existing ones in the database
-- limit the values added to this table as customers that have not yet received a discount and ones only from a specified national area
INSERT INTO customer_summary
SELECT
	customer_id,
	customer_total_rentals
FROM
	customer_details
WHERE 
	customer_details.customer_sent_discount = false AND
	-- change this national area value to target other regions
	customer_details.national_area = '1'
ORDER BY customer_total_rentals ASC;
END;
$BODY$;	

-- troubleshoot issues with the tables - will return 5 random records from the specified table
-- select * from customer_summary LIMIT 5;

-- create the reports
call total_rentals_reports();

-- troubleshoot issues with the tables - will return 5 random records from the specified table
-- select * from customer_details LIMIT 5;
-- select * from customer_summary LIMIT 5;

-- trigger function to update both the detail and summary report
CREATE OR REPLACE FUNCTION refresh_reports()
	RETURNS TRIGGER
	LANGUAGE PLpgSQL
AS $BODY$
BEGIN
	call total_rentals_reports();
RETURN NULL; 
END;
$BODY$;

-- create the trigger itself, which happens any time a record is updates in the details table
-- if trigger already exists, use this to remove the old trigger:
-- DROP TRIGGER refresh_reports_trigger;
CREATE TRIGGER refresh_reports_trigger
AFTER UPDATE
on customer_details
FOR EACH ROW
EXECUTE PROCEDURE refresh_reports();
