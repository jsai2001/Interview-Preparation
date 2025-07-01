# Python Database Connection Troubleshooting and Scenario-Based Interview Questions

This document contains a collection of on-call troubleshooting and scenario-based interview questions focused on connecting Python to databases, specifically Snowflake SQL, with references to general SQL databases where applicable. The questions are grouped by their characteristics to cover various technical and operational aspects.

## 1. Connection Setup and Configuration Issues
These questions focus on establishing and configuring connections to Snowflake or SQL databases using Python.

1. **Scenario: Connection Timeout Error**
   - You attempt to connect to a Snowflake database using the `snowflake.connector` library, but you receive a timeout error. What steps would you take to troubleshoot and resolve this issue?
   - *Key Considerations*: Network issues, firewall settings, Snowflake account configuration, connection string parameters.

```python
import snowflake.connector
import logging

logging.basicConfig(level=logging.DEBUG)

try:
    conn = snowflake.connector.connect(
        user='your_user',
        password='your_password',
        account='your_account.region.cloud',
        timeout=10,  # Set timeout to 10 seconds
        network_timeout=30  # Network timeout for retries
    )
    logging.info("Connection successful")
except snowflake.connector.errors.OperationalError as e:
    logging.error(f"Connection timeout: {e}")
    # Check network connectivity
    import socket
    try:
        socket.create_connection(("snowflake.com", 443), timeout=5)
        logging.info("Network is reachable")
    except socket.timeout:
        logging.error("Network issue: Unable to reach Snowflake")
    # Verify firewall settings
    logging.info("Check firewall rules for outbound connections to Snowflake")
    # Suggest increasing timeout or checking account configuration
    conn = snowflake.connector.connect(
        user='your_user',
        password='your_password',
        account='your_account.region.cloud',
        timeout=30  # Increased timeout
    )
finally:
    if 'conn' in locals():
        conn.close()
        logging.info("Connection closed")    
```

2. **Scenario: Invalid Account Name**
   - A Python script using `snowflake.connector` fails with an error: "Invalid account name provided." How would you debug this issue, and what specific configurations in the connection string should you verify?
   - *Key Considerations*: Snowflake account URL format, region, cloud provider (AWS, Azure, GCP).

```python
import snowflake.connector
import logging

logging.basicConfig(level=logging.DEBUG)

def verify_account_name(account):
    # Expected format: <account>.<region>.<cloud> or <account>
    parts = account.split('.')
    if len(parts) == 1:
        logging.info("No region/cloud specified, assuming default")
    elif len(parts) == 3:
        logging.info(f"Account: {parts[0]}, Region: {parts[1]}, Cloud: {parts[2]}")
    else:
        logging.error("Invalid account format")
        return False
    return True

try:
    account = 'your_account.region.cloud'  # e.g., myaccount.us-east-1.aws
    if verify_account_name(account):
        conn = snowflake.connector.connect(
            user='your_user',
            password='your_password',
            account=account
        )
        logging.info("Connection successful")
    else:
        logging.error("Fix account name format and retry")
except snowflake.connector.errors.DatabaseError as e:
    logging.error(f"Connection failed: {e}")
    # Debug account name
    logging.info("Verify account name format: <account>.<region>.<cloud>")
    logging.info("Check Snowflake account URL in admin console")
finally:
    if 'conn' in locals():
        conn.close()
        logging.info("Connection closed")
```

3. **Scenario: Missing Driver for SQL Database**
   - Your Python application fails to connect to a MySQL database with an error: "No suitable driver found." What could be the cause, and how would you resolve it?
   - *Key Considerations*: Missing `mysql-connector-python` or `pymysql` library, driver installation, import statements.

```python
import logging

logging.basicConfig(level=logging.DEBUG)

try:
    import mysql.connector
except ImportError:
    logging.error("MySQL driver not found. Installing mysql-connector-python")
    import subprocess
    subprocess.check_call(['pip', 'install', 'mysql-connector-python'])
    import mysql.connector

try:
    conn = mysql.connector.connect(
        host='localhost',
        user='your_user',
        password='your_password',
        database='your_database'
    )
    logging.info("MySQL connection successful")
except mysql.connector.Error as e:
    logging.error(f"Connection failed: {e}")
    logging.info("Ensure mysql-connector-python is installed: pip install mysql-connector-python")
finally:
    if 'conn' in locals():
        conn.close()
        logging.info("Connection closed")
```

4. **Scenario: Snowflake Connection Parameters**
   - A colleague reports that a Python script connecting to Snowflake works locally but fails in production with a "Connection refused" error. What parameters in the Snowflake connection configuration might differ between environments?
   - *Key Considerations*: Account, warehouse, database, schema, role, environment-specific credentials.

```python
import snowflake.connector
import os
import logging

logging.basicConfig(level=logging.DEBUG)

def load_env_config(env='prod'):
    configs = {
        'dev': {
            'account': 'dev_account.region.cloud',
            'warehouse': 'dev_warehouse',
            'database': 'dev_db',
            'schema': 'dev_schema',
            'role': 'dev_role'
        },
        'prod': {
            'account': 'prod_account.region.cloud',
            'warehouse': 'prod_warehouse',
            'database': 'prod_db',
            'schema': 'prod_schema',
            'role': 'prod_role'
        }
    }
    return configs.get(env, configs['prod'])

try:
    env = os.getenv('ENV', 'prod')
    config = load_env_config(env)
    conn = snowflake.connector.connect(
        user=os.getenv('SNOWFLAKE_USER', 'your_user'),
        password=os.getenv('SNOWFLAKE_PASSWORD', 'your_password'),
        account=config['account'],
        warehouse=config['warehouse'],
        database=config['database'],
        schema=config['schema'],
        role=config['role']
    )
    logging.info(f"Connected to Snowflake in {env} environment")
except snowflake.connector.errors.OperationalError as e:
    logging.error(f"Connection failed: {e}")
    logging.info("Check environment-specific configs: account, warehouse, database, schema, role")
    logging.info(f"Current config: {config}")
finally:
    if 'conn' in locals():
        conn.close()
        logging.info("Connection closed")
```

5. **Scenario: SSL Certificate Error**
   - When connecting to a Snowflake database, you encounter an SSL certificate verification error. How would you diagnose and fix this issue?
   - *Key Considerations*: SSL settings, CA certificates, `insecure_mode` in Snowflake connector, network proxies.

```python
import snowflake.connector
import logging
import os

logging.basicConfig(level=logging.DEBUG)

try:
    # Attempt connection with SSL verification
    conn = snowflake.connector.connect(
        user='your_user',
        password='your_password',
        account='your_account.region.cloud',
        insecure_mode=False  # Default SSL verification
    )
    logging.info("Connection successful")
except snowflake.connector.errors.OperationalError as e:
    logging.error(f"SSL error: {e}")
    # Try insecure mode for debugging (not recommended for production)
    logging.warning("Attempting connection with insecure_mode=True")
    conn = snowflake.connector.connect(
        user='your_user',
        password='your_password',
        account='your_account.region.cloud',
        insecure_mode=True  # Bypass SSL verification
    )
    logging.info("Connection successful with insecure_mode")
    logging.info("Check CA certificates or proxy settings")
    # Example: Set custom SSL certificates
    # os.environ['REQUESTS_CA_BUNDLE'] = '/path/to/ca-cert.pem'
finally:
    if 'conn' in locals():
        conn.close()
        logging.info("Connection closed")
```

6. **Scenario: Connection Pooling Setup**
   - Your application connects to a Snowflake database but experiences delays due to frequent connection openings. How would you implement connection pooling in Python to optimize performance?
   - *Key Considerations*: Libraries like `sqlalchemy` with connection pooling, Snowflake-specific pooling limitations.

