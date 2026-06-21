---
layout: post
title: "AWS Demo: Getting Started in the Cloud"
description: "A step-by-step guide of MLOps basics in cloud infrastructure"
---

## SIADS 699: Getting Started in the Cloud

One challenge a lot of teams encounter in their capstone project is sharing data. In the past we've seen teams send CSVs via slack, upload massive files to git, and some store their data in Google Drive. 

I think AWS is the best way to share data among your team. Here we'll discuss some benefits of doing so, while also acknowledging some tradeoffs of using AWS for your project. 

**Before you get started it's very important to mention that it's possible to rack up a bill in AWS. Using AWS is NOT required for this course. The University of Michigan, the MADS program, or your instructors are NOT responsible for any fees / bills you incur in AWS.**

_____________
**If you are interested in reading and writing files to the cloud while working in your local VS Code then this setup is for you.**

- Module 0: [Do You Need the Cloud?](#module-0)
- Module 1: [Setting up the AWS Account](#module-1)
- Module 2: [Setting up the Admin Role](#module-2)
- Module 3: [Setting up the Team Accounts](#module-3)
- Module 4: [Accessing S3 from VS Code ](#module-4)
- Optional: [Setting up Spending Threshold Alerting ](#module-5)


After your complete the steps above, your team will be able to write and read from the cloud with a single line of code like this:


* Saving a dataframe to the cloud:
```python
import pandas as pd
import numpy as np

# Create a fake dataframe with 10 records
np.random.seed(0)
df = pd.DataFrame({
    'feature_1': np.random.rand(10),
    'feature_2': np.random.rand(10),
    'feature_3': np.random.randint(0, 100, 10),
    'feature_4': np.random.choice(['A', 'B', 'C'], 10),
    'target': np.random.randint(0, 2, 10)
})

# Save data to S3
df.to_csv("s3://umich-capstone-project/data.csv", index=False)
```

* Reading data from the cloud:
```python
import pandas as pd

# Read data from S3
df = pd.read_csv("s3://umich-capstone-project/data.csv")
```
<a id="module-0"></a>
## SIADS 699 - Module 0: Is using AWS worth the effort?

### What is AWS?

### What is S3?
S3 is AWS's file storage solution. S3 stands for "Scalable Storage Solution," you can kind of think of it as a giant file storage system (like Finder on Mac or File Explorer on Windows).

However, the main benefit here is that you can easily share the file storage solution among your teammates, see more below. 

### Tradeoff of the Cloud
**You can rack up a bill.**
* Using AWS is NOT required for this course. The University of Michigan, the MADS program, or your instructors are NOT responsible for any fees / bills you incur in AWS.
* With that said, S3 storage is incredibly cheap. As of April 2026, they're only charging $0.023 per GB of data per month that you store in S3. 
    - At this price it would be very difficult to spend more than $5 throughout the capstone project, but it's something to be aware of.
* Check current pricing at https://aws.amazon.com/s3/pricing/

### Benefits of the cloud
* After you set it up, saving data in AWS S3 is very easy. You can easily backup data with one line of code:

```python
import pandas as pd

df.to_csv("s3://capstone-project-mcallister/teams.csv", index=False)
```
* Sharing data among your team is also very straightforward, pulling data down also only requires one line of code:

```python
import pandas as pd

df = pd.read_csv("s3://capstone-project-mcallister/teams.csv")
```

* You can do the same thing for model objects:

```python
### Create trained model object:

from sklearn.dummy import DummyClassifier
from sklearn.model_selection import train_test_split
import pandas as pd
import numpy as np

# Create a fake dataframe with 10 records
np.random.seed(0)
df = pd.DataFrame({
    'feature_1': np.random.rand(10),
    'feature_2': np.random.rand(10),
    'feature_3': np.random.randint(0, 100, 10),
    'feature_4': np.random.choice(['A', 'B', 'C'], 10),
    'target': np.random.randint(0, 2, 10)
})

# Split out target and features
X = df.drop('target', axis=1)
y = df['target']

# Train-test split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Create and train the dummy classifier
clf = DummyClassifier(strategy='most_frequent')
clf.fit(X_train, y_train)
```

```python
### Save the model to S3

import pickle
import boto3

# Save model locally to disk
pickle.dump(clf, open('dummy_classifier_local_copy.pkl', 'wb'))

# Upload to S3
s3_client = boto3.client('s3')
s3_client.upload_file(
    'dummy_classifier_local_copy.pkl', # Name of model object in local file system
    'capstone-project-mcallister',     # Name of folder in S3
    'dummy_classifier.pkl'             # What you want the file named in S3
)

print("Dummy classifier saved to S3!")
```

* Like with datasets, anyone on your team with access to the S3 bucket can read in this trained model object now:

```python
import boto3
import pickle
import io

# Create S3 client
s3_client = boto3.client('s3')

# Download the model from S3
response = s3_client.get_object(
    Bucket='capstone-project-mcallister',
    Key='dummy_classifier.pkl'
)

# Load the model from the response
model_buffer = io.BytesIO(response['Body'].read())
clf = pickle.load(model_buffer)
```

### Another tradeoff of using the Cloud:

**It takes time to set up.**
* We'll cover it in the next module, but setting up your team in AWS takes some effort (around 40 minutes). 

--
Being comfortable in the cloud is a good skill for any data scientist - if nothing else this is a great reason to use AWS in your project workflow. If this feels worth it to you, head to [the second module](https://github.com/chrismca13/aws-demo/tree/main/1_setting_up_the_aws_account)!


<a id="module-1"></a>
## SIADS 699 - Module 1: Download Setting up the Root User
One person on the team will be the "Root User." This person's responsibilites will include:
* Setting up the team's AWS accounts.
* Maintaining responsibility for billing (ie you will supply your own credit card)
* Managing access for your team's access and roles.

Whoever chooses to do this on your team doesn't need to be familiar with the cloud, but they should be excited about using it. 

### Visual Follow Along
This [Youtube Video](https://youtu.be/NhDYbskXRgc?si=wYFAvPrt7RrC4fX-&t=4775) (from 1:19:34 - 1:30:00) does a good job explaining the root user role, and how to set up other accounts with limited access.

### Pre-Requisites
Before getting started you need the following:
* An email address (your @umich.edu account will not be accepted by AWS)
* A credit card
* Be sure to run these steps on your personal laptop, there's no guarantee this will work on a laptop from work. 

Also assuming this is your first time connecting AWS to your local VS code you'll need to download the AWS CLI interface from here:
* https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html?

![alt text]({{ '../images/aws-demo/aws-cli.png' | relative_url }}){: width="70%"}

### Setting up your AWS account
1. Go to https://aws.amazon.com/console/
2. Click the Create Account button
3. Enter your email
4. It's optional, but I recommend assigning an account name. Something like `umich-capstone-YOURLASTNAME` is a good one. 
5. Set up a password.
6. Opt for the free plan.
7. Supply your credit card info
8. If they ask for a support plan, opt for the free version.
9. Congrats! You have now successfully set up the root user role. 

### What's next
In the [next module](https://github.com/chrismca13/aws-demo/tree/main/2_setting_up_the_admin_role) we will:
* Tell you how to set up an `admin` role for you to work from.
* Talk through how to set up roles with limited access for your teammates.

<a id="module-2"></a>
## SIADS 699 - Module 2: Setting up Admin Access
Now that the Root User role is set up, you can create an admin role for you to work out of. 

### Setting up an Admin Role
It's a best practice to stay out of the root user role when possible. The first thing you should do is create an `admin-role` by doing the following:
1. Search and click on `IAM` in the search bar at the top of the home page:
![alt text]({{ '../images/aws-demo/iam.png' | relative_url }}){: width="70%"}


2. On the side panel on the left click `IAM Users` and click `Create User`
![alt text]({{ '../images/aws-demo/create_user.png' | relative_url }}){: width="70%"}

3. Create a role named `admin-role`
4. Click the `Create Group` button on the top right of the `User Groups` box.
5. This will give you access to a bunch of pre-made access policies created by AWS. The one I recommend adding to your `admin-role` is called `AdministratorAccess` policy. Assing a name to the group too. In the example below I called it `admin`. The setup should look like what you see below. Click `Create User Group` when you're ready:

![alt text]({{ '../images/aws-demo/admin-access.png' | relative_url }}){: width="70%"}


6. Check the box next to your newly created `admin` group and click the orange next button:
![alt text]({{ '../images/aws-demo/create-group.png' | relative_url }}){: width="70%"}


### Logging in to the Admin Role
0. It's a best practice to only log in to the root-user when you really need to. Most of your work should be done in the admin role from here on out. 
1. Click on `Users` in the side bar on the left.
2. You should see your newly created role.
3. Click on the role, navigate to the Security Credentials tab, and click the Enable Console Access button:
![alt text]({{ '../images/aws-demo/console_access.png' | relative_url }}){: width="70%"}
4. Click the orange Enable Console Access button. This generates your user name and one time password. Save it and don't lose it. Click the `Download .csv file` to be safe. 
5. Copy the `Console sign-in link URL` and paste it into your browser. As the admin of the project you'll be using this role throughout the project. **Favorite this link so you can easily get back to the log in**
6. Sign in with your username and password (that you hopefully downloaded) and you're in!


[In the next module](https://github.com/chrismca13/aws-demo/tree/main/3_setting_team_accounts), we'll cover how to get access to S3 for you and your team so you all can easily save and share dataset. 

<a id="module-3"></a>
## SIADS 699 - Module 3: Setting up your Team's Accounts
Now that you have your admin role ready we're going to create accounts for you and your team. 

### Setting up your team's access group.
0. You should still be logged in to the `admin-role` from the previous section. Next we'll follow a similar series of steps to create accounts to get your team set up.
1. Search `IAM` again and click on the `IAM` service.
2. Click `IAM user groups` on the left side panel.
3. Click `Create group`. This time we're going to give S3 access to your team so you can easily share datasets.
4. This part is **VERY important.** Here we have two options. We can either give the team full read and write access to S3, or just read access. 
    * Giving them write access opens the possibility to store a massive dataset in S3, meaning you (as the admin / root user) would get billed for it. 
    * Giving them read access only means they won't be able to contribute to dataset creation. 
    * I recommend giving full access since S3 storage is very cheap and the odds of incurring a big bill are low, but know that you're opening yourself up to risk of losing money. 
4. Assuming you want to give your team read and write access give the name `full-s3-access` to user group, do not check any of the users in the second box, check the box next to `AmazonS3FullAccess` policy, and click the `Create user group`. It should look like this when you're done:
    * If you only want to provide only read access, simply check the box next to `AmazonS3ReadOnlyAccess` instead. Update the name of the role too so it says `read-s3-access`

![alt text]({{ '../images/aws-demo/s3-access-group.png' | relative_url }}){: width="70%"}


### Setting up your team's accounts.
1. Go back to `IAM` and click the `IAM users` on the left side panel.
2. Click the orange `Create user` button in the top left
3. Here you'll create an account for you teammate. Call it something like `firstname-lastname-MADS`. Click next.
4. Give them access to the s3 policy you chose above by checking the box next to the User Group and click next.
5. Click the orange `Create user` button in the bottom right. 
6. Repeat steps 2-5 for all team members on your team.

### Giving you team access to S3:
1. Go back to `IAM` and click `IAM users` on the left side panel.
2. Click the blue link on you teammates name:
![alt text]({{ '../images/aws-demo/click-user.png' | relative_url }}){: width="70%"}


3. Click the `Security credentials` banner across the top and click the `Create access key` button about halfway down the page:
![alt text]({{ '../images/aws-demo/access-key.png' | relative_url }}){: width="70%"}

4. Click the `Command Line Interface (CLI)` circle under "Use case", check the box at the bottom, and click `Next`

5. Type in a description for the key. Something like `FirstName_LastName_key` is great.

6. You won't be able to generate this key again, so it's very important you don't lose it. Like before I recommend click the `Download .csv file` button and sharing it with your teammate.

![alt text]({{ '../images/aws-demo/access-key-download.png' | relative_url }}){: width="70%"}


7. Repeat steps 1-6 for all of your teammates.

### Giving yourself access to connect to S3 locally in VS Code. These steps are similar to what you did before, but now you're doing it for you admin role.
1. Go back to `IAM` and click the `IAM users` on the left side panel.
2. Click the blue link on your `admin-role` name.
3. Click the `Security Credentials` again.
4. Click `Create access key`
5. Fill in the `Command Line Interface (CLI)` bubble, check the box at the bottom, and click Next
6. Give it a description, something like `admin_role_CLI_access`
7. Download the access Key as a CSV, and keep it somewhere safe. We'll need it for our next step. 


[In the final module](https://github.com/chrismca13/aws-demo/tree/main/4_accessing_S3_from_VS_code) we'll show you how to use these credentials to connect to AWS in your local VS Code environment. 


<a id="module-4"></a>
## SIADS 699 - Module 4: Using S3 from VS Code
Now that your roles are created and you gave your team access to S3, you can read share datasets from local VS code!

### Logging in to AWS from VS Code
1. Open the terminal window in VS code (command + J on Mac)
2. Type ```aws configure``` into your terminal. This will prompt you for the following pieces of information. Fill them out:

```bash
# Enter your details when prompted:
# AWS Access Key ID: YOUR_ACCESS_KEY (from the CSV you downloaded for yourself)
# AWS Secret Access Key: YOUR_SECRET_KEY (from the same CSV)
# Default region name: us-east-2
# Default output format: json
```
- If this errored out, make sure you downloaded the `AWS CLI` package [listed as a pre-requisite in module 1. ](https://github.com/chrismca13/aws-demo/blob/main/1_setting_up_the_aws_account/README.md#pre-requisites)

3. Let's test the connection!

* Fire up a jupyter notebook and run this code to test saving a CSV to the cloud:

```python
from sklearn.dummy import DummyClassifier
from sklearn.model_selection import train_test_split
import pandas as pd
import numpy as np

# Create a fake dataframe with 10 records
np.random.seed(0)
df = pd.DataFrame({
    'feature_1': np.random.rand(10),
    'feature_2': np.random.rand(10),
    'feature_3': np.random.randint(0, 100, 10),
    'feature_4': np.random.choice(['A', 'B', 'C'], 10),
    'target': np.random.randint(0, 2, 10)
})

df.to_csv("s3://umich-capstone-project/data.csv", index=False)
```

* After your team gets access, try running this code:
```python
import pandas as pd

df = pd.read_csv("s3://umich-capstone-project/data.csv")
```

The (optional but recommended) [5th module](https://github.com/chrismca13/aws-demo/tree/main/5_setting_up_spend_alerting) will show you how to set up budget alerts to make sure you're not spending money you don't intend to. 

### Final Thoughts
If you have any questions or feedback please post them in the SIADS 699 channel or DM me (Chris McAllister) and we'll help get things sorted out. 

If this was interesting to you at all and you'd like to know more about the cloud, a good way to learn more is by obtaining [AWS Certifications.](https://aws.amazon.com/certification/)

The `Certificed Cloud Practioner` --> `AWS Certified Machine Learning Engineer` --> `Certified Data Engineer` is a great path to focus on after you complete the MADS program. Good luck!

<a id="module-5"></a>
## Optional: Setting up Spend Alerts
Now that you've got your team set up, you've tested the connection, and everything is working it's time to take some steps to reduce the likelihood of you getting a huge AWS bill. 

Like we said before, AWS is not required for this course and your instructors, the University of Michigan, and the MADS program are NOT responsible for any bills you rack up in AWS. 

### Setting up a Budget Alert:

1. Sign out of the `admin-role` 
2. Log in as the root user you set up way back in Module 1.  
3. Search for `Billing and Cost Management` in the search bar above and click it.
4. Click `Budgets` on the side panel on the left. Everything should look like this at this point:
![alt text]({{ '../images/aws-demo/budgets.png' | relative_url }}){: width="70%"}

5. Click the orange `Create budget` button in the top right. 
6. On the next page only change the following:
- Update the budget name to whatever you want, I called mine `My $0.05 Budget Alert`
- Update the `Email Recipients` to whatever email you check most frequenty. 
- The set up should look like this:

![alt text]({{ '../images/aws-demo/budget-setup.png' | relative_url }}){: width="70%"}


7. Click the orange `Create budget` button at the bottom right. 
8. Click the blue link on your newly created budget on the next page. 
9. Click the `Edit` button in the top right corner. 
10. Here you can customize to make sure you get alerted to whatever dollar amount your comfortable with. If you're only using AWS for data storage in S3, this is a good set up unders `Set budget amount`
- This will send you an email if you ever spend more than $.05 on a day:

![alt text]({{ '../images/aws-demo/budget-amount.png' | relative_url }}){: width="70%"}


11. Leave the other setting as is, scroll to the bottom, and click the orange `Next` button. 
12. Optionally, you can set additional alterting here that will give you an email if you start to approach the threshold you set in step 10. 
13. Next you can set up a budget action, I recommend skipping this for now and clicking `Next`
14. Click save - your budget is set up now!

With how affordable S3 storage there's a good chance you'll stay under your $.05 budget, but it doesn't hurt to have the alert just in case. 