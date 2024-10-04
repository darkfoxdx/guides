```
DELIMITER $$

CREATE PROCEDURE `COUNT_ALL_RECORDS_BY_TABLE`()
BEGIN
  DECLARE done INT DEFAULT 0;
  DECLARE TNAME CHAR(255);

  -- Declare a cursor for selecting table names from the current database
  DECLARE table_names CURSOR FOR 
    SELECT table_name 
    FROM INFORMATION_SCHEMA.TABLES 
    WHERE TABLE_SCHEMA = DATABASE() AND TABLE_TYPE = 'BASE TABLE';

  -- Handle when the cursor reaches the end of the table list
  DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

  -- Open the cursor
  OPEN table_names;

  -- Drop the table if it exists and create a temporary table to store counts
  DROP TEMPORARY TABLE IF EXISTS TCOUNTS;
  CREATE TEMPORARY TABLE TCOUNTS (
    TABLE_NAME CHAR(255),
    RECORD_COUNT INT
  );

  -- Fetch each table name and count records
  read_loop: LOOP
    FETCH table_names INTO TNAME;

    IF done THEN
      LEAVE read_loop;
    END IF;

    -- Construct dynamic SQL to count records for each table
    SET @SQL_TXT = CONCAT('INSERT INTO TCOUNTS(TABLE_NAME, RECORD_COUNT) ',
                          'SELECT ''', TNAME, ''' AS TABLE_NAME, COUNT(*) AS RECORD_COUNT FROM `', TNAME, '`');

    -- Debugging step (optional): Print the SQL being executed
    -- SELECT @SQL_TXT;

    -- Prepare, execute and deallocate the statement
    PREPARE stmt_name FROM @SQL_TXT;
    EXECUTE stmt_name;
    DEALLOCATE PREPARE stmt_name;
  END LOOP;

  -- Close the cursor
  CLOSE table_names;

  -- Select all counts and the total record count
  SELECT * FROM TCOUNTS;
  SELECT SUM(RECORD_COUNT) AS TOTAL_DATABASE_RECORD_CT FROM TCOUNTS;

END$$

DELIMITER ;
```