```python
from sqlalchemy import create_engine
from sqlalchemy.pool import QueuePool
import snowflake.connector
import logging

logging.basicConfig(level=logging.DEBUG)

# Snowflake does not support traditional connection pooling; use SQLAlchemy for MySQL
mysql_url = "mysql+mysqlconnector://your_user:your_password@localhost/your_database"
snowflake_url = "snowflake://your_user:your_password@your_account.region.cloud/your_database/your_schema?warehouse=your_warehouse"

try:
    # MySQL connection pooling with SQLAlchemy
    mysql_engine = create_engine(
        mysql_url,
        poolclass=QueuePool,
        pool_size=5,  # Max connections
        max_overflow=10,  # Allow extra connections
        pool_timeout=30  # Wait time for connection
    )
    with mysql_engine.connect() as conn:
        conn.execute("SELECT 1")
        logging.info("MySQL pooled connection successful")

    # Snowflake: Minimal pooling (Snowflake manages connections server-side)
    snowflake_conn = snowflake.connector.connect(
        user='your_user',
        password='your_password',
        account='your_account.region.cloud',
        warehouse='your_warehouse',
        database='your_database',
        schema='your_schema'
    )
    logging.info("Snowflake connection successful")
except Exception as e:
    logging.error(f"Connection error: {e}")
finally:
    if 'snowflake_conn' in locals():
        snowflake_conn.close()
        logging.info("Snowflake connection closed")
    if 'mysql_engine' in locals():
        mysql_engine.dispose()
        logging.info("MySQL engine disposed")
```

7. **Scenario: Environment Variable Mismatch**
   - A Python script fails to connect to Snowflake because the credentials stored in environment variables are incorrect. How would you verify and fix the issue?
   - *Key Considerations*: Use of `os.getenv()`, environment variable names, `.env` files, debugging environment setup.

```python
import snowflake.connector
import os
import logging
from dotenv import load_dotenv

logging.basicConfig(level=logging.DEBUG)

# Load environment variables from .env file
load_dotenv()

def verify_env_vars():
    required_vars = ['SNOWFLAKE_USER', 'SNOWFLAKE_PASSWORD', 'SNOWFLAKE_ACCOUNT']
    for var in required_vars:
        if not os.getenv(var):
            logging.error(f"Missing environment variable: {var}")
            return False
    return True

try:
    if verify_env_vars():
        conn = snowflake.connector.connect(
            user=os.getenv('SNOWFLAKE_USER'),
            password=os.getenv('SNOWFLAKE_PASSWORD'),
            account=os.getenv('SNOWFLAKE_ACCOUNT'),
            warehouse=os.getenv('SNOWFLAKE_WAREHOUSE', 'default_warehouse'),
            database=os.getenv('SNOWFLAKE_DATABASE', 'default_db'),
            schema=os.getenv('SNOWFLAKE_SCHEMA', 'default_schema')
        )
        logging.info("Connection successful")
    else:
        logging.error("Environment variable verification failed")
        logging.info("Check .env file or system environment variables")
except snowflake.connector.errors.DatabaseError as e:
    logging.error(f"Connection failed: {e}")
    logging.info("Run: set SNOWFLAKE_USER=your_user SNOWFLAKE_PASSWORD=your_password SNOWFLAKE_ACCOUNT=your_account")
finally:
    if 'conn' in locals():
        conn.close()
        logging.info("Connection closed")
```

## 2. Authentication and Security Issues
These questions address authentication failures, credential management, and security best practices.

8. **Scenario: Authentication Failure in Snowflake**
   - A Python script fails with an "Authentication failed" error when connecting to Snowflake. What steps would you take to troubleshoot this issue?
   - *Key Considerations*: Username/password, key pair authentication, MFA, token expiration.

```python
# Check credentials and MFA configuration
import snowflake.connector
from snowflake.connector.errors import DatabaseError
import os

try:
    conn = snowflake.connector.connect(
        user=os.getenv('SNOWFLAKE_USER'),
        password=os.getenv('SNOWFLAKE_PASSWORD'),
        account=os.getenv('SNOWFLAKE_ACCOUNT'),
        authenticator='snowflake',  # Use 'externalbrowser' for MFA if needed
        warehouse='COMPUTE_WH',
        database='TEST_DB',
        schema='PUBLIC'
    )
    print("Connection successful")
except DatabaseError as e:
    if "Authentication failed" in str(e):
        print("Authentication failed. Verify username, password, or MFA settings.")
        print("Check environment variables: SNOWFLAKE_USER, SNOWFLAKE_PASSWORD, SNOWFLAKE_ACCOUNT")
        print("For MFA, ensure authenticator='externalbrowser' and browser is configured.")
    raise
finally:
    conn.close()
```

9. **Scenario: Hardcoded Credentials Detected**
   - During a code review, you notice hardcoded Snowflake credentials in a Python script. How would you refactor the code to securely manage credentials?
   - *Key Considerations*: Use of environment variables, secret managers (e.g., AWS Secrets Manager), `keyring` library.

```python
# Refactor to use environment variables and AWS Secrets Manager
import snowflake.connector
import os
import boto3
from dotenv import load_dotenv

# Load environment variables from .env file
load_dotenv()

# Retrieve credentials from AWS Secrets Manager
def get_snowflake_credentials(secret_name, region_name):
    client = boto3.client('secretsmanager', region_name=region_name)
    response = client.get_secret_value(SecretId=secret_name)
    return response['SecretString'].split(':')

# Connect using environment variables or secrets
secret_name = "snowflake/credentials"
region_name = "us-west-2"
user, password = get_snowflake_credentials(secret_name, region_name) if os.getenv('USE_SECRETS') else (os.getenv('SNOWFLAKE_USER'), os.getenv('SNOWFLAKE_PASSWORD'))

conn = snowflake.connector.connect(
    user=user,
    password=password,
    account=os.getenv('SNOWFLAKE_ACCOUNT'),
    warehouse='COMPUTE_WH',
    database='TEST_DB',
    schema='PUBLIC'
)
conn.close()
```

10. **Scenario: Role-Based Access Issue**
    - A Python script can connect to Snowflake but fails to execute queries due to a "Insufficient privileges" error. How would you diagnose and resolve this?
    - *Key Considerations*: Snowflake role configuration, granting permissions, checking active role in connection.

```python
# 10. Scenario: Role-Based Access Issue
# Check and set the correct role during connection
import snowflake.connector
import os

conn = snowflake.connector.connect(
    user=os.getenv('SNOWFLAKE_USER'),
    password=os.getenv('SNOWFLAKE_PASSWORD'),
    account=os.getenv('SNOWFLAKE_ACCOUNT'),
    warehouse='COMPUTE_WH',
    database='TEST_DB',
    schema='PUBLIC',
    role='REQUIRED_ROLE'  # Specify the correct role
)

try:
    # Verify role and privileges
    conn.cursor().execute("SELECT CURRENT_ROLE()")
    current_role = conn.cursor().fetchone()[0]
    print(f"Current role: {current_role}")
    conn.cursor().execute("SELECT * FROM TEST_TABLE LIMIT 1")
except snowflake.connector.errors.ProgrammingError as e:
    if "Insufficient privileges" in str(e):
        print("Insufficient privileges. Verify role permissions or use a different role.")
        conn.cursor().execute("USE ROLE REQUIRED_ROLE")  # Switch role if needed
    raise
finally:
    conn.close()
```

11. **Scenario: Key Pair Authentication Failure**
    - You are using key pair authentication to connect to Snowflake, but the connection fails with an "Invalid private key" error. What could be the cause, and how would you fix it?
    - *Key Considerations*: Key format, passphrase, file permissions, `snowflake.connector` configuration.
```python
# Configure key pair authentication with correct key format and permissions
import snowflake.connector
import os
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.backends import default_backend

# Load private key
with open("snowflake_key.p8", "rb") as key_file:
    private_key = serialization.load_pem_private_key(
        key_file.read(),
        password=os.getenv('KEY_PASSPHRASE').encode() if os.getenv('KEY_PASSPHRASE') else None,
        backend=default_backend()
    )

# Ensure file permissions are secure (e.g., chmod 600 snowflake_key.p8)
os.chmod("snowflake_key.p8", 0o600)

conn = snowflake.connector.connect(
    user=os.getenv('SNOWFLAKE_USER'),
    private_key=private_key,
    account=os.getenv('SNOWFLAKE_ACCOUNT'),
    warehouse='COMPUTE_WH',
    database='TEST_DB',
    schema='PUBLIC'
)
conn.close()
```

