### Smart Home Management App - Lab Instructions

**Objective:**
In this lab, you will create a Smart Home Management App using various AWS services. The app will allow users to control smart home devices, store device settings, and manage authentication. The setup includes using Amazon RDS, AWS Lambda, API Gateway, Cognito, and CloudWatch.

##### 1. Amazon RDS Setup (Relational Database Service)

###### Step 1.1: Create an RDS Database
1. Navigate to the Amazon RDS console.
2. Click on "Create Database."
3. Select a database engine (e.g., MySQL or PostgreSQL).
4. Choose Free Tier or Dev/Test for the database template.
5. Instance Type: Select db.t3.micro.
6. Storage: Use General Purpose SSD with a maximum of 20GB.
7. Ensure the database is publicly accessible (for the lab).
8. Database Name: Set a name (e.g., SmartHomeDB).
9. Master Username/Password: Set up the admin credentials.
10. Launch the database.

###### Step 1.2: Configure Database Connectivity

Set up VPC Security Groups to allow access from your application and development environment.

###### Step 1.3: Create Tables

Access your RDS database via Cloud9 or a local MySQL client.
Run the following SQL commands to create the necessary tables:

sql

CREATE TABLE UserSettings (
    UserID SERIAL PRIMARY KEY,
    DeviceID VARCHAR(100),
    DeviceState VARCHAR(50),
    Preferences TEXT
);

CREATE TABLE DeviceLogs (
    LogID SERIAL PRIMARY KEY,
    DeviceID VARCHAR(100),
    LogTime TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    LogMessage TEXT
);


##### 2. AWS Lambda Functions (Backend Logic)
###### Step 2.1: Create Lambda Functions
1. Go to the AWS Lambda console.
2. Create Function and choose Author from Scratch.
3.Name the function (e.g., UpdateDeviceState).
4.Runtime: Choose Python or Node.js.
5.Role: Use the pre-configured LabRole.
6.Code: Add the following logic to update device states in the RDS database:

python

import pymysql
import os

DB_HOST = os.environ['DB_HOST']
DB_USER = os.environ['DB_USER']
DB_PASS = os.environ['DB_PASS']
DB_NAME = os.environ['DB_NAME']

def lambda_handler(event, context):
    connection = pymysql.connect(host=DB_HOST, user=DB_USER, password=DB_PASS, db=DB_NAME)
    try:
        with connection.cursor() as cursor:
            sql = "UPDATE UserSettings SET DeviceState=%s WHERE DeviceID=%s"
            cursor.execute(sql, (event['DeviceState'], event['DeviceID']))
            connection.commit()
        return {"statusCode": 200, "body": "Device state updated"}
    except Exception as e:
        return {"statusCode": 500, "body": str(e)}
    finally:
        connection.close()

###### Step 2.2: Set Environment Variables

1.In Lambda, go to Configuration > Environment Variables.
2.Add these variables:
- DB_HOST: Your RDS endpoint.
- DB_USER: The RDS master username.
- DB_PASS: The RDS master password.
- DB_NAME: The name of your RDS database.

##### 3. API Gateway for Frontend Communication
###### Step 3.1: Create API Gateway
1.Navigate to API Gateway and create a REST API.
2.Create Resource /devices.
3.Add Method: Choose POST to invoke the Lambda function (UpdateDeviceState).
4.Deploy the API to a stage (e.g., prod).

###### Step 3.2: Test the API
1.Use Postman or curl to test the API.
2.Ensure the request updates device states in the RDS database via Lambda.

##### 4. User Authentication with Amazon Cognito
###### Step 4.1: Set Up Cognito User Pool
1.Go to Amazon Cognito and create a User Pool.
2.Configure Email as the sign-up method.
3.Enable MFA if needed.
4.Set up password policies and create the user pool.

###### Step 4.2: Add Cognito Authorizer to API Gateway
1.In API Gateway, go to Authorization.
2.Create Cognito User Pool Authorizer and link it to your user pool.
3.Secure the API so that only authenticated users can interact with it.

##### 5. Monitoring and Logging with CloudWatch
###### Step 5.1: Enable CloudWatch Logging for Lambda
1.Go to CloudWatch and enable logging for the Lambda function.
2.Monitor Lambda execution, errors, and performance.

###### Step 5.2: Set Up CloudWatch Alarms
1.Create Alarms for Lambda function errors or API Gateway latency.
2.Configure notifications with SNS for alarm triggers.

##### 6. Frontend Development (Cloud9 or EC2)
###### Step 6.1: Use Cloud9 for Development
1.Launch Cloud9 and build the frontend using React, Vue.js, or Angular.
2.Use Axios or Fetch API to communicate with your API Gateway.

###### Step 6.2: Example Frontend Code (React)

javascript

import React, { useState } from 'react';
import axios from 'axios';

function DeviceControl() {
  const [deviceID, setDeviceID] = useState('');
  const [deviceState, setDeviceState] = useState('');

  const updateDeviceState = () => {
    axios.post('https://your-api-endpoint/devices', {
      DeviceID: deviceID,
      DeviceState: deviceState
    })
    .then(response => console.log(response.data))
    .catch(error => console.error(error));
  };

  return (
    <div>
      <input value={deviceID} onChange={e => setDeviceID(e.target.value)} placeholder="Device ID" />
      <input value={deviceState} onChange={e => setDeviceState(e.target.value)} placeholder="Device State" />
      <button onClick={updateDeviceState}>Update Device</button>
    </div>
  );
}

export default DeviceControl;

##### 7. Testing and Deployment
###### Step 7.1: End-to-End Testing
1.Test the app by using the frontend to send device updates to the backend.
2.Verify that updates are stored in the RDS database and displayed in CloudWatch logs.

###### Step 7.2: Deploy the Frontend
1.Deploy the frontend to Amazon S3 (for static sites) or CloudFront.
2.Ensure secure integration with API Gateway and Cognito for user authentication.




