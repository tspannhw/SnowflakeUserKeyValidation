### SnowflakeUserKeyValidation

Snowflake User Key Validation with Apache NiFi


![image](https://github.com/user-attachments/assets/46219139-50f2-4423-90ee-9565dc8fd384)


![image](https://github.com/user-attachments/assets/aa9dc5ac-f1ea-4edc-b307-122d8031bc02)


![image](https://github.com/user-attachments/assets/37ef8ef5-5154-4160-a40b-f18d50416a54)




### Steps

ExecuteSQLRecord: Call Snowflake with this SQL to get back a list of users
  SHOW USERS;

QueryRecord: Limit to only RSA users
  SELECT * FROM FLOWFILE
WHERE has_rsa_public_key = 'true'
and  disabled = 'false'

SplitRecord:  Split json records to 1 per flowfile

EvaluateJsonPath:  Extract key attributes from the flowfile including user name.

ExecuteSQLRecord:  Call Snowflake with this SQL to get back the RSA public key
  DESC USER ${username};

QueryRecord:  Limit only to RSA details
  SELECT * FROM FLOWFILE
WHERE property = 'RSA_PUBLIC_KEY' 

SplitRecord:  Split json records to 1 per flowfile

EvaluateJsonPath:  Extract $.value into a key field

RouteOnAttribute:  If has a header, don't add one
  ${rsakey:contains("-----BEGIN PUBLIC KEY")}

ReplaceText:   Replace file with ${rsakey}
(or)
ReplaceText:   Replace file with -----BEGIN PUBLIC KEY-----
${rsakey}
-----END PUBLIC KEY-----

UpdateAttribute:  Set the new filename to sf.pub

PutFile:  Put the public key in a file

ExecuteStreamCommand:  Run Shell Script against that file
  openssl rsa -pubin -in /Users/tspann/Downloads/code/nifi-2.4.0/sf.pub -text -noout

RouteOnContent:   Split out 1024, 2048, 4096 and unknown key size returned by openssl rsa public key validation

PublishSlack:   Push file and details to Slack.

### Slack output


![image](https://github.com/user-attachments/assets/14b5fabd-6529-4387-9dd6-38ff62fba3a1)


#### Resources

* https://emn178.github.io/online-tools/rsa/verify/
* https://dinochiesa.github.io/httpsig/
* https://anycript.com/crypto/rsa
* https://docs.snowflake.com/en/user-guide/key-pair-auth



#### Snowflake Intelligence Generated


Automated RSA Key Validation Workflow with Apache NiFi and Snowflake

This article describes an automated workflow for validating Snowflake users' RSA public keys using Apache NiFi, with results published to Slack for monitoring and alerting purposes.

System Components and Architecture
Apache NiFi serves as the workflow automation platform, integrating with Snowflake for data retrieval and Slack for notifications. The workflow uses ExecuteSQLRecord processors to interact with Snowflake databases  and leverages NiFi's built-in record processing capabilities for data transformation and validation .

Workflow Steps

1. User Discovery and Filtering
- The process begins with ExecuteSQLRecord querying Snowflake to list all users 
- QueryRecord filters for active users with RSA public keys
- Records are split into individual flowfiles for processing

2. Key Retrieval and Processing
- For each user, the workflow retrieves their RSA public key using DESC USER commands 
- The public key information is extracted and formatted with proper PEM headers
- Keys are validated to ensure proper PKCS8 formatting standards 

3. Key Validation Process
- Each public key is saved to a file for validation
- OpenSSL commands verify the key's validity and size
- The process checks for standard key sizes (1024, 2048, 4096 bits)
- Snowflake requires a minimum 2048-bit RSA key pair for authentication 

4. Slack Integration and Reporting
- NiFi's Slack processors handle notification delivery 
- Results include key validation status and size information
- Real-time alerts are sent for any non-compliant keys

Monitoring and Alerting
The workflow provides continuous monitoring of RSA key compliance across the Snowflake environment. The Slack integration enables real-time notifications , allowing security teams to quickly identify and address any key-related issues.

This automated solution helps organizations maintain security compliance by ensuring all RSA keys meet required standards and specifications. The workflow can be scheduled to run periodically or triggered by specific events, providing flexible key validation management .
