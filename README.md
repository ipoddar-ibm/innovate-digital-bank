# Innovate: Digital Bank
Innovate is a dummy digital bank composed of a set of microservices that communicate with each other; created to demonstrate cloud-native web apps.
<!-- A live version deployed on a kubernetes cluster in IBM Cloud is available here: -->

Table of Contents
=================

   * [About](#innovate-digital-bank)
      * [Flow](#flow)
   * [Guide: Deploying on IBM Cloud Private](#guide-deploying-on-ibm-cloud-private)
      * [Creating an instance of MongoDB](#creating-an-instance-of-mongodb)
      * [Configuring your Environment Variables](#configuring-your-environment-variables)
      * [Deploying all Components](#deploying-all-components)
      * [(Optional) Adding Support with Watson Conversation](#optional-adding-support-with-watson-conversation)
   * [Docs](#docs)
      * [Microservices](#microservices)
         * [Portal [3000:30060]](#portal-300030060)
         * [Authentication [3200:30100]](#authentication-320030100)
            * [Endpoints:](#endpoints)
               * [/api/user/create](#apiusercreate)
               * [/api/user/authenticate](#apiuserauthenticate)
               * [/api/user/get](#apiuserget)
         * [Accounts [3400:30120]](#accounts-340030120)
            * [Endpoints:](#endpoints-1)
               * [/api/accounts/create](#apiaccountscreate)
               * [/api/accounts/get](#apiaccountsget)
               * [/api/accounts/deposit](#apiaccountsdeposit)
               * [/api/accounts/withdraw](#apiaccountswithdraw)
               * [/api/accounts/drop](#apiaccountsdrop)
         * [Transactions [3600:30140]](#transactions-360030140)
            * [Endpoints:](#endpoints-2)
               * [/api/transactions/create](#apitransactionscreate)
               * [/api/transactions/get](#apitransactionsget)
               * [/api/transactions/drop](#apitransactionsdrop)
         * [Bills [3800:30160]](#bills-380030160)
            * [Endpoints:](#endpoints-3)
               * [/api/bills/create](#apibillscreate)
               * [/api/bills/get](#apibillsget)
               * [/api/bills/drop](#apibillsdrop)
         * [Support [4000:30180]](#support-400030180)
         * [Userbase [3600:30140]](#userbase-360030140)

## Flow
![Demo architecture](docs/flow.png)

# Guide: Deploying on IBM Cloud Private

## Creating an instance of MongoDB
This demo heavily depends on mongo as a session & data store.

#### 1. Create a persistent volume
Give it a name and a capacity, choose storage type _**Hostpath**_, and add a _**path parameter**_

![Persistent Volume](docs/1.png)

#### 2. Create a persistent volume claim
Give it a name and a storage request value

![Persistent Volume Claim](docs/2.jpg)

#### 3. Create and configure mongo
From the catalog, choose MongoDb. Give it a **_name_**, specify the **_existing volume claim name_**, and give it a *_password_*

![Mongo](docs/3.jpg)

![Mongo](docs/4.jpg)

#### 4. Get your mongo connection string
Your mongo connection string will be in the following format:
```
mongodb://<USERNAME>:<PASSWORD>@<HOST>:<PORT>/<DATABASE_NAME>
```

Almost all your microservices need it; keep it safe!

## Configuring your Environment Variables
Each of the 8 microservices must have a _**.env**_ file.

An example is already provided within each folder. From the directory of each microservice, copy the example file, rename it to _**.env**_, and fill it with the appropriate values.

For example, from within the /innovate folder, navigate into the accounts folder

```
cd accounts
```

Next, copy and rename the _**.env.example**_ folder

```
cp .env.example .env
```

Finally, edit your .env folder and add your Mongodb connection string

#### Repeat those steps for all microservices. In addition to your mongo url, the portal microservice will need the address of your ICP.

## Deploying all Components
#### 1. Add your ICP's address to your hosts file
Add an entry to your /etc/hosts file as follows

```
<YOUR_ICP_IP_ADDRESS> mycluster.icp
```

#### 2. Login to docker

```
docker login mycluster.icp:8500
```

#### 3. Configure kubectl
From your ICP's dashboard, copy the kubectl commands under admin > configure client

![kubectl config](docs/5.png)

#### 4. Deploy
Finally, navigate to each microservice, and run the following command
```
bx dev deploy
```
_If you don't have the IBM Cloud Developer Tools CLI installed, get it [here](https://console.bluemix.net/docs/cli/reference/bluemix_cli/download_cli.html) first_

## (Optional) Adding Support with Watson Conversation
The support microservice connects to an instance of Watson Conversation on IBM Cloud to simulate a chat with a virtual support agent.

#### 1. Create an instance of Watson Conversation
From the [IBM Cloud Catalog](bluemix.net), choose Watson Conversation, and click create.

![Watson Conversation](docs/6.png)

#### 2. Import the support workspace
Import the [support workspace](/support/conversation-workspace.json) into your newly created Watson Conversation instance

![Watson Conversation](docs/7.png)

#### 3. Get your credentials
Navigate to the deploy tab and copy your username, password, and workspace ID

![Watson Conversation](docs/8.png)

#### 4. Edit your .env file
From within the support folder, edit your .env to include your newly acquired credentials

#### 5. Deploy
Redeploy the microservice, the support feature should now be accessible through the portal.

```
bx dev deployed
```

# Docs

## Microservices

### Portal [3000:30060]

Loads the UI and takes care of user sessions. Communicates with all other microservices.

### Authentication [3200:30100]

Handles user profile creation, as well as login & logout.

#### Endpoints:

##### /api/user/create

Description: Creates a new user account

Method: POST

Example input:

```
{
  uuid: String,
  name: String,
  email: String,
  phone: String,
  gender: String,
  dob: String,
  eid: String,
  password: String
}
```

##### /api/user/authenticate

Description: Authenticates a user

Method: POST

Example input:

```
{
  email: String,
  password: String
}
```

##### /api/user/get

Description: Returns a list of all users

Method: GET

### Accounts [3400:30120]

Handles creation, management, and retrieval of a user's banking accounts.

#### Endpoints:

##### /api/accounts/create

Description: Creates a new user account

Method: POST

Example input:

```
{
  uuid: String,
  type: String,
  currency: String,
}
```
Notes:

The parameter uuid links the account to a user's unique identifier. Type has to be one of the following: current, savings, credit, prepaid

##### /api/accounts/get

Description: Retrieves a user's accounts

Method: POST

Example input:

```
{
  uuid: String
}
```

##### /api/accounts/deposit

Description: Deposits an amount to a user's account

Method: POST

Example input:

```
{
  number: String,
  amount: Number
}
```

Notes:

The parameter number references an account

##### /api/accounts/withdraw

Description: Withdraws an amount from a user's account

Method: POST

Example input:

```
{
  number: String,
  amount: Number
}
```

##### /api/accounts/drop

Description: Drops the accounts collection

Method: GET

### Transactions [3600:30140]

Handles creation and retrieval of transactions

#### Endpoints:

##### /api/transactions/create

Description: Creates a new transaction

Method: POST

Example input:

```
{
  uuid: String,
  amount: String,
  currency: String,
  description: String,
  date: String,
  category: String
}
```

##### /api/transactions/get

Description: Retrieves a user's transactions

Method: POST

Example input:

```
{
  uuid: String
}
```

##### /api/transactions/drop

Description: Drops the transactions collection

Method: GET

### Bills [3800:30160]

Handles creation, payment, and retrieval of bills

#### Endpoints:

##### /api/bills/create

Description: Creates a new bill

Method: POST

Example input:

```
{
  uuid: String,
  category: String,
  entity: String,
  account_no: String,
  amount: String,
  date: String
}
```

##### /api/bills/get

Description: Retrieves a user's bills

Method: POST

Example input:

```
{
  uuid: String
}
```

##### /api/bills/drop

Description: Drops the bills collection

Method: GET

### Support [4000:30180]

Handles communication with Watson Conversation on IBM Cloud to enable a dummy support chat feature.

### Userbase [3600:30140]

Simulates a fake userbase for the app. Periodically loops through all user accounts and adds randomized bills and transactions for them.
