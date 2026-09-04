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

                    
