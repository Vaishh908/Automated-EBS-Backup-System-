# Automated EBS Backup System Using AWS Lambda

### (On-Demand Serverless Backup)

---

##  Introduction

Data backup is a critical part of cloud infrastructure to ensure data safety and recovery. This project automates the process of creating Amazon EBS snapshots using AWS Lambda instead of manually performing backups from the console.

It demonstrates how serverless computing can simplify cloud operations with minimal effort.

---

##  Services Used

* AWS Lambda — Executes backup logic
* Amazon EC2 — Hosts instances
* Amazon EBS — Volumes to backup
* AWS IAM — Manages permissions

---

##  Architecture Overview

**Manual Trigger → Lambda → EBS Volume → Snapshot Created**

> The Lambda function is triggered manually to create snapshots of all active EBS volumes.

<img width="1100" height="604" alt="image" src="https://github.com/user-attachments/assets/e62ea9e2-758e-459c-ab34-9eb5b491933c" />

---

##  Implementation Steps

### 🔹 Step 1: Create IAM Role

* Create a role for Lambda
* Attach policy:

  * `AmazonEC2FullAccess`

    <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/03254b97-4483-4e69-a9f4-d5ff4612cbcd" />
    <img width="640" height="360" alt="image" src="https://github.com/user-attachments/assets/41cdf52f-f36f-4422-960b-a810bbf0ee55" />
    <img width="640" height="360" alt="image" src="https://github.com/user-attachments/assets/81fb97c0-ad88-4f23-8caa-635ef25a7b8f" />
    <img width="640" height="360" alt="image" src="https://github.com/user-attachments/assets/f4216fac-e66a-41c4-aed8-c75be7d92c55" />

---

### 🔹 Step 2: Create Lambda Function

* Runtime: Python 3.x
* Add code to:

  * Fetch all regions
  * Get EBS volumes
  * Create snapshots

    <img width="640" height="360" alt="image" src="https://github.com/user-attachments/assets/99116095-9801-46b6-a51c-950bad742895" />
    <img width="640" height="360" alt="image" src="https://github.com/user-attachments/assets/b9fe8857-18bc-442c-a51e-cade1a497da4" />

---

### 🔹 Step 3: (Optional) Tag Volumes

* Key: `Backup`
* Value: `true`

---

### 🔹 Step 4: Trigger Lambda

* Go to Lambda Console
* Click **Test**
* Execute function to create snapshots

  <img width="640" height="311" alt="image" src="https://github.com/user-attachments/assets/ded03fb9-14cc-4b22-983d-1dd5df54dd2b" />
  <img width="1100" height="619" alt="image" src="https://github.com/user-attachments/assets/eb93116f-7e89-4f7a-aa18-2286042000e1" />
  <img width="634" height="357" alt="image" src="https://github.com/user-attachments/assets/976fb375-f858-43b9-93c8-cb9a84e43284" />

---

##  Lambda Code

```python
import boto3

def lambda_handler(event, context):
    regions = []

    # Get all regions
    ec2_client = boto3.client('ec2')
    regions_response = ec2_client.describe_regions()

    for region in regions_response['Regions']:
        regions.append(region['RegionName'])

    snapshots_created = []

    # Loop through each region
    for region in regions:
        print(f"Processing region: {region}")
        ec2 = boto3.client('ec2', region_name=region)

        volumes = ec2.describe_volumes(
            Filters=[{'Name': 'status', 'Values': ['in-use']}]
        )['Volumes']

        for volume in volumes:
            volume_id = volume['VolumeId']
            print(f"Creating snapshot for Volume: {volume_id}")

            try:
                snapshot = ec2.create_snapshot(
                    VolumeId=volume_id,
                    Description=f"Snapshot of {volume_id} from region {region}"
                )

                snapshots_created.append({
                    "Region": region,
                    "VolumeId": volume_id,
                    "SnapshotId": snapshot['SnapshotId']
                })

                print(f"Snapshot created: {snapshot['SnapshotId']}")

            except Exception as e:
                print(f"Error: {str(e)}")

    return {
        "statusCode": 200,
        "body": f"Snapshots created: {snapshots_created}"
    }

```
<img width="1100" height="619" alt="image" src="https://github.com/user-attachments/assets/97fdf2ff-45da-4237-a272-361776b01aad" />


---

## How It Works

1. User triggers Lambda manually
2. Lambda fetches EBS volumes
3. Snapshots are created instantly
4. Backups are stored in AWS

---

## Key Features

✔ Serverless execution
✔ On-demand backup
✔ Simple and easy setup
✔ No infrastructure management
✔ Supports multiple volumes

---

##  Use Cases

* Quick manual backups before deployment
* Testing backup processes
* Small-scale environments

---

##  Benefits

* Reduces manual console work
* Faster backup execution
* Easy to extend for automation later

---

## Limitations

* Not fully automated (no scheduling)
* No monitoring or alerts

---

## Conclusion

This project demonstrates how to use AWS Lambda to simplify backup operations. Even though it is manually triggered, it provides a strong foundation for building a fully automated backup system.

---

##  Author

**Vaishnavi Jagtap**

---

## License

This project is licensed under the MIT License.