12. **Scenario: SQL Injection Vulnerability**
    - A Python script uses string concatenation to build SQL queries for a MySQL database, raising security concerns. How would you rewrite the code to prevent SQL injection when connecting to Snowflake or MySQL?
    - *Key Considerations*: Parameterized queries, `snowflake.connector` bind variables, `sqlalchemy` query builder.
```python
# Use parameterized queries to prevent SQL injection
import snowflake.connector
import os
from sqlalchemy import create_engine

# Snowflake parameterized query
conn = snowflake.connector.connect(
    user=os.getenv('SNOWFLAKE_USER'),
    password=os.getenv('SNOWFLAKE_PASSWORD'),
    account=os.getenv('SNOWFLAKE_ACCOUNT'),
    warehouse='COMPUTE_WH',
    database='TEST_DB',
    schema='PUBLIC'
)

try:
    cursor = conn.cursor()
    user_input = "user_input"
    query = "SELECT * FROM TEST_TABLE WHERE column_name = %s"
    cursor.execute(query, (user_input,))
    results = cursor.fetchall()
except Exception as e:
    print(f"Query error: {e}")
finally:
    conn.close()

# MySQL with SQLAlchemy
engine = create_engine(f"mysql+pymysql://{os.getenv('MYSQL_USER')}:{os.getenv('MYSQL_PASSWORD')}@localhost/TEST_DB")
with engine.connect() as conn:
    result = conn.execute("SELECT * FROM TEST_TABLE WHERE column_name = %s", (user_input,))
```
13. **Scenario: Expired OAuth Token**
    - A Python application using OAuth to authenticate with Snowflake fails with a "Token expired" error. How would you handle token refresh in your script?
    - *Key Considerations*: OAuth refresh token flow, `snowflake.connector` OAuth support, external authentication libraries.
```python
# Implement OAuth token refresh
import snowflake.connector
import os
import requests

def refresh_oauth_token(refresh_token, client_id, client_secret):
    url = "https://<snowflake_account>.snowflakecomputing.com/oauth/token"
    data = {
        "grant_type": "refresh_token",
        "refresh_token": refresh_token,
        "client_id": client_id,
        "client_secret": client_secret
    }
    response = requests.post(url, data=data)
    return response.json()['access_token']

try:
    access_token = refresh_oauth_token(
        refresh_token=os.getenv('SNOWFLAKE_REFRESH_TOKEN'),
        client_id=os.getenv('SNOWFLAKE_CLIENT_ID'),
        client_secret=os.getenv('SNOWFLAKE_CLIENT_SECRET')
    )
    conn = snowflake.connector.connect(
        user=os.getenv('SNOWFLAKE_USER'),
        account=os.getenv('SNOWFLAKE_ACCOUNT'),
        authenticator='oauth',
        token=access_token,
        warehouse='COMPUTE_WH',
        database='TEST_DB',
        schema='PUBLIC'
    )
except snowflake.connector.errors.DatabaseError as e:
    if "Token expired" in str(e):
        print("Token expired. Refreshing token...")
        access_token = refresh_oauth_token(
            os.getenv('SNOWFLAKE_REFRESH_TOKEN'),
            os.getenv('SNOWFLAKE_CLIENT_ID'),
            os.getenv('SNOWFLAKE_CLIENT_SECRET')
        )
        conn = snowflake.connector.connect(
            user=os.getenv('SNOWFLAKE_USER'),
            account=os.getenv('SNOWFLAKE_ACCOUNT'),
            authenticator='oauth',
            token=access_token,
            warehouse='COMPUTE_WH',
            database='TEST_DB',
            schema='PUBLIC'
        )
finally:
    conn.close()
```
## 3. Query Execution and Performance Issues
These questions focus on issues related to executing queries and optimizing performance.

14. **Scenario: Slow Query Performance in Snowflake**
    - A Python script executes a complex query on Snowflake that takes too long to complete. How would you optimize the query and the Python code to improve performance?
    - *Key Considerations*: Query optimization, indexing, caching results, warehouse size in Snowflake.
    - **Solution**:
      ```python
      import snowflake.connector
      from snowflake.connector import DictCursor

      # Connect to Snowflake with optimized warehouse
      conn = snowflake.connector.connect(
          user='your_user',
          password='your_password',
          account='your_account',
          warehouse='OPTIMIZED_WAREHOUSE',  # Use appropriately sized warehouse
          database='your_db',
          schema='your_schema'
      )

      # Optimize query with simplified logic and indexing
      query = """
      SELECT c.customer_id, c.name, COUNT(o.order_id) as order_count
      FROM customers c
      JOIN orders o ON c.customer_id = o.customer_id
      WHERE o.order_date >= '2023-01-01'
      GROUP BY c.customer_id, c.name
      HAVING order_count > 10
      """

      try:
          cursor = conn.cursor(DictCursor)
          # Enable query result caching
          cursor.execute("ALTER SESSION SET USE_CACHED_RESULT = TRUE")
          cursor.execute(query)
          results = cursor.fetchall()
          for row in results:
              print(row)
      finally:
          cursor.close()
          conn.close()
      ```
      - **Notes**: Simplifies query by reducing joins and conditions, uses a larger warehouse for compute-intensive queries, and enables Snowflake's result caching to avoid recomputation.

15. **Scenario: Query Timeout Error**
    - A Python script running a long query on a SQL database times out. What steps would you take to troubleshoot and resolve this issue for both Snowflake and a generic SQL database?
    - *Key Considerations*: Query complexity, database timeout settings, indexing, breaking down queries.
    - **Solution**:
      ```python
      import snowflake.connector
      import mysql.connector
      from snowflake.connector.errors import ProgrammingError

      # Snowflake connection with increased timeout
      snowflake_conn = snowflake.connector.connect(
          user='your_user',
          password='your_password',
          account='your_account',
          warehouse='your_warehouse',
          database='your_db',
          schema='your_schema',
          session_parameters={'QUERY_TIMEOUT': 300}  # Set timeout to 5 minutes
      )

      # MySQL connection with timeout settings
      mysql_conn = mysql.connector.connect(
          host='localhost',
          user='your_user',
          password='your_password',
          database='your_db',
          connect_timeout=300
      )

      # Break down complex query into smaller parts
      snowflake_query = """
      SELECT * FROM large_table
      WHERE date_column BETWEEN '2023-01-01' AND '2023-12-31'
      """
      mysql_query = """
      SELECT * FROM large_table
      WHERE date_column BETWEEN '2023-01-01' AND '2023-12-31'
      LIMIT 1000 OFFSET %s
      """

      try:
          # Snowflake query execution
          snowflake_cursor = snowflake_conn.cursor()
          snowflake_cursor.execute("CREATE INDEX IF NOT EXISTS idx_date ON large_table(date_column)")
          snowflake_cursor.execute(snowflake_query)
          snowflake_results = snowflake_cursor.fetchall()

          # MySQL query execution with pagination
          mysql_cursor = mysql_conn.cursor()
          offset = 0
          while True:
              mysql_cursor.execute(mysql_query, (offset,))
              results = mysql_cursor.fetchall()
              if not results:
                  break
              offset += 1000
      except ProgrammingError as e:
          print(f"Snowflake Error: {e}")
      except mysql.connector.Error as e:
          print(f"MySQL Error: {e}")
      finally:
          snowflake_cursor.close()
          snowflake_conn.close()
          mysql_cursor.close()
          mysql_conn.close()
      ```
      - **Notes**: Increases timeout settings, adds indexing on frequently queried columns, and breaks down large queries into smaller chunks for MySQL using LIMIT/OFFSET.

16. **Scenario: Large Result Set Handling**
    - A Python script retrieves a large result set from Snowflake, causing memory issues. How would you modify the script to handle large datasets efficiently?
    - *Key Considerations*: Fetching in chunks, using cursors, streaming results, `snowflake.connector` fetch_pandas_all().
    - **Solution**:
      ```python
      import snowflake.connector
      import pandas as pd

      conn = snowflake.connector.connect(
          user='your_user',
          password='your_password',
          account='your_account',
          warehouse='your_warehouse',
          database='your_db',
          schema='your_schema'
      )

      query = "SELECT * FROM large_table"

      try:
          cursor = conn.cursor()
          cursor.execute(query)
          
          # Fetch results in chunks
          chunk_size = 10000
          while True:
              rows = cursor.fetchmany(chunk_size)
              if not rows:
                  break
              # Convert chunk to pandas DataFrame for processing
              df = pd.DataFrame(rows, columns=[desc[0] for desc in cursor.description])
              # Process chunk (e.g., save to file)
              df.to_csv('output.csv', mode='a', index=False)
      finally:
          cursor.close()
          conn.close()
      ```
      - **Notes**: Uses `fetchmany()` to retrieve data in chunks, reducing memory usage, and processes results incrementally with pandas for efficient handling.

17. **Scenario: Inconsistent Query Results**
    - A Python script querying Snowflake returns inconsistent results across multiple runs. What could be causing this, and how would you investigate?
    - *Key Considerations*: Data updates, caching, transaction isolation levels, query logic errors.
    - **Solution**:
      ```python
      import snowflake.connector
      from snowflake.connector import DictCursor

      conn = snowflake.connector.connect(
          user='your_user',
          password='your_password',
          account='your_account',
          warehouse='your_warehouse',
          database='your_db',
          schema='your_schema'
      )

      query = """
      SELECT * FROM orders
      WHERE order_date = CURRENT_DATE
      """

      try:
          cursor = conn.cursor(DictCursor)
          # Disable caching to ensure fresh data
          cursor.execute("ALTER SESSION SET USE_CACHED_RESULT = FALSE")
          # Set transaction isolation level
          cursor.execute("SET TRANSACTION ISOLATION LEVEL = SERIALIZABLE")
          cursor.execute(query)
          results = cursor.fetchall()
          
          # Log results for debugging
          with open('query_log.txt', 'a') as f:
              f.write(str(results))
          
          # Check for query logic errors
          if not results:
              print("No results returned. Verify query conditions.")
      except Exception as e:
          print(f"Error: {e}")
      finally:
          cursor.close()
          conn.close()
      ```
      - **Notes**: Disables result caching, sets strict transaction isolation to avoid concurrent data changes, and logs results to detect query logic issues.

18. **Scenario: Batch Insert Performance**
    - Inserting thousands of rows into Snowflake using Python is slow. How would you optimize the batch insert process?
    - *Key Considerations*: Bulk loading, `COPY INTO` command, batch size optimization, parallel execution.
    - **Solution**:
      ```python
      import snowflake.connector
      import pandas as pd
      from snowflake.connector.pandas_tools import write_pandas

      conn = snowflake.connector.connect(
          user='your_user',
          password='your_password',
          account='your_account',
          warehouse='your_warehouse',
          database='your_db',
          schema='your_schema'
      )

      # Sample data
      data = pd.DataFrame({
          'id': range(10000),
          'name': [f'name_{i}' for i in range(10000)],
          'value': [i * 1.5 for i in range(10000)]
      })

      try:
          # Use write_pandas for efficient bulk loading
          success, nchunks, nrows, _ = write_pandas(
              conn=conn,
              df=data,
              table_name='target_table',
              auto_create_table=True
          )
          print(f"Inserted {nrows} rows in {nchunks} chunks")
          
          # Alternative: Use COPY INTO for CSV
          cursor = conn.cursor()
          cursor.execute("PUT file://data.csv @my_stage")
          cursor.execute("""
          COPY INTO target_table
          FROM @my_stage/data.csv
          FILE_FORMAT = (TYPE = CSV SKIP_HEADER = 1)
          """)
      finally:
          cursor.close()
          conn.close()
      ```
      - **Notes**: Uses `write_pandas` for optimized bulk inserts and `COPY INTO` for CSV-based loading, with appropriate batch sizing to balance performance and memory.

19. **Scenario: Snowflake Warehouse Overload**
    - Your Python application causes a Snowflake warehouse to become overloaded, resulting in slow query performance. How would you address this issue?
    - *Key Considerations*: Warehouse scaling, query prioritization, splitting workloads, monitoring usage.
    - **Solution**:
      ```python
      import snowflake.connector
      from snowflake.connector import DictCursor

      conn = snowflake.connector.connect(
          user='your_user',
          password='your_password',
          account='your_account',
          warehouse='your_warehouse',
          database='your_db',
          schema='your_schema'
      )

      try:
          cursor = conn.cursor(DictCursor)
          # Scale up warehouse for heavy workloads
          cursor.execute("ALTER WAREHOUSE your_warehouse SET WAREHOUSE_SIZE = 'LARGE'")
          
          # Set query priority
          cursor.execute("ALTER SESSION SET QUERY_TAG = 'high_priority_job'")
          
          # Split workload into smaller queries
          queries = [
              "SELECT * FROM large_table WHERE region = 'A'",
              "SELECT * FROM large_table WHERE region = 'B'"
          ]
          
          for query in queries:
              cursor.execute(query)
              results = cursor.fetchall()
              print(f"Processed {len(results)} rows")
          
          # Monitor warehouse usage
          cursor.execute("SELECT * FROM TABLE(GET_WAREHOUSE_USAGE_HISTORY())")
          usage = cursor.fetchall()
          print(usage)
      finally:
          cursor.close()
          conn.close()
      ```
      - **Notes**: Scales warehouse size dynamically, tags queries for prioritization, splits large queries to reduce load, and monitors usage via Snowflake’s built-in functions.

## 4. Error Handling and Debugging
These questions cover handling and debugging errors during database operations.

20. **Scenario: Connection Dropped Mid-Query**
    - A Python script loses its connection to Snowflake during a long-running query. How would you handle this gracefully in your code?
    - *Key Considerations*: Retry logic, connection timeout settings, error handling with try-except, reconnection strategies.
    - **Solution**:
      Use a retry mechanism with exponential backoff to handle connection drops and ensure proper reconnection. The code below implements retry logic using the `tenacity` library and manages connection timeouts.

```python
import snowflake.connector
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type
from snowflake.connector.errors import OperationalError

# Configure connection parameters
conn_params = {
    'account': 'your_account',
    'user': 'your_user',
    'password': 'your_password',
    'database': 'your_database',
    'schema': 'your_schema',
    'warehouse': 'your_warehouse',
    'timeout': 30  # Set connection timeout
}

# Retry decorator for handling connection errors
@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=4, max=10),
    retry=retry_if_exception_type(OperationalError)
)
def execute_query(query):
    try:
        # Establish connection
        conn = snowflake.connector.connect(**conn_params)
        cursor = conn.cursor()
        cursor.execute(query)
        results = cursor.fetchall()
        cursor.close()
        conn.close()
        return results
    except OperationalError as e:
        print(f"Connection error: {e}. Retrying...")
        raise  # Re-raise for retry
    except Exception as e:
        print(f"Error: {e}")
        raise

# Example usage
try:
    results = execute_query("SELECT * FROM large_table")
    print(results)
except Exception as e:
    print(f"Failed after retries: {e}")
```

21. **Scenario: Syntax Error in Query**
    - A Python script fails with a syntax error when executing a dynamically generated Snowflake query. How would you debug and prevent this issue?
    - *Key Considerations*: Logging queries, parameterized queries, validating SQL syntax, testing queries.
    - **Solution**:
      Log the generated query and use parameterized queries to prevent syntax errors. The code below logs the query and validates it before execution.

```python
import snowflake.connector
import logging

# Configure logging
logging.basicConfig(level=logging.DEBUG, format='%(asctime)s - %(levelname)s - %(message)s')

# Connection parameters
conn_params = {
    'account': 'your_account',
    'user': 'your_user',
    'password': 'your_password',
    'database': 'your_database',
    'schema': 'your_schema'
}

def execute_safe_query(query, params=None):
    try:
        conn = snowflake.connector.connect(**conn_params)
        cursor = conn.cursor()
        # Log the query for debugging
        logging.debug(f"Executing query: {query} with params: {params}")
        # Use parameterized query to avoid syntax errors
        cursor.execute(query, params or ())
        results = cursor.fetchall()
        cursor.close()
        conn.close()
        return results
    except snowflake.connector.errors.ProgrammingError as e:
        logging.error(f"Syntax error in query: {e}")
        raise
    except Exception as e:
        logging.error(f"Unexpected error: {e}")
        raise

# Example usage with parameterized query
try:
    query = "SELECT * FROM employees WHERE department = %s AND salary > %s"
    params = ('Engineering', 50000)
    results = execute_safe_query(query, params)
    print(results)
except Exception as e:
    print(f"Query failed: {e}")
```

22. **Scenario: Data Type Mismatch**
    - A Python script inserting data into Snowflake fails due to a data type mismatch error. How would you identify and resolve this issue?
    - *Key Considerations*: Schema validation, data type casting, Snowflake data type mapping, error logs.
    - **Solution**:
      Validate data types against the Snowflake schema and cast inputs as needed. The code below checks the schema and casts data before insertion.

```python
import snowflake.connector
import logging

# Configure logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

# Connection parameters
conn_params = {
    'account': 'your_account',
    'user': 'your_user',
    'password': 'your_password',
    'database': 'your_database',
    'schema': 'your_schema'
}

def get_column_types(table_name):
    conn = snowflake.connector.connect(**conn_params)
    cursor = conn.cursor()
    cursor.execute(f"DESCRIBE TABLE {table_name}")
    schema = {row[1]: row[2] for row in cursor.fetchall()}
    cursor.close()
    conn.close()
    return schema

def insert_data(table_name, data):
    try:
        # Get schema to validate data types
        schema = get_column_types(table_name)
        logging.info(f"Table schema: {schema}")
        
        # Cast data to match schema
        for row in data:
            for key, value in row.items():
                if 'NUMBER' in schema[key] and isinstance(value, str):
                    row[key] = int(value) if value else None
                elif 'FLOAT' in schema[key] and isinstance(value, str):
                    row[key] = float(value) if value else None
                # Add more type mappings as needed

        conn = snowflake.connector.connect(**conn_params)
        cursor = conn.cursor()
        columns = ', '.join(data[0].keys())
        placeholders = ', '.join(['%s'] * len(data[0]))
        query = f"INSERT INTO {table_name} ({columns}) VALUES ({placeholders})"
        cursor.executemany(query, [list(row.values()) for row in data])
        conn.commit()
        cursor.close()
        conn.close()
        logging.info("Data inserted successfully")
    except snowflake.connector.errors.ProgrammingError as e:
        logging.error(f"Data type mismatch: {e}")
        raise
    except Exception as e:
        logging.error(f"Unexpected error: {e}")
        raise

# Example usage
data = [
    {'id': '1', 'name': 'John', 'salary': '50000.0'},
    {'id': '2', 'name': 'Jane', 'salary': '60000.0'}
]
try:
    insert_data('employees', data)
except Exception as e:
    print(f"Insert failed: {e}")
```

23. **Scenario: Transaction Rollback Failure**
    - A Python script fails to roll back a transaction in a SQL database after an error. How would you ensure proper transaction management in Snowflake and other SQL databases?
    - *Key Considerations*: Transaction scope, commit/rollback logic, `snowflake.connector` transaction handling.
    - **Solution**:
      Use a context manager to ensure proper transaction management, with explicit commit and rollback logic. The code below demonstrates transaction handling for Snowflake.

```python
import snowflake.connector
import logging

# Configure logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

# Connection parameters
conn_params = {
    'account': 'your_account',
    'user': 'your_user',
    'password': 'your_password',
    'database': 'your_database',
    'schema': 'your_schema'
}

def execute_transaction(queries):
    conn = None
    try:
        conn = snowflake.connector.connect(**conn_params)
        conn.autocommit(False)  # Disable autocommit for transaction control
        cursor = conn.cursor()
        
        for query in queries:
            logging.info(f"Executing: {query}")
            cursor.execute(query)
        
        conn.commit()
        logging.info("Transaction committed successfully")
        cursor.close()
    except Exception as e:
        logging.error(f"Transaction failed: {e}")
        if conn:
            conn.rollback()
            logging.info("Transaction rolled back")
        raise
    finally:
        if conn:
            conn.close()

# Example usage
queries = [
    "INSERT INTO employees (id, name) VALUES (1, 'John')",
    "UPDATE employees SET salary = 50000 WHERE id = 1"
]
try:
    execute_transaction(queries)
except Exception as e:
    print(f"Transaction failed: {e}")
```

24. **Scenario: Deadlock in SQL Database**
    - Your Python application encounters a deadlock error when performing concurrent updates on a MySQL database. How would you troubleshoot and prevent deadlocks?
    - *Key Considerations*: Transaction isolation levels, query order, locking strategies, retry mechanisms.
    - **Solution**:
      Implement retry logic and enforce consistent query order to prevent deadlocks. The code below uses `pymysql` for MySQL with retry handling and sets an appropriate isolation level.

```python
import pymysql
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type
import logging

# Configure logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

# Connection parameters
conn_params = {
    'host': 'localhost',
    'user': 'your_user',
    'password': 'your_password',
    'database': 'your_database'
}

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=5),
    retry=retry_if_exception_type(pymysql.err.OperationalError)
)
def execute_concurrent_update(query):
    try:
        conn = pymysql.connect(**conn_params, charset='utf8mb4', cursorclass=pymysql.cursors.DictCursor)
        conn.autocommit(False)
        cursor = conn.cursor()
        
        # Set transaction isolation level to avoid deadlocks
        cursor.execute("SET TRANSACTION ISOLATION LEVEL READ COMMITTED")
        
        # Ensure consistent query order to prevent deadlocks
        logging.info(f"Executing: {query}")
        cursor.execute(query)
        conn.commit()
        cursor.close()
        conn.close()
        logging.info("Update successful")
    except pymysql.err.OperationalError as e:
        logging.error(f"Deadlock detected: {e}")
        if conn:
            conn.rollback()
        raise
    except Exception as e:
        logging.error(f"Unexpected error: {e}")
        if conn:
            conn.rollback()
        raise
    finally:
        if conn:
            conn.close()

# Example usage
query = "UPDATE employees SET salary = salary + 1000 WHERE id = 1"
try:
    execute_concurrent_update(query)
except Exception as e:
    print(f"Update failed: {e}")
```

## 5. Advanced Scenarios and Edge Cases
These solutions address complex or less common scenarios that may arise.

25. **Scenario: Snowflake Time Travel Query Failure**
    - A Python script attempts to use Snowflake’s Time Travel feature to query historical data but fails with an "Invalid timestamp" error. How would you troubleshoot this?
    - *Key Considerations*: Time Travel retention period, timestamp format, query syntax, data retention policies.
    ```python
    import snowflake.connector
    from datetime import datetime
    import pytz

    # Establish connection
    conn = snowflake.connector.connect(
        user='your_user',
        password='your_password',
        account='your_account',
        warehouse='your_warehouse',
        database='your_database',
        schema='your_schema'
    )

    # Ensure timestamp is in correct format (UTC)
    timestamp = datetime.now(pytz.UTC).strftime('%Y-%m-%d %H:%M:%S %Z')
    try:
        cursor = conn.cursor()
        # Check retention period
        cursor.execute("SHOW TABLES LIKE 'your_table'")
        retention_period = cursor.fetchone()[6]  # Retention time in days
        print(f"Table retention period: {retention_period} days")

        # Use correct Time Travel syntax
        query = f"""
        SELECT * FROM your_table 
        AT (TIMESTAMP => '{timestamp}'::TIMESTAMP_TZ)
        """
        cursor.execute(query)
        results = cursor.fetchall()
        print(results)
    except snowflake.connector.errors.ProgrammingError as e:
        print(f"Error: {e}")
        # Check if timestamp is within retention period
        print("Verify timestamp is within table's retention period and format is correct.")
    finally:
        cursor.close()
        conn.close()
    ```

