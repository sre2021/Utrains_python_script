## Write a script that will check if all AWS users have been created correctly as far as password, MFA set up and tag. This script should run every day for security purposes.

## The script is below:

``` python
import boto3
from botocore.exceptions import ClientError
import schedule
import time

def list_uncorrect_users():
    _iam = boto3.client('iam')
    users = _iam.list_users()
    _iam_users = []
    _uncorrect_users=[]
    for i in users['Users']:
        _iam_users.append(i['UserName'])

    for users in _iam_users:
        mfa =_iam.list_mfa_devices(UserName=users)
        users_tags= _iam.list_user_tags(UserName=users)
        email_tag = list(filter(lambda tag: tag['Key'] == 'email', users_tags['Tags']))
        try:
            user_profile=_iam.get_login_profile(UserName=users)
            user_profile_date=user_profile['LoginProfile']['CreateDate']
            #print(users, user_profile_date)
        except ClientError as e:
            if (len(mfa['MFADevices'])==0) and (len(email_tag)==0):
                _uncorrect_users.append(users)
                print(users)
                #print(mfa['MFADevices'])
                print("******************")
    print(_uncorrect_users)

# run the script every day by 06:00 am for security purposes
schedule.every().day.at("06:00").do(list_uncorrect_users)

while True:
    # run all pending scheduler
    schedule.run_pending()
    time.sleep(1)
```
