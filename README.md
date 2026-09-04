# Setup-Scheduling-of-EC2-Server.-any-time-frame-you-choose-
Objective

Automatically manage one EC2 instance:

9:00 AM IST, Monday-Friday → START EC2
7:00 PM IST, Monday-Friday → STOP EC2

                 Amazon EventBridge Scheduler
                           │
                 ┌─────────┴─────────┐
                 │                   │
                 ▼                   ▼
          Start Schedule        Stop Schedule
          9:00 AM IST           7:00 PM IST
          Mon-Fri               Mon-Fri
                 │                   │
                 ▼                   ▼
          EC2 StartInstances   EC2 StopInstances
                 │                   │
                 └─────────┬─────────┘
                           ▼
                      EC2 Instance

                    
Step 1: Identify the EC2 Instance

Go to:

AWS Console → EC2 → Instances

Select the instance you want to schedule.

Copy its Instance ID.

Example:

i-0123456789abcdef0

Keep this ID. You will use it in both schedules.

Step 2: Create IAM Role for Scheduler

This is an important part.

EventBridge Scheduler needs permission to call the EC2 API.

Go to:

AWS Console → IAM → Roles → Create role

Choose:

Trusted entity type:
AWS service

For the service/use case, choose:

EventBridge Scheduler

Then attach a policy that allows the required EC2 actions.

Recommended permissions

For this practical, you only need:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances"
      ],
      "Resource": "*"
    }
  ]
}

Give the role a name such as:

EC2-Scheduler-Role
Important

You do not need EventBridgeFullAccess for the Scheduler execution role.

EventBridgeFullAccess is an administrative permission and is not what gives the scheduler permission to start/stop your EC2.

The Scheduler execution role needs permission to perform:

ec2:StartInstances
ec2:StopInstances
Step 3: Open EventBridge Scheduler

Go to:

AWS Console → EventBridge → Scheduler

Then select:

Create schedule

Step 4: Create EC2 Start Schedule

Set:

Schedule name
EC2-Start-9AM
Schedule pattern

Choose:

Recurring schedule

Then choose:

Cron-based schedule
Recommended approach: Select timezone

Instead of manually converting IST to UTC, set the timezone to:

Asia/Kolkata

Then use:

0 9 ? * MON-FRI *

This means:

Minute       = 0
Hour         = 9
Day          = ?
Month        = *
Day of week  = MON-FRI
Year         = *

So:

Monday-Friday at 9:00 AM IST.

Step 5: Disable Flexible Time Window

Find:

Flexible time window

Set:

Off

This is important because you want the EC2 action to happen at the scheduled time rather than allowing Scheduler to invoke it within a flexible window.

Step 6: Configure Target

For the target, choose the AWS API target option.

Select:

AWS service

Then:

EC2

Choose the API action:

StartInstances

For the request/input, enter:

{
  "InstanceIds": [
    "i-0123456789abcdef0"
  ]
}

Replace:

i-0123456789abcdef0

with your actual EC2 Instance ID.

Step 7: Select IAM Role

Under execution role, select:

EC2-Scheduler-Role

The role must have:

ec2:StartInstances

permission.

Then create the schedule.

Step 8: Verify Start Schedule

Your configuration should look approximately like:

Schedule name:
EC2-Start-9AM

Schedule type:
Recurring

Timezone:
Asia/Kolkata

Cron:
0 9 ? * MON-FRI *

Flexible time window:
Off

Target:
EC2 → StartInstances

Instance:
i-xxxxxxxxxxxxxxxxx

Execution role:
EC2-Scheduler-Role

State:
Enabled
Step 9: Create Stop Schedule

Now create a second schedule.

Go to:

EventBridge → Scheduler → Create schedule

Schedule name
EC2-Stop-7PM
Schedule type
Recurring schedule
Cron
0 19 ? * MON-FRI *
Timezone
Asia/Kolkata

This means:

Monday-Friday at 7:00 PM IST.

Step 10: Configure Stop Target

Choose:

AWS service
→ EC2
→ StopInstances

Input:

{
  "InstanceIds": [
    "i-0123456789abcdef0"
  ]
}

Again, replace it with your actual instance ID.

Select:

EC2-Scheduler-Role

Make sure the role has:

ec2:StopInstances

permission.

Then create the schedule.

Step 11: Final Configuration

You should now have two schedules.

Schedule	Timezone	Cron	Action
EC2-Start-9AM	Asia/Kolkata	0 9 ? * MON-FRI *	Start EC2
EC2-Stop-7PM	Asia/Kolkata	0 19 ? * MON-FRI *	Stop EC2
Daily flow
Monday-Friday

        9:00 AM
           │
           ▼
 EventBridge Scheduler
           │
           ▼
 StartInstances API
           │
           ▼
      EC2 RUNNING
           │
           │
       Working Day
           │
           ▼
        7:00 PM
           │
           ▼
 EventBridge Scheduler
           │
           ▼
 StopInstances API
           │
           ▼
       EC2 STOPPED
Step 12: How to Test It

You don't need to wait until 9 AM or 7 PM to test it.

For your practical, temporarily create schedules a few minutes ahead of the current time.

For example, if the current time is 4:30 PM, create:

Start → 4:35 PM
Stop  → 4:40 PM

with timezone:

Asia/Kolkata

Then watch:

EC2 → Instances

You should see:

4:35 PM
stopped → pending → running

and then:

4:40 PM
running → stopping → stopped

After testing, you can change the schedules back to:

9:00 AM → Start
7:00 PM → Stop
One important correction in your original notes

You have this:

Cron: 0 19 * * ? *

and also:

cron(30 13 ? * MON-FRI *)

Both can represent 7 PM IST depending on whether you are using the timezone field or manually converting to UTC.

If timezone = Asia/Kolkata

Use:

0 19 ? * MON-FRI *
If timezone = UTC

7 PM IST = 1:30 PM UTC, so use:

30 13 ? * MON-FRI *

For your practical, I strongly recommend using Asia/Kolkata and 0 19 ? * MON-FRI *. It is much easier to understand and document.

Also, if your actual requirement is now 5 PM instead of 7 PM, simply change the stop schedule to:

0 17 ? * MON-FRI *

That will stop the EC2 at 5:00 PM IST.