26. **Scenario: Multi-Database Integration**
    - Your Python application needs to connect to both Snowflake and PostgreSQL to transfer data between them. How would you design and troubleshoot this integration?
    - *Key Considerations*: Connection management, data type compatibility, transaction consistency, error handling.
    ```python
    import snowflake.connector
    import psycopg2
    import pandas as pd

    # Snowflake connection
    snow_conn = snowflake.connector.connect(
        user='your_user',
        password='your_password',
        account='your_account',
        warehouse='your_warehouse',
        database='your_database',
        schema='your_schema'
    )

    # PostgreSQL connection
    pg_conn = psycopg2.connect(
        dbname='your_db',
        user='your_user',
        password='your_password',
        host='localhost',
        port='5432'
    )

    try:
        # Fetch data from Snowflake
        snow_cursor = snow_conn.cursor()
        snow_cursor.execute("SELECT * FROM your_snowflake_table")
        snow_data = pd.DataFrame(snow_cursor.fetchall(), columns=[desc[0] for desc in snow_cursor.description])

        # Ensure data type compatibility
        snow_data = snow_data.astype(str)  # Convert to string to handle type mismatches

        # Insert into PostgreSQL with transaction
        pg_cursor = pg_conn.cursor()
        for _, row in snow_data.iterrows():
            pg_cursor.execute(
                "INSERT INTO your_pg_table (column1, column2) VALUES (%s, %s)",
                (row['COLUMN1'], row['COLUMN2'])
            )
        pg_conn.commit()
    except Exception as e:
        pg_conn.rollback()
        print(f"Error: {e}")
        # Log and troubleshoot type mismatches or connection issues
    finally:
        snow_cursor.close()
        pg_conn.close()
        snow_conn.close()
    ```

27. **Scenario: Handling Special Characters**
    - A Python script fails to insert data containing special characters (e.g., emojis) into Snowflake. How would you resolve this issue?
    - *Key Considerations*: UTF-8 encoding, Snowflake string handling, Python string encoding, database collation.
    ```python
    import snowflake.connector
    import sys

    # Ensure UTF-8 encoding
    sys.stdout.reconfigure(encoding='utf-8')

    conn = snowflake.connector.connect(
        user='your_user',
        password='your_password',
        account='your_account',
        warehouse='your_warehouse',
        database='your_database',
        schema='your_schema'
    )

    try:
        cursor = conn.cursor()
        # Create table with UTF-8 support
        cursor.execute("CREATE OR REPLACE TABLE test_table (data VARCHAR(100)) COLLATE 'utf8'")

        # Insert data with special characters
        special_data = "Hello 😊 World"
        cursor.execute(
            "INSERT INTO test_table (data) VALUES (%s)",
            (special_data.encode('utf-8').decode('utf-8'),)
        )
        conn.commit()
        print("Data inserted successfully")
    except snowflake.connector.errors.ProgrammingError as e:
        print(f"Error: {e}")
        # Ensure table collation and Python encoding are set to UTF-8
    finally:
        cursor.close()
        conn.close()
    ```

28. **Scenario: Dynamic Schema Switching**
    - A Python script needs to dynamically switch between multiple Snowflake schemas during runtime. How would you implement and troubleshoot this functionality?
    - *Key Considerations*: Dynamic connection parameters, `USE SCHEMA` command, error handling, schema permissions.
    ```python
    import snowflake.connector

    conn = snowflake.connector.connect(
        user='your_user',
        password='your_password',
        account='your_account',
        warehouse='your_warehouse',
        database='your_database'
    )

    schemas = ['schema1', 'schema2']
    try:
        cursor = conn.cursor()
        for schema in schemas:
            # Dynamically switch schema
            cursor.execute(f"USE SCHEMA {schema}")
            # Verify schema change
            cursor.execute("SELECT CURRENT_SCHEMA()")
            current_schema = cursor.fetchone()[0]
            print(f"Switched to schema: {current_schema}")
            # Execute query in current schema
            cursor.execute("SELECT * FROM your_table LIMIT 1")
            print(cursor.fetchall())
    except snowflake.connector.errors.ProgrammingError as e:
        print(f"Error: {e}")
        # Check schema permissions and existence
        cursor.execute("SHOW SCHEMAS")
        print([row[1] for row in cursor.fetchall()])
    finally:
        cursor.close()
        conn.close()
    ```

29. **Scenario: Snowflake Stage File Upload Failure**
    - A Python script fails to upload a file to a Snowflake stage for bulk loading. What steps would you take to diagnose and fix this issue?
    - *Key Considerations*: Stage configuration, file format, `snowflake.connector` file upload methods, permissions.
    ```python
    import snowflake.connector
    import os

    conn = snowflake.connector.connect(
        user='your_user',
        password='your_password',
        account='your_account',
        warehouse='your_warehouse',
        database='your_database',
        schema='your_schema'
    )

    try:
        cursor = conn.cursor()
        # Create internal stage
        cursor.execute("CREATE OR REPLACE STAGE my_stage FILE_FORMAT = (TYPE = CSV)")

        # Verify file exists and format
        file_path = "data.csv"
        if not os.path.exists(file_path):
            raise FileNotFoundError("File not found")

        # Upload file to stage
        cursor.execute(f"PUT file://{file_path} @my_stage AUTO_COMPRESS = TRUE")
        print("File uploaded successfully")

        # Verify upload
        cursor.execute("LIST @my_stage")
        print(cursor.fetchall())
    except snowflake.connector.errors.ProgrammingError as e:
        print(f"Error: {e}")
        # Check stage permissions and file format
        cursor.execute("SHOW STAGES")
        print(cursor.fetchall())
    finally:
        cursor.close()
        conn.close()
    ```

30. **Scenario: Concurrent Connections Overload**
    - Multiple Python scripts connecting to Snowflake cause the account to exceed its connection limit. How would you mitigate this issue?
    - *Key Considerations*: Connection pooling, limiting concurrent connections, Snowflake account settings, load balancing.
    ```python
    from snowflake.connector import connect
    from sqlalchemy import create_engine
    from urllib.parse import quote_plus

    # Use SQLAlchemy for connection pooling
    snowflake_url = (
        f"snowflake://your_user:{quote_plus('your_password')}@your_account/"
        f"your_database/your_schema?warehouse=your_warehouse"
    )
    engine = create_engine(snowflake_url, pool_size=5, max_overflow=10)

    try:
        # Limit concurrent connections
        with engine.connect() as conn:
            result = conn.execute("SELECT CURRENT_WAREHOUSE(), CURRENT_DATABASE()")
            print(result.fetchall())

            # Example query
            conn.execute("SELECT * FROM your_table LIMIT 1")
            print("Query executed successfully")
    except Exception as e:
        print(f"Error: {e}")
        # Check Snowflake account connection limits
    finally:
        engine.dispose()  # Clean up connections
    ```

31. **Scenario: Cross-Region Snowflake Connection**
    - A Python script in one cloud region fails to connect to a Snowflake instance in a different region. How would you troubleshoot and resolve this?
    - *Key Considerations*: Snowflake cross-region replication, network latency, DNS configuration, region-specific account URLs.
    ```python
    import snowflake.connector

    # Use region-specific account URL
    conn = snowflake.connector.connect(
        user='your_user',
        password='your_password',
        account='your_account.us-west-2',  # Specify region
        warehouse='your_warehouse',
        database='your_database',
        schema='your_schema'
    )

    try:
        cursor = conn.cursor()
        # Verify connection
        cursor.execute("SELECT CURRENT_REGION()")
        region = cursor.fetchone()[0]
        print(f"Connected to region: {region}")

        # Execute test query
        cursor.execute("SELECT * FROM your_table LIMIT 1")
        print(cursor.fetchall())
    except snowflake.connector.errors.OperationalError as e:
        print(f"Error: {e}")
        # Check DNS and network latency
        print("Verify account URL and network connectivity")
    finally:
        cursor.close()
        conn.close()
    ```

