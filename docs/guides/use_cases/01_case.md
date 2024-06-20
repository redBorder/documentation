# Brute force authentication attack alert

We can use Redborder to alert of possible brute force attacks thanks to the use of its correlation engine and the rules that are established. First, we must go to the Correlation Engine Rules section located in the Tools part.

![CEP rules](images/ch30_img002_a.png)
  
Then we will activate the rule we want to use. In this case: SSH Brute Force Authentication. Finally, we must apply the changes.

![CEP rules](images/ch30_img002_b.png)
	
This rule is capable of generating an alert when 200 unsuccessful log-in attempts occur in a 20 minute time window on a Linux machine via SSH protocol. For this, it is necessary to have the corresponding vault sensor created previously by referring to the IP of the machine which we want to protect. Once this is done, in case of receiving a brute force attack, we could visualize the corresponding alert by accessing the Vault module. To be able to view it easily, we can add a sensor tab and filter by CEP.

![Vault module: Brute force attack alert.](images/ch30_img002_c.png)

We can also display the alert message if we switch to the message tab.

![Vault module: Brute force attack alert message.](images/ch30_img002_d.png)
To expand the use cases, we can access the following link where the advanced use cases are located: