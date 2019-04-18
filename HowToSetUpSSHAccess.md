###How to set up/access ssh an ec2 instance
1. Go to ec2 dashboard
2. Click create auto scaling group. *See other guide on how to do this if you dont know*
3. Go all the way through the process until you come to a button that says *create auto scaling group* and click it.
4. A popup shows that says *Select an existing keypair or create a new one* Choose *create a new one*.
5. Download the key pair and put it somewhere safe on your computer.
6. Open the terminal and run chmod 400 <mykeypair>
7. After the instance is created run this to ssh to your EC2.

```ssh -i path/to/keypair/mykeypair.pem ec2-user@<public dns>```

Tips: For the second part of the last value ec2-user@this_part ... get that value from the ec2 dashboard.
It is located under the ec2 description and is called `Public DNS (IPv4)`
