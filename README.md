# SnowflakeUserKeyValidation
Snowflake User Key Validation with Apache NiFi



### Steps

Call Snowflake with this SQL to get back a list of users
  SHOW USERS;


Call Snowflake with this SQL to get back the RSA public key
  DESC USER ${username};

Put the public key in a file

Run Shell Script against that file
  openssl rsa -pubin -in /Users/tspann/Downloads/code/nifi-2.4.0/sf.pub -text -noout
