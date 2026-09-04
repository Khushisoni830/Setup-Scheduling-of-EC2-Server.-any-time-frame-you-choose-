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

                    


Step 2: Create IAM Role for Scheduler

This is an important part.

EventBridge Scheduler needs permission to call the EC2 API.

Go to:

AWS Console → IAM → Roles → Create role

Choose:

Choose:

Trusted entity type:
AWS service

For the service/use case, choose:

EventBridge Scheduler

Then attach a policy that allows the required EC2 actions.

Recommended permissions
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
