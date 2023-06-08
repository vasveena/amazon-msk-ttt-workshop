# **Step 1 - Enabling open monitoring for an existing Amazon MSK cluster**

```
To enable open monitoring, make sure that the cluster is in the ACTIVE state.
```
**Using the AWS Management Console**
1. Sign in to the AWS Management Console, and open the Amazon MSK console at https://console.aws.amazon.com/msk/home?region=us-east-1#/home/.
2. Choose the name of the cluster that you want to update. This takes you to a page the contains details for the cluster.
![Intro_Choose_Cluster](images/Intro_Choose_Cluster.png)
3. On the Properties tab, scroll down to find the Monitoring section.
![Properties](images/Properties.png)
4. Choose Edit.
![Edit_Monitoring](images/Edit_Monitoring.png)
5. Select the check box next to Enable open monitoring with Prometheus.
![Enable](images/Enable.png)
6. Choose Save changes.

# **Step 2 - Create Securirty group rules to allow monitoring access**