32. **Scenario: Handling NULL Values**
    - A Python script querying Snowflake misinterprets NULL values, causing application errors. How would you handle NULLs in your code?
    - *Key Considerations*: NULL checking, Python `None` handling, default values, query logic.
    ```python
    import snowflake.connector
    import pandas as pd

    conn = snowflake.connector.connect(
        user='your_user',
        password='your_password',
        account='your_account',
        warehouse='your_warehouse',
        database='your_database',
        schema='your_schema'
    )

    try:
        cursor = conn.cursor()
        cursor.execute("SELECT column1, column2 FROM your_table")
        df = pd.DataFrame(cursor.fetchall(), columns=[desc[0] for desc in cursor.description])

        # Handle NULL values
        df.fillna({'column1': 'default_value', 'column2': 0}, inplace=True)
        for index, row in df.iterrows():
            column1 = row['column1'] if row['column1'] is not None else 'default_value'
            column2 = row['column2'] if row['column2'] is not None else 0
            print(f"Row {index}: {column1}, {column2}")
    except snowflake.connector.errors.ProgrammingError as e:
        print(f"Error: {e}")
        # Verify query logic and NULL handling
    finally:
        cursor.close()
        conn.close()
    ```

33. **Scenario: Snowflake Query Tag Issues**
    - A Python script sets query tags in Snowflake for cost tracking, but the tags are not appearing in the query history. How would you debug this?
    - *Key Considerations*: `snowflake.connector` query tag settings, session parameters, tag visibility, permissions.
    ```python
    import snowflake.connector

    conn = snowflake.connector.connect(
        user='your_user',
        password='your_password',
        account='your_account',
        warehouse='your_warehouse',
        database='your_database',
        schema='your_schema'
    )

    try:
        cursor = conn.cursor()
        # Set query tag
        cursor.execute("ALTER SESSION SET QUERY_TAG = 'cost_tracking_tag'")
        cursor.execute("SELECT * FROM your_table LIMIT 1")
        print(cursor.fetchall())

        # Verify query tag in history
        cursor.execute("SELECT * FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY()) WHERE QUERY_TAG = 'cost_tracking_tag'")
        print(cursor.fetchall())
    except snowflake.connector.errors.ProgrammingError as e:
        print(f"Error: {e}")
        # Check session permissions and tag settings
        cursor.execute("SHOW PARAMETERS LIKE 'QUERY_TAG'")
        print(cursor.fetchall())
    finally:
        cursor.close()
        conn.close()
    ```

34. **Scenario: Legacy SQL Driver Conflict**
    - A Python application using an older SQL driver for MySQL conflicts with a newer Snowflake connector. How would you resolve this dependency issue?
    - *Key Considerations*: Python virtual environments, dependency management, driver compatibility, `pip` resolution.
    ```python
    # Create a virtual environment
    # Command-line: python -m venv snowflake_env
    # Activate: source snowflake_env/bin/activate (Linux/Mac) or snowflake_env\Scripts\activate (Windows)

    # Install compatible drivers in virtual environment
    # Command-line: pip install snowflake-connector-python==2.9.0 mysql-connector-python==8.0.33

    import snowflake.connector
    import mysql.connector

    # Snowflake connection
    snow_conn = snowflake.connector.connect(
        user='your_user',
        password='your_password',
        account='your_account',
        warehouse='your_warehouse',
        database='your_database',
        schema='your_schema'
    )

    # MySQL connection
    mysql_conn = mysql.connector.connect(
        host='localhost',
        user='your_user',
        password='your_password',
        database='your_db'
    )

    try:
        snow_cursor = snow_conn.cursor()
        snow_cursor.execute("SELECT * FROM your_table LIMIT 1")
        print(snow_cursor.fetchall())

        mysql_cursor = mysql_conn.cursor()
        mysql_cursor.execute("SELECT * FROM your_mysql_table LIMIT 1")
        print(mysql_cursor.fetchall())
    except Exception as e:
        print(f"Error: {e}")
        # Verify dependency versions: pip list
    finally:
        snow_cursor.close()
        mysql_cursor.close()
        snow_conn.close()
        mysql_conn.close()
    ```

35. **Scenario: Snowflake Stream Failure**
    - A Python script monitoring a Snowflake stream for change data capture fails to detect new data. What could be the cause, and how would you fix it?
    - *Key Considerations*: Stream configuration, polling logic, change tracking, Snowflake stream retention.
    ```python
    import snowflake.connector
    import time

    conn = snowflake.connector.connect(
        user=' your_user',
        password='your_password',
        account='your_account',
        warehouse='your_warehouse',
        database='your_database',
        schema='your_schema'
    )

    try:
        cursor = conn.cursor()
        # Create stream
        cursor.execute("CREATE OR REPLACE STREAM my_stream ON TABLE your_table")

        # Polling loop to monitor stream
        while True:
            cursor.execute("SELECT * FROM my_stream")
            changes = cursor.fetchall()
            if changes:
                print(f"New changes detected: {changes}")
                # Process changes
            else:
                print("No new changes")
            time.sleep(60)  # Poll every minute

            # Check stream retention
            cursor.execute("SHOW STREAMS LIKE 'my_stream'")
            stream_info = cursor.fetchone()
            print(f"Stream retention: {stream_info}")
    except snowflake.connector.errors.ProgrammingError as e:
        print(f"Error: {e}")
        # Verify stream configuration and table change tracking
        cursor.execute("SHOW TABLES LIKE 'your_table'")
        print(cursor.fetchall())
    finally:
        cursor.close()
        conn.close()
    ```

## 6. Monitoring and Maintenance
These questions focus on ongoing maintenance and monitoring of database connections.

36. **Scenario: Connection Leak in Long-Running Application**
    - A long-running Python application connecting to Snowflake causes a connection leak, exhausting database resources. How would you detect and prevent this?
    - *Key Considerations*: Connection closing, context managers, monitoring tools, `snowflake.connector` cleanup.
    - *Solution*:
      ```python
      import snowflake.connector
      from contextlib import contextmanager
      import logging
      import psutil
      import os

      # Set up logging
      logging.basicConfig(level=logging.INFO)
      logger = logging.getLogger(__name__)

      @contextmanager
      def snowflake_connection(config):
          conn = None
          try:
              conn = snowflake.connector.connect(**config)
              yield conn
          except Exception as e:
              logger.error(f"Connection error: {e}")
              raise
          finally:
              if conn:
                  conn.close()
                  logger.info("Connection closed")

      def monitor_connections():
          process = psutil.Process(os.getpid())
          open_files = process.open_files()
          logger.info(f"Open file descriptors: {len(open_files)}")
          return len(open_files)

      # Example usage
      config = {
          'user': os.getenv('SNOWFLAKE_USER'),
          'password': os.getenv('SNOWFLAKE_PASSWORD'),
          'account': os.getenv('SNOWFLAKE_ACCOUNT'),
          'warehouse': os.getenv('SNOWFLAKE_WAREHOUSE'),
          'database': os.getenv('SNOWFLAKE_DATABASE'),
          'schema': os.getenv('SNOWFLAKE_SCHEMA')
      }

      # Monitor and limit connections
      if monitor_connections() > 50:  # Threshold for open connections
          logger.warning("Too many open connections, aborting")
          raise ResourceWarning("Connection limit exceeded")

      with snowflake_connection(config) as conn:
          cursor = conn.cursor()
          cursor.execute("SELECT CURRENT_VERSION()")
          result = cursor.fetchone()
          logger.info(f"Snowflake version: {result[0]}")
      ```

37. **Scenario: Monitoring Query Performance**
    - How would you implement logging and monitoring in a Python script to track Snowflake query performance and errors?
    - *Key Considerations*: Logging libraries, Snowflake query history, performance metrics, alerting.
    - *Solution*:
      ```python
      import snowflake.connector
      import logging
      import time
      from datetime import datetime
      import os

      # Set up logging with file and console handlers
      logging.basicConfig(
          level=logging.INFO,
          format='%(asctime)s - %(levelname)s - %(message)s',
          handlers=[
              logging.FileHandler('query_performance.log'),
              logging.StreamHandler()
          ]
      )
      logger = logging.getLogger(__name__)

      def execute_query_with_monitoring(conn, query):
          start_time = time.time()
          try:
              cursor = conn.cursor()
              cursor.execute("ALTER SESSION SET QUERY_TAG = 'performance_monitoring'")
              cursor.execute(query)
              results = cursor.fetchall()
              execution_time = time.time() - start_time
              logger.info(f"Query executed successfully: {query[:50]}... Time: {execution_time:.2f}s")
              return results
          except Exception as e:
              execution_time = time.time() - start_time
              logger.error(f"Query failed: {query[:50]}... Error: {e}, Time: {execution_time:.2f}s")
              # Send alert (e.g., via email or monitoring service)
              send_alert(f"Query error: {e}")
              raise

      def send_alert(message):
          # Placeholder for alerting (e.g., integrate with Slack or email)
          logger.warning(f"Alert: {message}")

      # Example usage
      config = {
          'user': os.getenv('SNOWFLAKE_USER'),
          'password': os.getenv('SNOWFLAKE_PASSWORD'),
          'account': os.getenv('SNOWFLAKE_ACCOUNT'),
          'warehouse': os.getenv('SNOWFLAKE_WAREHOUSE'),
          'database': os.getenv('SNOWFLAKE_DATABASE'),
          'schema': os.getenv('SNOWFLAKE_SCHEMA')
      }

      conn = snowflake.connector.connect(**config)
      query = "SELECT * FROM my_table WHERE date > '2023-01-01'"
      results = execute_query_with_monitoring(conn, query)
      conn.close()
      ```

38. **Scenario: Scheduled Database Maintenance**
    - A Python script fails during a scheduled Snowflake maintenance window. How would you handle such interruptions in your application?
    - *Key Considerations*: Retry logic, maintenance schedules, error handling, failover strategies.
    - *Solution*:
      ```python
      import snowflake.connector
      import time
      import logging
      import os
      from retrying import retry

      # Set up logging
      logging.basicConfig(level=logging.INFO)
      logger = logging.getLogger(__name__)

      # Retry decorator for maintenance-related errors
      @retry(
          stop_max_attempt_number=3,
          wait_exponential_multiplier=1000,
          wait_exponential_max=10000,
          retry_on_exception=lambda e: isinstance(e, snowflake.connector.errors.OperationalError)
      )
      def execute_query_with_retry(conn, query):
          cursor = conn.cursor()
          cursor.execute(query)
          return cursor.fetchall()

      def run_query_with_maintenance_handling(config, query):
          try:
              conn = snowflake.connector.connect(**config)
              results = execute_query_with_retry(conn, query)
              logger.info(f"Query executed successfully: {query[:50]}...")
              return results
          except snowflake.connector.errors.OperationalError as e:
              logger.error(f"Maintenance window likely in progress: {e}")
              # Failover strategy: Log and switch to backup process or delay
              logger.info("Switching to failover: delaying execution")
              time.sleep(3600)  # Wait 1 hour for maintenance to complete
              conn = snowflake.connector.connect(**config)
              results = execute_query_with_retry(conn, query)
              return results
          finally:
              if 'conn' in locals():
                  conn.close()

      # Example usage
      config = {
          'user': os.getenv('SNOWFLAKE_USER'),
          'password': os.getenv('SNOWFLAKE_PASSWORD'),
          'account': os.getenv('SNOWFLAKE_ACCOUNT'),
          'warehouse': os.getenv('SNOWFLAKE_WAREHOUSE'),
          'database': os.getenv('SNOWFLAKE_DATABASE'),
          'schema': os.getenv('SNOWFLAKE_SCHEMA')
      }

      query = "SELECT * FROM my_table"
      results = run_query_with_maintenance_handling(config, query)
      ```

39. **Scenario: Database Version Upgrade**
    - After a Snowflake version upgrade, a Python script encounters compatibility issues. How would you identify and resolve these issues?
    - *Key Considerations*: Checking release notes, updating `snowflake.connector`, testing in a staging environment.
    - *Solution*:
      ```python
      import snowflake.connector
      import logging
      import subprocess
      import os

      # Set up logging
      logging.basicConfig(level=logging.INFO)
      logger = logging.getLogger(__name__)

      def check_snowflake_version(conn):
          cursor = conn.cursor()
          cursor.execute("SELECT CURRENT_VERSION()")
          version = cursor.fetchone()[0]
          logger.info(f"Current Snowflake version: {version}")
          return version

      def update_snowflake_connector():
          try:
              subprocess.check_call(['pip', 'install', '--upgrade', 'snowflake-connector-python'])
              logger.info("Snowflake connector updated successfully")
          except subprocess.CalledProcessError as e:
              logger.error(f"Failed to update snowflake-connector-python: {e}")
              raise

      def test_connection(config):
          try:
              conn = snowflake.connector.connect(**config)
              version = check_snowflake_version(conn)
              cursor = conn.cursor()
              cursor.execute("SELECT * FROM my_table LIMIT 1")
              logger.info("Test query executed successfully")
              conn.close()
          except Exception as e:
              logger.error(f"Compatibility issue detected: {e}")
              # Check release notes manually or update connector
              update_snowflake_connector()
              # Retry connection
              conn = snowflake.connector.connect(**config)
              cursor = conn.cursor()
              cursor.execute("SELECT * FROM my_table LIMIT 1")
              logger.info("Test query executed successfully after update")
              conn.close()

      # Example usage
      config = {
          'user': os.getenv('SNOWFLAKE_USER'),
          'password': os.getenv('SNOWFLAKE_PASSWORD'),
          'account': os.getenv('SNOWFLAKE_ACCOUNT'),
          'warehouse': os.getenv('SNOWFLAKE_WAREHOUSE'),
          'database': os.getenv('SNOWFLAKE_DATABASE'),
          'schema': os.getenv('SNOWFLAKE_SCHEMA')
      }

      # Run in staging environment first
      test_connection(config)
      ```

40. **Scenario: Automated Credential Rotation**
    - Your organization requires periodic rotation of Snowflake credentials. How would you automate this process in a Python application?
    - *Key Considerations*: Secret management, credential refresh scripts, secure storage, minimal downtime.
    - *Solution*:
      ```python
      import snowflake.connector
      import boto3  # For AWS Secrets Manager
      import json
      import logging
      import os

      # Set up logging
      logging.basicConfig(level=logging.INFO)
      logger = logging.getLogger(__name__)

      def get_new_credentials(secret_id):
          client = boto3.client('secretsmanager')
          response = client.get_secret_value(SecretId=secret_id)
          return json.loads(response['SecretString'])

      def rotate_credentials(secret_id, config):
          try:
              # Fetch new credentials from AWS Secrets Manager
              new_creds = get_new_credentials(secret_id)
              config['user'] = new_creds['user']
              config['password'] = new_creds['password']
              logger.info("Credentials rotated successfully")

              # Test new credentials
              conn = snowflake.connector.connect(**config)
              cursor = conn.cursor()
              cursor.execute("SELECT CURRENT_VERSION()")
              logger.info(f"Connection test successful with new credentials")
              conn.close()
              return config
          except Exception as e:
              logger.error(f"Credential rotation failed: {e}")
              # Rollback to previous credentials or failover
              raise

      # Example usage
      config = {
          'user': os.getenv('SNOWFLAKE_USER'),
          'password': os.getenv('SNOWFLAKE_PASSWORD'),
          'account': os.getenv('SNOWFLAKE_ACCOUNT'),
          'warehouse': os.getenv('SNOWFLAKE_WAREHOUSE'),
          'database': os.getenv('SNOWFLAKE_DATABASE'),
          'schema': os.getenv('SNOWFLAKE_SCHEMA')
      }

      secret_id = 'snowflake/credentials'
      updated_config = rotate_credentials(secret_id, config)

      # Schedule this script to run periodically (e.g., via AWS Lambda or cron)
      ```